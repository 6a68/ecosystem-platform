---
id: integration-with-subscription-platform
title: Integration with Subscription Platform
sidebar_label: Integration with Subscription Platform
---

## Overview

The Subscription Platform handles all aspects of subscription management for Firefox Accounts. Integrations require first handling basic integration with FxA, after which the additional subscription capabilities can be determined, configured, and deployed. When integrated with the Subscription Platform, an additional event will be sent via [webhook][webhook] and the users subscription capabilities will be added to the [user’s profile data][profile-data].

## Terminology

Stripe is used to handle payments and subscriptions in the Subscription Platform. Terminology dealing with products and plans uses the [Stripe definitions of Product/Plan](https://stripe.com/docs/billing/subscriptions/products-and-plans).

### Subscription Capability

Each product is associated with the capabilities that it confers to a user. A bundle product may include multiple capabilities provided by varying [relying parties (RPs)][relying-party], or a single product may confer a single capability provided by multiple RPs. These capabilities are typically RP-specific.

Subscription capabilities are lower-case strings that will be included in the users profile data and  [webhook][webhook] [subscription state notifications](https://github.com/mozilla/fxa/tree/master/packages/fxa-event-broker#subscription-state-change).

## Pre-Development

Before getting started, you should have had a RRA-style meeting with the Firefox Accounts team. During or shortly after this meeting, an initial subscription capability string should have been agreed upon. If the product/plan did not exist yet, one will be created for the FxA stage environment. You will need these three strings to develop a subscription platform integration:

- Subscription Capability string
- Product id
- Plan id

## Development



[relying-party]: https://en.wikipedia.org/wiki/Relying_party
[webhook]: https://en.wikipedia.org/wiki/Webhook
[profile-data]: https://mozilla.github.io/application-services/docs/accounts/faq.html#what-information-does-firefox-accounts-store-about-the-user
