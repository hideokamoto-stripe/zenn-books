---
title: "Step 3: Learning to Generate Product Pages Using Stripe API"
---

We successfully launched the e-commerce system.

Sales are going smoothly, and new products are gradually increasing.

## Task: Let's streamline the work of reflecting added or discontinued products
One day, you are consulted by the operations team that "it's hard to add code snippets every time you add or delete products."

They also want to add a "cart function" in the future that allows you to order multiple products at once.

Let's update the integration with Stripe so that it can handle the addition of a cart function without changing the application's code.

## 1: Create a type information file
We will add custom type definitions that both the API and React will use.

Create `app/types.ts` and add the following code.

```ts:app/types.ts
import Stripe from "stripe"

export type ProductWithPrices = Stripe.Product & {
    prices: Array<Stripe.Price>
}
```

## 2: Retrieve Product Information from the Stripe API
Let's first retrieve product information from Stripe.

Create `app/api/prices/route.ts` and add the following code.

```ts:app/api/prices/route.ts
import { NextResponse } from "next/server";
import { ProductWithPrices } from "@/app/types";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY as string, {
  apiVersion: '2022-11-15'
})

export async function GET() {
    const { data: prices } = await stripe.prices.list({
        limit: 100,
        expand: ['data.product']
    })
    const products: Array<ProductWithPrices> = []
    prices.forEach(price => {
        const product = price.product as Stripe.Product
        // Stripe CLIが生成するサンプルデータを除外する
        if (product.description === '(created by Stripe CLI)') {
            return
        }
        const productIndex = products.findIndex(prod => prod.id === product.id)
        const _price = {
            ...price,
            product: product.id
        }
        if (productIndex < 0) {
            const _product: ProductWithPrices = {
                ...product,
                prices: [_price]
            }
            products.push(_product)
        } else {
            const targetProduct = products[productIndex]
            targetProduct.prices.push(_price)
            products[productIndex] = targetProduct
        }
    })
    return NextResponse.json(products)
}
```

Now try calling the API with cURL.

```bash
curl http://localhost:3000/api/prices  
```

If you see a response like the following, it is successful:



```bash
[  {
    "id": "prod_OO33gMiWolXtwS",
    "object": "product",
    "active": true,
    "attributes": [],
    "created": 1691128416,
    "default_price": "price_1NbGxALQkVoOEzC2eiPDrjXm",
    "description": null,
    "images": [
      "https://files.stripe.com/links/MDB8YWNjdF8xTlpwaDhMUWtWb09FekMyfGZsX3Rlc3RfOUhFQ1ZpNHpnSFpUTXcxMUh5Z1FCRnRv00KU7V8wof"
    ],
    "livemode": false,
    "metadata": {},
    "name": "Sofa",
    "package_dimensions": null,
    "shippable": null,
    "statement_descriptor": null,
    "tax_code": null,
    "type": "service",
    "unit_label": null,
    "updated": 1691128417,
    "url": null,
    "prices": [
      {
        "id": "price_1NbGxALQkVoOEzC2eiPDrjXm",
        "object": "price",
        "active": true,
        "billing_scheme": "per_unit",
        "created": 1691128416,
        "currency": "jpy",
        "custom_unit_amount": null,
        "livemode": false,
        "lookup_key": null,
        "metadata": {},
        "nickname": null,
        "product": "prod_OO33gMiWolXtwS",
        "recurring": null,
        "tax_behavior": "unspecified",
        "tiers_mode": null,
        "transform_quantity": null,
        "type": "one_time",
        "unit_amount": 150000,
        "unit_amount_decimal": "150000"
      }
    ]
  }
]
```


:::message
**Code Explanation**
With Stripe, when fetching both pricing and product information, you have two choices: "Fetch all at once with the Price API" or "Combine the Product API and Price API to fetch". In this case, we're using the less commonly used example of "Fetch all at once with the Price API".

This method has the advantage of allowing you to "fetch all product information associated with the price", but the drawback of requiring array manipulation to generate responses based on products.
:::

### Call the Stripe API from Next.js
Next, let's call the API we created from the UI (`page.tsx`).

Change `app/products/page.tsx` as follows:

```diff tsx:app/products/page.tsx
import { Metadata } from 'next'
+import { ProductWithPrices } from '@/app/types'
+import { headers } from 'next/headers'
export const metadata: Metadata = {
  title: 'Products',
}
-export default function Page() {
+export default async function Page() {
+    const headersData = headers()
+    const protocol = headersData.get('x-forwarded-proto')
+    const host = headersData.get('host')
+    const apiBase = `${protocol}://${host}`
+    const products:Array<ProductWithPrices> = await fetch(`${apiBase}/api/prices`, {
+        next: {
+            revalidate: 10 * 60 // 10 minutes
+        }
+    }).then(data => data.json())
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
+            <pre><code>{JSON.stringify(products,null,2)}</code></pre>
            <h1 className='text-4xl font-extrabold'>Products</h1>
            <script async
                src="https://js.stripe.com/v3/buy-button.js">
            </script>
            <stripe-buy-button
                buy-button-id="buy_btn_1xxxxx"
                publishable-key="pk_test_xxxxx"
            >    
            </stripe-buy-button>
        </div>
    </main>
  )
}
```

Try reloading the page at http://localhost:3000/products.

![](https://storage.googleapis.com/zenn-user-upload/98b8cc1d8d57-20230807.png)

Data fetched from the API is now displayed.

https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating

:::message
**Whether to 'staticize' or 'cache'**

From Next.js Version 13 onwards, you can call the Stripe API directly from `page.tsx`.

However, if you call it directly, the data will only be updated during the build time of the app (`next build`). This is called "static rendering".

https://nextjs.org/docs/app/building-your-application/rendering/static-and-dynamic

Therefore, in scenarios other than where "product updates are infrequent and manual rebuild is possible", we recommend the caching method introduced here.
:::

### Display product list from retrieved product information

Let's replace the Buy Button with the product data we have retrieved.

Change `app/products/page.tsx` as follows:


```diff tsx:app/products/page.tsx
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
-            <pre><code>{JSON.stringify(products,null,2)}</code></pre>
            <h1 className='text-4xl font-extrabold'>Products</h1>
-            <script async
-                src="https://js.stripe.com/v3/buy-button.js">
-            </script>
-            <stripe-buy-button
-                buy-button-id="buy_btn_1xxxxx"
-                publishable-key="pk_test_xxxxx"
-            >    
-            </stripe-buy-button>
+            {products.map(product => {
+                return (
+                    <section key={product.id} className="bg-white pb-10 rounded-lg">
+                        <Image
+                            src={product.images[0]}
+                            alt={`Product image of ${product.name}`}
+                            width={350}
+                            height={500}
+                            className='rounded-t-lg'
+                        />
+                        <div className="px-10 mt-5">
+                            <h2 className='text-xl font-bold'>{product.name}</h2>
+                            {product.prices.map(price => {
+                                if (price.recurring) return null
+                                return (
+                                    <p
+                                        key={price.id}
+                                        className="flex justify-between my-2 items-center"
+                                    >
+                                        <span>{price.unit_amount?.toLocaleString()} {price.currency}</span>
+                                        <button
+                                            className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
+                                        >Buy now</button>
+                                    </p>
+                                )
+                            })}
+                        </div>
+                    </section>
+                )
+            })}
        </div>
    </main>
  )
```

After reload the page, the display will switch from the Buy Button to your custom product list page.

![](https://storage.googleapis.com/zenn-user-upload/74808493c7f6-20230807.png)


## 3: Use Stripe Checkout to Display an Order Button
Finally, let's add the process of ordering the product we have displayed.


### Prepare an API to Redirect to the Payment Page

First, let's prepare an API to create a session of Stripe Checkout.

Create `app/api/checkout/route.ts` and add the following code.


```ts:app/api/checkout/route.ts
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY as string, {
  apiVersion: '2022-11-15'
})

export async function POST(request: NextRequest) {
    const origin = request.headers.get('origin') || 'http://localhost:3000'
    const referer = request.headers.get('referer') || 'http://localhost:3000'

    if (request.headers.get('content-type') === 'application/x-www-form-urlencoded') {
        const body = await request.formData()
        const priceId = body.get('price_id')?.toString()
        if (!priceId) {
            return NextResponse.json({
                message: 'Cart is empty'
            }, {
                status: 400
            })
        }

        const session = await stripe.checkout.sessions.create({
            line_items: [{
                price: priceId,
                quantity: 1,
            }],
            cancel_url: referer,
            success_url: `${origin}?success=true`,
            mode: 'payment'
        })
        if (session.url) {
            return NextResponse.redirect(new URL(session.url), 303)
        } else {
            return NextResponse.json({
                message: 'Failed to create a new checkout session. Please check your Stripe Dashboard.'
            }, {
                status: 400
            })
        }
    } else if (request.headers.get('content-type') === 'application/json') {
        // If you want to add cart feature, use this section
    }
    return NextResponse.json({
        message: 'Invalid request'
    }, {
        status: 400
    })
}
```


:::message
**[Advanced Task] Try to support Orders for Multiple Products**
In this workshop, we only introduce how to order each product.
However, Stripe Checkout also supports orders for multiple types and quantities of products.

Using the `else if (request.headers.get('content-type') === 'application/json') {}` part,
let's think about how to process orders for multiple types and quantities of products.
:::


### Add a Process to Move to the Payment Page from the Order Button

Let's link the API we created with the order button.

Change `app/products/page.tsx` to the following.

```diff tsx:app/products/page.tsx
    {product.prices.map(price => {
        if (price.recurring) return null
        return (
-            <p
+            <form
+                action={`/api/checkout`}
+                method='POST'
+                key={price.id}
                className="flex justify-between my-2 items-center"
            >
+                <input type="hidden" name="price_id" value={price.id} />
                <span>{price.unit_amount?.toLocaleString()} {price.currency}</span>
                <button
+                    type="submit"
                    className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                >Buy now</button>
+            </form>
-            </p>
        )
    })}
```

With this, we have a mechanism to move to the Stripe Checkout order page for each product.

![](https://storage.googleapis.com/zenn-user-upload/0b22fb0a7d02-20230807.png)

## Quick recap

- With Next.js, you can implement both server-side and client-side processing
- The Stripe API allows you to retrieve product and price information
- Stripe Checkout supports dynamic payment flows and cart functionalitiesる
