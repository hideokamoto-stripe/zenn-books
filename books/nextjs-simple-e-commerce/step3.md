---
title: "Step 2: Developing a Next.js REST API and Integrating Order Processing"
---

In Step 1, we were able to prepare product registration and display, as well as a flow to the order page.

In e-commerce sites, after a customer orders a product, it is necessary to prepare and ship the product.

In order to send order information to the internal shipping system, let's develop a "Webhook API" that links the orders received from Stripe to the external system.

:::message
**Supplement on "Integration with internal systems"**
In the workshop, we will develop up to the API that receives order information sent from Stripe.

Let's proceed with the development flow ... such as 
*"We prepare the mechanism to receive the order information, and the actual integration work is handled by another team."*
:::

## Adding a REST API with Next.js(v13 App Router)

## Relay Stripe Webhook Events to Next.js with Stripe CLI

## Implement the Process to Display Order Details

## Quick recap
- In the App Router of Next.js(v13), you can add an API with `app/[path name]/route.ts`
- Which methods to support can be defined by `export function [method name]`
- With Stripe CLI, you can forward Stripe's Webhook events to the local environment