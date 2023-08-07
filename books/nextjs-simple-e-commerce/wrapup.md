---
title: "Wrap Up: Taking Your E-commerce App to the Next Level"
---

Congratulations on completing the workshop! You've now learned the essential skills for developing an efficient e-commerce solution. Keep exploring to further enhance your app and skills.

## Hints for More Practical Use Cases
Finally, let's introduce some use cases where you often need to research and consider when actually building an e-commerce site.
I'll also introduce some Stripe document URLs, so please think about "how would you implement it?"

### 1: Deploying to Hosting Services
In this workshop, we worked in a local environment. However, to actually publish a site, you need to deploy your application to services like AWS, Firebase, Vercel/Cloudflare, etc.

Try deploying while investigating things like "Does it support Next.js (App Router)?", "Do I need to handle maintenance tasks like Node or Nginx?", "How to set environment variables and build/deploy flow?" etc.

https://aws.amazon.com/jp/blogs/news/amplify-next-js-13/

https://vercel.com/docs/concepts/projects/environment-variables

### 2: Databases & Inventory Management Systems
Stripe alone cannot provide a database for inventory management. However, for large e-commerce sites that frequently conduct sales, strict inventory management is essential.

Let's explore methods of realizing inventory management using tools such as Vercel Postgress or Cloudflare D1, and their integration with Stripe.

https://stripe.com/docs/payments/checkout/managing-limited-inventory

https://vercel.com/docs/storage/vercel-postgres

### 3: Subscriptions
With Stripe, you can easily incorporate regular sales or subscriptions. Taking as an example the "furniture mail-order site" introduced as a sample in this workshop, please try to consider:

- What kind of subscription service can you provide?
- What kind of pricing system can be proposed with Stripe?
- How will you integrate your created Next.js application?
- How will you support changes in customer payment information or plan changes?

and so on.

https://stripe.com/docs/billing/quickstart

https://stripe.com/docs/customer-management