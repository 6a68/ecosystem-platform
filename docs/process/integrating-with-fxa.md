---
id: integration-with-fxa
title: Integration with FxA
sidebar_label: Integration with FxA
---

## Overview

Firefox Accounts integration is available for Mozilla groups on request. This integration is handled using [OAuth 2.0][oauth], OpenID Connect, and [webhooks][webhook] for authentication, authorization, and receiving events regarding FxA users.

## Pre-Development

Before getting started, request a short RRA-style meeting with the Firefox Accounts team. This meeting will cover the integration topics listed below:

1. Type of integration (web site vs native app vs extension vs in browser)
2. Do you know how to implement OAuth?
3. Will you need access to a [user’s profile data][profile-data]?
   - If yes, will you need to write to a user’s profile data?
4. Will you need to access Sync data?
5. Will you need encryption keys to encrypt user data?
6. Will you need ongoing access to either a user’s profile or Sync data?
7. Will your application display its own “enter your email” form?
8. Who are the stakeholders?
9. Who can be contacted for important updates, e.g., API changes?
10. What are the QA dates?
11. What is the target release date?

The meeting request should be sent to fxa-staff[at]mozilla.com. Please be prepared to discuss these integration topics with the FxA team.

## OAuth Integration

### Development

1. A basic understanding of [OAuth 2.0][oauth] is required.
2. Register for OAuth credentials at https://oauth-stable.dev.lcip.org/console/login. See [OAuth credentials](#oauth-credentials).
3. Development servers point to: https://stable.dev.lcip.org
4. User authentication follows the [OAuth 2.0][oauth] protocol.
5. If the multiple Service Providers (SPs) are accessed, distinct tokens are created with appropriate scopes for each SP.
6. [Query parameters](#authorization-query-parameters) are set and validate when redirecting to Firefox Accounts.
7. [Self hosted email-first](#self-hosted-email-first-flow) flows initialize and propagate top of funnel metrics.
8. [User data and account notifications are properly](#user-data-hygiene-gdpr) handled and GDPR compliant.
9. An icon suitable to display in FxA’s [Devices & apps][devices-and-apps] list has been sent to FxA devs.

### Preparing for Production

1. Register for a production OAuth credentials by filing a [deployment bug][deployment-bug]. See [OAuth credentials](#oauth-credentials).
2. Production servers point to with https://oauth.accounts.firefox.com.
3. Someone from the FxA team has reviewed the integration code and tested the flow.

### User Authentication with OAuth 2.0 / OpenID Connect in a nutshell

1. Create a state token, associate it with a local session.
2. Send [/authentication request](#authorization-query-parameters) to Firefox Accounts. Upon completion, Firefox Accounts redirects back to your app with state and code.
3. Confirm returned state token by comparing it with state token associated with the local session.
4. Exchange code for an access token and possibly a refresh token (for clients that request offline use).
5. Fetch user profile information with access token
6. Associate profile information with local session and possibly create an account.

### OAuth Credentials

1. client_id - a public identifier that is used to identify your service. Can be public.
2. client_secret - a private secret that is sent from the backend when interacting with the OAuth server. Must not be shared publicly, checked into a public repository, or bundled with compiled code.

### Self Hosted Email-first Flow

1. Initialize top of funnel metrics by calling [/metrics-flow request][metrics-flow-request] with the required query parameters:
   a. entrypoint
   b. form_type (must be the string ‘email’)
   c. utm_source
   d. utm_campaign
2. Propagate email, flow_id and flow_begin_time query parameters to /authentication request

To test without CORS errors using https://stable.dev.lcip.org/, your test application must have one of the following URLs:

- http://127.0.0.1:8001
- http://localhost:8000
- http://127.0.0.1:8000
- https://www.mozilla.org
- https://www.allizom.org
- https://www-demo5.allizom.org
- https://www-demo4.allizom.org
- https://www-demo3.allizom.org
- https://www-dev.allizom.org

### /authorization query parameters

1. client_id (required)
2. [scope](#scopes) (required)
3. state (required)
4. entrypoint (required)
5. email (required for [self hosted email-first flow](#self-hosted-email-first-flow))
6. flow_begin_time (required for [self hosted email-first flow](#self-hosted-email-first-flow))
7. flow_id (required for [self hosted email-first flow](#self-hosted-email-first-flow))
8. code_challenge (required for PKCE)
9. code_challenge_method (required for PKCE)
10. action (suggested, should be the string ‘email’)
11. access_type (suggested)
12. utm_campaign (suggested)
13. utm_source (suggested)
14. utm_medium (optional)
15. utm_term (optional)

### Scopes

1. Sync data
2. Profile data

### User Data Hygiene / GDPR

1. Accounts should use uid rather than email address as the primary key. An account’s primary email address can change.
2. [Primary email changed notifications](https://github.com/mozilla/fxa-auth-server/blob/master/docs/service_notifications.md#change-of-primary-email-address-event) should update the contact email stored with the account.
3. If profile information is stored, listen for [profile changed notifications](https://github.com/mozilla/fxa-auth-server/blob/master/docs/service_notifications.md#change-of-profile-data) or periodically refresh the profile information by using refresh token to create a fresh access token that can fetch profile information.
4. [Account deletion notifications](https://github.com/mozilla/fxa-auth-server/blob/master/docs/service_notifications.md#account-deletion-event) should remove any server side data related to the user.
5. Profile information should not be shared with 3rd parties without explicit consent.
6. [Destroy any outstanding access tokens and refresh tokens](https://github.com/mozilla/fxa-auth-server/blob/master/fxa-oauth-server/docs/api.md#post-v1destroy) whenever a user signals their session or account should be terminated, e.g., the user signs out of your site, closes their account on your site, or unsubscribes from all functionality.

[metrics-flow-request]: https://mozilla.github.io/application-services/docs/accounts/metrics.html#self-hosted-email-forms-and-metrics-tracking-aka-the-fxa-email-first-flow
[deployment-bug]: https://bugzilla.mozilla.org/enter_bug.cgi?assigned_to=nobody%40mozilla.org&bug_ignored=0&bug_severity=normal&bug_status=NEW&cf_fx_iteration=---&cf_fx_points=---&cf_status_firefox65=---&cf_status_firefox66=---&cf_status_firefox67=---&cf_status_firefox_esr60=---&cf_tracking_firefox65=---&cf_tracking_firefox66=---&cf_tracking_firefox67=---&cf_tracking_firefox_esr60=---&cf_tracking_firefox_relnote=---&component=Operations%3A%20Deployment%20Requests&contenttypemethod=list&contenttypeselection=text%2Fplain&defined_groups=1&flag_type-37=X&flag_type-5=X&flag_type-607=X&flag_type-708=X&flag_type-721=X&flag_type-737=X&flag_type-748=X&flag_type-787=X&flag_type-800=X&flag_type-803=X&flag_type-846=X&flag_type-864=X&flag_type-929=X&flag_type-935=X&form_name=enter_bug&groups=mozilla-employee-confidential&maketemplate=Remember%20values%20as%20bookmarkable%20template&op_sys=Unspecified&priority=--&product=Cloud%20Services&rep_platform=Unspecified&target_milestone=---&version=unspecified
[devices-and-apps]: https://accounts.firefox.com/settings/clients
[profile-data]: https://mozilla.github.io/application-services/docs/accounts/faq.html#what-information-does-firefox-accounts-store-about-the-user
[oauth]: https://auth0.com/docs/protocols/oauth2
[webhook]: https://en.wikipedia.org/wiki/Webhook
