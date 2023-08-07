---
title: "Introduction: Setting the Stage for the workshop"
---

Let's get started with the workshop right away.

First, let's prepare the necessary tools and accounts.

## Creating a Stripe Account
You'll need a Stripe account to store your data.
If you don't have an account yet, please create one using the following URL:

https://dashboard.stripe.com/register

![](https://storage.googleapis.com/zenn-user-upload/f047e1f8729e-20230731.png)

Once you've created an account, log in to the dashboard.

https://dashboard.stripe.com/

### Create a New Test Account
First, let's create a store account for our test app.

Stripe allows you to create and manage multiple accounts for different stores or services.
Having separate accounts can help minimize the risk of unintended changes to customer data or billing amounts due to API testing or accidents during the workshop.

Click the icon in the top left corner and then click New Account.

![](https://storage.googleapis.com/zenn-user-upload/2729990856bd-20230731.png)

For your account, set the business or company name and the country where you operate.

Since this is a demo app, let's set it up as follows:

- Account Name: demo.furni.com
- Country of Operation: Japan


![](https://storage.googleapis.com/zenn-user-upload/bde24e643ada-20230731.png)

Once your settings are complete, you will be directed to your account's main page.

![](https://storage.googleapis.com/zenn-user-upload/dff17b4068c9-20230731.png)

:::message
**Tips: Using a Production Environment**
When you create an account, a test environment is provided.

Test environments have limitations such as "rejecting input other than Stripe-provided test card numbers" and "restricting the email addresses that can be sent invoices."

To actually accept customer payments, it's a good idea to apply for a production environment as soon as possible.

![](https://storage.googleapis.com/zenn-user-upload/49772e02c9a5-20230731.png)

Please note that there is a risk that your account may be suspended if "the information provided about your business is deemed insufficient" or "your business is determined to fall under the [prohibited/restricted category]((https://stripe.com/ja-it/legal/restricted-businesses))." We recommend that you make sure your business is not subject to any prohibited or restricted categories and provide as much detailed business information as possible.

:::

## Installing Stripe CLI
During the workshop, you'll need to forward Stripe webhook events to your local application.

For this purpose, let's install Stripe CLI.

### Installing with Homebrew
Installing with Homebrew is easy.

```bash
brew install stripe/stripe-cli/stripe  
```

### Installing on macOS / Windows
For macOS and Windows, download from the latest GitHub release.

https://github.com/stripe/stripe-cli/releases/latest

For macOS, it's "stripe_X.X.X_mac-os_x86_64.tar.gz" and for Windows, it's "stripe_X.X.X_windows_x86_64.zip". Each OS has its own download file.

After downloading, execute the file to make Stripe CLI available.

For other environments' installation methods, please check the Stripe documentation.

https://stripe.com/docs/stripe-cli#install

:::message
**If you are unable to install**
If you cannot install Stripe CLI, please use the Stripe Shell which is available in the browser.

While logged in to the dashboard, go to the [Stripe CLI documentation page](https://stripe.com/docs/stripe-cli).

Click the Try Online button.

![](https://storage.googleapis.com/zenn-user-upload/0344d992f943-20220420.png)

The CLI operation screen will be displayed at the bottom of the browser.

![](https://storage.googleapis.com/zenn-user-upload/b09650f24d2e-20220420.png)

:::

## Connecting to your Stripe account with Stripe CLI
Now that you've installed Stripe CLI, it's time to connect it to your Stripe account.

Stripe CLI accesses Stripe resources through the Stripe API. Therefore, you need to connect to your Stripe account and retrieve the API key.

### Connect to your Stripe account with stripe login
Use the stripe login command to connect to your Stripe account.
When you run the command, you'll see a message like "Press Enter to open the browser." Press the Enter key.

```bash
stripe login  
Your pairing code is: amuse-revive-timely-sturdy  
This pairing code verifies your authentication with Stripe.  
Press Enter to open the browser (^C to quit)  
```

:::message
**In case you are already logged in with a different account**
In Stripe CLI, you can connect to multiple Stripe accounts using --project-name.
If you want to change the default connected account to another Stripe account, modify the stripe command that appears from now on as follows.

```bash
stripe --project-name demo-furni
```

**Example: Getting a list of customers**

```bash
stripe --project-name demo-furni customers list
```
:::

If you are already logged in to the Stripe Dashboard, you'll see an access permission confirmation screen like the one below.

![](https://storage.googleapis.com/zenn-user-upload/d5ecd73d2edf-20230731.png)

Click Allow Access.

![](https://storage.googleapis.com/zenn-user-upload/8d56b179f6a7-20230731.png)

When you go back to the Stripe CLI screen, you'll see a message in English indicating that the connection was successful.

```
> Done! The Stripe CLI is configured for api-test with account id acct_xxxxxxxx  
  
Please note: this key will expire after 90 days, at which point you'll need to re-authenticate.  
```

Are you ready? Let's dive right into the workshop.

## Imagine the Story of the E-Commerce Site Development

Before we dive into the workshop, let's imagine the context for developing this system.

You've been assigned to an e-commerce project for a furniture sales company.

![](https://storage.googleapis.com/zenn-user-upload/f0caba6b4d68-20230727.png)

- This company produces high-quality **furniture** made-to-order and delivers it to customers upon request.
- Since the website doesn't have a lot of traffic, there's **no need for inventory management** or order limitations at this stage.
- Once you've **prepared the product sales system**, other team members will begin working on the site's design and content.
- Therefore, the goal is to **create a product sales feature** with a **short schedule** and **minimal code**.

Have you got the idea?

Now, let's explore how to integrate product sales functionality using Stripe into your **e-commerce site**.
