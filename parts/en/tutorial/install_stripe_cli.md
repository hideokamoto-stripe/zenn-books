
## Installing Stripe CLI
During the workshop, you'll need to forward Stripe webhook events to your local application.

For this purpose, let's install Stripe CLI.

### Installing with Homebrew
Installing with Homebrew is easy.

```bash
brew install stripe/stripe-cli/stripe  
```

### Installing on macOS / Windows
For macOS and Windows and the other environments' installation methods, please check the Stripe documentation.

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
