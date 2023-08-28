
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
