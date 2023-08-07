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
First, let's create the API.

Create the app/api/webhook/route.ts file and add the following code:

```ts:app/api/webhook/route.ts
import { NextResponse } from "next/server";

export async function POST() {
  return NextResponse.json({
      message: "Hello! Stripe Webhook."
  })
 }
```
In Next.js's App Router, the "directory name" becomes the path of the API, and if route.ts is located, the function corresponding to the method in the file (`POST`, `GET`, etc.) will be executed.

https://nextjs.org/docs/app/building-your-application/routing

This time, to handle a `POST` request to `/api/webhook`, we placed the `POST` function in the `app/api/webhook/route.ts file`.

You are successful if the message "Hello! Stripe Webhook." appears when executing a cURL command as follows:

```bash
 curl -XPOST http://localhost:3000/api/webhook

{
  "message": "Hello! Stripe Webhook."
}
```

### How to Check Request Content
To check the data sent from Stripe, etc., check the arguments of the `POST` function.

Let's change the `app/api/webhook/route.ts` file as follows:


```diff ts:app/api/webhook/route.ts
-export async function POST() {
+export async function POST(request: Request) {
+  const body = await request.json()
+  console.log(JSON.stringify(body, null, 2))
  return NextResponse.json({
-      message: "Hello! Stripe Webhook."
+      message: `Hello ${body.name ?? "there"}!`
  })
 }
```

Data sent in JSON can be obtained from `request.json()`.

Let's call the API with the following cURL command.


```bash
 curl -XPOST http://localhost:3000/api/webhook -d '{"name": "John"}'
{
  "message": "Hello John!"
}
```

You can confirm that the name you sent is included in the response.


## Relaying Stripe Webhook Events to Next.js Using Stripe CLI


Having prepared our API, the next crucial step that we will tackle in our workshop is the configuration process. This involves setting up the API to seamlessly receive and process events that occur on Stripe's platform.

One of the game-changing tools we will be using is the Stripe's webhook. With the webinar, you'll learn just how easy it is to transmit various essential events - from the completion or failure of payments to changes in subscription status - to external systems.

We also understand that these processes aren't limited to the live environments. Therefore, we'll delve into how to use Stripe CLI's `--forward-to` option to relay Stripe webhook events to your API in the local environment.

We'll run through the execution of the following commands:

```bash
stripe listen --forward-to http://locahost:3000/api/webhook
```

Once you see `Ready` displayed on your command line, you can celebrate a successful configuration setup! This sets the foundation for you to continue building and exploring in your local environment.

```
> Ready! You are using Stripe API Version [2022-11-15]. Your webhook signing secret is whsec_xxxxxx (^C to quit)
```

### Utilizing Stripe CLI to Confirm Successful Relay to Next.js

Let's validate if the events are being successfully recieved by the API created in Next.js.

We will send a Webhook event from Stripe CLI.


```bash
stripe trigger checkout.session.completed
```

The CLI execution screen will display logs from Stripe's API calls as follows:

```bash
Setting up fixture for: product
Running fixture for: product
Setting up fixture for: price
Running fixture for: price
Setting up fixture for: checkout_session
Running fixture for: checkout_session
Setting up fixture for: payment_page
Running fixture for: payment_page
Setting up fixture for: payment_method
Running fixture for: payment_method
Setting up fixture for: payment_page_confirm
Running fixture for: payment_page_confirm
Trigger succeeded! Check dashboard for event details.
```

Additionally, the terminal screen where `stripe listen` is being executed will display the results of the Webhook transmission.

```bash
2023-08-07 12:04:06   --> product.created [evt_1NcJjmLQkVoOEzC2Fbwk9riN]
2023-08-07 12:04:06  <--  [200] POST http://localhost:3000/api/webhook [evt_1NcJjmLQkVoOEzC2Fbwk9riN]
```

And, if the following event logs are displayed on the terminal screen running `next dev` or `npm run dev`, then it indicates a successful implementation.

```log
{
  "id": "evt_3NcJjpLQkVoOEzC20xKExGCI",
  "object": "event",
  "api_version": "2022-11-15",
  "created": 1691377449,
  "data": {
    "object": {
      "id": "pi_3NcJjpLQkVoOEzC20n0doDzG",
      "object": "payment_intent",
      "amount": 3000,
      "amount_capturable": 0,
      "amount_details": {
        "tip": {}
      },
...

```

This completes the setup of your development system to coordinate with various events happening within your Stripe account.


## Verifying Whether an API Request was Sent from Stripe
When integrating a system with a Webhook sent by a SaaS like Stripe, it is essential to "deny API requests that do not originate from that specific service."

In a situation where anyone can call up the API, there's a potential risk of exploitation, such as the "transmission of fraudulent order events."

**Example of sending fraudulent order data**
```bash
curl -XPOST http://localhost:3000/api/webhook -d '{ "id": "evt_3NcJjpLQkVoOEzC20xKExGCI", "object": "event", "api_version": "2022-11-15", "created": 1691377449, "data": { "object": { "id": "pi_3NcJjpLQkVoOEzC20n0doDzG", "object": "payment_intent", "amount": 3000, "amount_capturable": 0, "amount_details": { "tip": {} } } } }' -H 'Content-Type: application/json'
```

To counter this, Stripe uses two tools - the Webhook's signature secret and secret API key. You'll learn how to apply these in order to verify the authenticity of incoming requests.

### Retrieving the Signature Secret
Let's review the results from executing the stripe listen command earlier.

```
> Ready! You are using Stripe API Version [2022-11-15]. Your webhook signing secret is whsec_xxxxxx (^C to quit)
```

As specified, `whsec_xxx` is the signature secret when developing locally.

Create a `.env.local` file and save the copied value into that.

```env:.env.local
STRIPE_WEBHOOK_SECRET=wsec_xxx
```

### Adding Stripe's Secret API Key to the Environment Variables
For the signature verification process, you will also need Stripe's Secret API Key.

To check your API key, open the [Developers] menu on your dashboard.

![](https://storage.googleapis.com/zenn-user-upload/09ee8e5b4365-20230807.png)

Next, click on [API Keys] from the middle section of the menu.

![](https://storage.googleapis.com/zenn-user-upload/63c23d454cd6-20230807.png)


:::message
Tips: Three Types of Stripe API Keys

There are three types of API keys.

The first one is a ''Publishable Key''. This key is used on the frontend app for operations like tokenizing a card.
The second key is a ''Secret Key.'' This key provides access to all public APIs provided by Stripe. If leaked, there is a risk that customer information or order data could be retrieved.
The last key is a ''Restricted Key''. This key, like the Secret Key, is used to access Stripe's API, but access to resources and operations can be individually configured.

With a Publishable Key and a Secret Key, you can develop an app that uses Stripe.
However, if you want to operate your Stripe account more securely, we recommend using a Restricted Key instead of a Secret Key.
:::

The [Secret Key] in the [Standard Keys] section is the API key used on the server-side.
![](https://storage.googleapis.com/zenn-user-upload/63c23d454cd6-20230807.png)


Click the [Reveal test key] button.

![](https://storage.googleapis.com/zenn-user-upload/1cdad85e7692-20230807.png)

This will display your Secret API Key.
![](https://storage.googleapis.com/zenn-user-upload/0f607666f9c1-20230807.png)

You can copy the Secret API key simply by clicking on the displayed API key.

Let's add the acquired API key to your `.env.local` file.

```diff:.env.local
STRIPE_WEBHOOK_SECRET=wsec_xxx
+STRIPE_API_KEY=rk_test_xxxxx
```


### Adding Signature Verification Process to Next.js API
Now, let's enhance our Next.js API by adding a verification process to determine if the API is being called from a Stripe Webhook. This will be done utilizing the signature secret we retrieved in the previous step.

```diff ts:t.ts

```

  
A successful implementation can be verified by sending a Webhook event from Stripe CLI and observing no errors from the Next.js API.

```bash

```

You will notice that any attempt to directly call the API results in error.


```bash


```
  
Now, you are ready to safely operate your API integrated with Stripe.

## Implement the Process to Display Order Details

## Quick recap
- In the App Router of Next.js(v13), you can add an API with `app/[path name]/route.ts`
- Which methods to support can be defined by `export function [method name]`
- With Stripe CLI, you can forward Stripe's Webhook events to the local environment