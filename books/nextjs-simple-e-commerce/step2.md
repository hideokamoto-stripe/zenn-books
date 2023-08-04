---
title: "Step 1: Implementing a Simple Product Sales Page with Low Code"
---

 In this step, we'll walk you through implementing a simple product sales page with minimal code, allowing you to focus on the core features of your e-commerce site.

## Set up a web app with Next.js (version 13)

First, let's prepare the web app.

In this document, we will use Next.js (version 13).

Run the `create-next-app` command to set up the app.

https://www.npmjs.com/package/create-next-app


```bash
npx create-next-app@13
```

:::message
The Next.js documentation recommends $ npx create-next-app@latest.
However, this workshop document is created assuming the Next.js version 13 series (using version 13.4.12 at the time of document creation).

Therefore, for convenience, the document is set to always use Version 13.
:::

After the downloads are complete, you'll be asked if you want to download the create-next-app command. Enter y.

```bash
Need to install the following packages:
  create-next-app@13.4.12
Ok to proceed? (y) 
```

After the download is complete, the project setup will start in an interactive format.

First, you will be asked for the project name. Enter nextjs-stripe-e-commerce.


```bash
? What is your project named? › nextjs-stripe-e-commerce
```

You will be asked if you want to use TypeScript, and reply with 'y'.


```bash
? Would you like to use TypeScript? › No / Yes
```

You'll be asked if you want to use ESLint, and reply with 'y'.

```bash
? Would you like to use ESLint? › No / Yes
```

If you answer 'y' to the next question, Tailwind CSS will be set up.

```bash
? Would you like to use Tailwind CSS? › No / Yes
```

You will be asked about the directory structure. Since we don't use the src directory, choose 'no'.


```bash
? Would you like to use `src/` directory? › No / Yes
```

App Router is a feature added from Next.js version 13.

https://nextjs.org/docs/app

Since we will use it in this workshop, choose 'yes'.


```bash
? Would you like to use App Router? (recommended) › No / Yes
```

Lastly, you will be asked about Import aliases. Choose the default 'no' here as well.

```bash
? Would you like to customize the default import alias? › No / Yes
```

Once the settings are complete, the app setup will begin.


```bash

Creating a new Next.js app in /Users/hideokamoto/stripe/sandbox/javascript/nextjs-stripe-e-commerce.

Using npm.

Initializing project with template: app-tw 


Installing dependencies:
- react
- react-dom
- next
- typescript
- @types/react
- @types/node
- @types/react-dom
- tailwindcss
- postcss
- autoprefixer
- eslint
- eslint-config-next
```

If you see a message like the following, the setup is complete.


```bash

131 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Initialized a git repository.

Success! Created nextjs-stripe-e-commerce at /Users/stripe/sandbox/nextjs-stripe-e-commerce
```

## Starting a Next.js App in a Local Environment

Let's try running the setup code locally.

```bash
npm run dev
```
By executing this command, the app will be launched within your working PC.

```bash
$ next dev
- ready started server on 0.0.0.0:3000, url: http://localhost:3000
```

If the `3000` port is being used by another app, find an available port to use.

```bash
$ next dev
- warn Port 3000 is in use, trying 3001 instead.
- ready started server on 0.0.0.0:3001, url: http://localhost:3001
```

Accessing this URL from your browser will display the default page of the set up Next.js app.

![](https://storage.googleapis.com/zenn-user-upload/24d5928f6859-20230728.png)

## Setting up Layout in Next.js v13

In Next.js, you can use `app/layout.tsx` to set up the layout for the entire site.

https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts

Let's quickly add a header.

Please modify `app/layout.tsx` as follows:

```diff:typescript app/layout.tsx

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
+        <header className="relative bg-white">
+          <nav aria-label="Top" className="max-w-7xl mx-auto py-2 px-4 sm:px-6 md:px-8">
+            <div className="h-16 flex items-center">
+              <div className="hidden md:ml-8 md:block md:self-stretch">
+                <div className="h-full flex space-x-8">
+                  <a href="/" className="flex items-center text-sm font-medium text-gray-700 hover:underline">Home</a>
+                  <a href="/products" className="flex items-center text-sm font-medium text-gray-700 hover:underline">Products</a>
+                </div>
+              </div>
+              <div className="ml-auto flex items-center">
+                <div className="hidden md:flex md:flex-1 md:items-center md:justify-end md:space-x-6">
+                  <a href="/" className="flex items-center text-sm font-medium text-gray-700 hover:underline">My page</a>
+                </div>
+              </div>
+            </div>
+          </nav>
+        </header>
        {children}
      </body>
    </html>
  )
}
```

After saving the changes, the page will automatically reload.

If the header menu has been added, the edit was successful.

![](https://storage.googleapis.com/zenn-user-upload/ddf58cb9e85d-20230728.png)

## Register Your Products for Sale on Stripe

Now that your web app is ready, let's register the products you are going to list on Stripe.

Open the Stripe dashboard in your browser.

Click "products" from the dashboard menu.

![](https://storage.googleapis.com/zenn-user-upload/6deb907038be-20230804.png)

Direct link: https://dashboard.stripe.com/test/products

There's an "Add Product" button, let's click that.

![](https://storage.googleapis.com/zenn-user-upload/d4eb3a31930f-20230804.png)

The product registration screen will appear.

### Register Product Information
For the demo product, let's input and upload the following text and image.

|Product Name|Image（ unsplash.com ）|
|:--|:--|
|Sofa|https://images.unsplash.com/photo-1616627451515-cbc80e5ece35?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80|

![](https://storage.googleapis.com/zenn-user-upload/1f65c05d8c8d-20230804.png)

### Register Pricing Information and Save
Next, let's register the product price.

Scroll down the page to display the 'Pricing Information' form and input the following data.

|Pricing Model|Price|Currency|Type|
|:--|:--|:--|:--|
|Standard pricing|15000|JPY|One-Time|

![](https://storage.googleapis.com/zenn-user-upload/49502b8f841d-20230804.png)

Once you've finished entering, click 'Save Product' at the top right of the page.
![](https://storage.googleapis.com/zenn-user-upload/bd43e47770c1-20230804.png)

Once you've navigated to the product page, the registration is complete.

![](https://storage.googleapis.com/zenn-user-upload/31df2dfcd66a-20230804.png)

:::message
**Add Multiple Products**
I've prepared some samples as test data below. 
You don't need to register all of them.
But the more you have, the more it will look like an online store, so please try registering if you have time.

|Product Name|Price|Image|Pricing Model|Currency|Type|
|:--|:--|:--|:--|:--|:--|
|Chair|97000|[Link](https://images.unsplash.com/photo-1622147681210-d7da05b4a7d7?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&fm=png&fit=crop&w=600&q=80)| Standard pricing|JPY|One-Time|
|Wooden Stool|6000|[Link](https://images.unsplash.com/photo-1503602642458-232111445657?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80)| Standard pricing|JPY|One-Time|
|Retro Chair|12000|[Link](https://images.unsplash.com/photo-1519947486511-46149fa0a254?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80)| Standard pricing|JPY|One-Time|

:::

## Create Product Order Links and Buttons  
  
Stripe allows you to create an order page without any coding.   
  
Let's generate the URL of the order page from the dashboard.   
  
With the dashboard displayed, press `[c]` and then `[l]` on your keyboard.   

![](https://storage.googleapis.com/zenn-user-upload/2d629e71eed2-20230804.png)

This will open up the payment link creation screen.  
  
![](https://storage.googleapis.com/zenn-user-upload/98d5a663d252-20230804.png)
  
In this way, the Stripe dashboard provides multiple [keyboard shortcuts] to make operations smoother.   
  
![](https://storage.googleapis.com/zenn-user-upload/821551879e02-20230804.png)
  
:::message  
#### Other ways to open the payment link creation page  
Apart from keyboard shortcuts, there are other ways to navigate.  
  
##### Moving from the header menu  
Click the [Create] button at the top of the page and select [Payment link].  
![](https://storage.googleapis.com/zenn-user-upload/79811879e07d-20230804.png)
  
##### Directly moving via URL   
You can also bookmark and directly navigate to https://paymentlinks.new.  
:::  



On the Payment Links creation page, you can register products that can be ordered.


![](https://storage.googleapis.com/zenn-user-upload/98d5a663d252-20230804.png)
  

Click on the search form in the "Products" section on the left side of the setting screen.

A "Add New Product" button and a list of already registered products will be displayed.

![](https://storage.googleapis.com/zenn-user-upload/2bde0a6d1448-20230804.png)

Select the "Sofa" product.

Now the product has been added to the payment link you are editing.

![](https://storage.googleapis.com/zenn-user-upload/7db258c51ac8-20230804.png)

Then, by clicking on "Create Link" at the top right of the screen.

![](https://storage.googleapis.com/zenn-user-upload/40045f41fae6-20230804.png)

With just this, you have been able to issue a URL for ordering.

![](https://storage.googleapis.com/zenn-user-upload/e7146fe97166-20230804.png)

By copying the displayed URL or downloading and sharing the QR code, you can accept orders and payments from customers.

### Embedding the Buy Button Code Snippet

From 2023, it has become possible to embed payment links in your site as a button or in card format.

https://stripe.com/docs/payment-links/buy-button

Click on the "Buy Button" button in the Payment Link management screen.

![](https://storage.googleapis.com/zenn-user-upload/0454317ebdd9-20230804.png)

When you click, it will open the edit screen of the Buy Button, which can be embedded on websites and apps.

![](https://storage.googleapis.com/zenn-user-upload/82a63ddade49-20230804.png)

Click on "Copy Code".

With just pasting the copied code, you can add order and subscription application functions to websites and apps.


## Creating a Product Page in Next.js v13

Let's use the copied code snippet to add a product page to our web app.

### Adding a Product Page in the App Router

First, let's create a file for the product page.

Create `app/products/page.tsx`.

Let's create the layout for the product page.


```ts:app/products/page.tsx
import { Metadata } from 'next'
 
export const metadata: Metadata = {
  title: 'Products',
}
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
            <h1 className='text-4xl font-extrabold'>Products</h1>

         </div>
    </main>
  )
}

```

![](https://storage.googleapis.com/zenn-user-upload/54aa6315092e-20230804.png)

And paste the copied code.

```diff:tsx app/products/page.tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
            <h1 className='text-4xl font-extrabold'>Products</h1>
+            <script async
+                src="https://js.stripe.com/v3/buy-button.js">
+            </script>

+            <stripe-buy-button
+                buy-button-id="buy_btn_xxxx"
+                publishable-key="pk_test_xxxxxx"
+            >    
+            </stripe-buy-button>
        </div>
    </main>
  )
}
```

Now we were able to display the products registered with Stripe.

![](https://storage.googleapis.com/zenn-user-upload/eadac29a7f35-20230804.png)

:::message
***[Advanced] Advanced Register Multiple Products***

For those who registered multiple products in the previous step, let's try repeating this step to create a product list.

When adding multiple Buy Buttons, you don't have to add more `script` tag per page.

```diff:tsx app/products/page.tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
            <h1 className='text-4xl font-extrabold'>Products</h1>
            <script async
                src="https://js.stripe.com/v3/buy-button.js">
            </script>
            <stripe-buy-button
                buy-button-id="buy_btn_xxxx"
                publishable-key="pk_test_xxxxxx"
            >    
            </stripe-buy-button>

+            <stripe-buy-button
+                buy-button-id="buy_btn_xxxx"
+                publishable-key="pk_test_xxxxxx"
+            >    
+            </stripe-buy-button>
        </div>
    </main>
  )
}
```

![](https://storage.googleapis.com/zenn-user-upload/174da718a960-20230804.png)

:::


### Resolving TypeScript Errors
When developing in TypeScript, you may encounter the following type-related error.

```
./app/products/page.tsx:17:9
Type error: Property 'stripe-buy-button' does not exist on type 'JSX.IntrinsicElements'.
```

To resolve this error, let's add a`types/stripe.d.ts` file.

```ts:types/stripe.d.ts
declare namespace JSX {
    interface IntrinsicElements {
      'stripe-buy-button': {
        'buy-button-id': string;
        'publishable-key': string;
        children: unknown;
      };
    }
  }
```

This will eliminate the error.

## Try Ordering with Stripe's Test Card Information

Now that the product page has been created, let's try placing an actual order.

Stripe has prepared multiple payment information exclusively for the test environment for checking operation during development.

https://stripe.com/docs/testing

### Let's Try Ordering with a Credit Card Payment
Click the [Order] button on the product page.

[]

Stripe's redirect payment form will be displayed.

Here, enter the following card information.

|Field|Value|
|:--|:--|
|Card number| `4242424242424242`|
|Card expiration| `12`/`30` |
|CVC| `424`|
|Name| `Demo customer`|
|Email| `test@example.com`|

[]

When you click the Order button, a test payment will be made with your credit card.

[]

If you look at the "Payments" tab on the Stripe dashboard, you'll see the order from [Demo customer].

:::message
**[Advanced] Simulating Payment Failure and Fraud**

With Stripe, you can also test situations such as "When credit card balance is insufficient" or "When the product does not arrive and a complaint is received by the credit card company".

We have picked up three test cases from the documentation site, so please give them a try.

|Field|Value|
|:--|:--|
|Card Number (Insufficient Card Balance)| `4000000000009995`|
|Card Number (3D Secure Authentication)| `4000002760003184`|
|Card Number (Fraudulent Charge Claim)| `4000000000002685`|
|Card expiration| `12`/`30` |
|CVC| `424`|
|Name| `Demo customer`|
|Email| `test@example.com`|

:::

## Quick recap

- The overall layout of the site is set in `app/layout.tsx`
- When adding a page, use `app/[path name]/page.tsx`
- By registering a product with Stripe, you can create a payment form URL with no code required
- Stripe Payment Links can generate code snippets for embedding
