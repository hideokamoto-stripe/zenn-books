---
title: "Before starting the workshop"
---


Welcome to the Stripe E-commerce Site Development Workshop!
In this workshop, you'll learn how to create a simple e-commerce site using Stripe and Next.js.


## Prerequisites
Before starting the workshop or self-paced study, make sure you've prepared the following environment. In particular, the terminal (the so-called "black screen") will frequently appear during library installation and app building.

- Node.js version 12 or higher is installed
- An IDE such as Visual Studio Code is installed
- You can execute CLI commands using a terminal
- You have an internet connection and can download files using npm install

## How to Proceed with the Workshop
In every step, we provide "instructions with images," "commands to run," or "code to add."

### Instructions with Images
The main focus is on operating the Stripe dashboard and IDE.
If necessary, direct access links are provided in the instructions, so feel free to use those as well.

### Commands to Run
We use `npm` and `next` commands to install libraries and set up the development environment.

Commands are described as "run npm run dev" or "enter npm run dev and press the Enter key." Copy and paste these commands into the terminal, or type them in directly to execute them.

To avoid confusion, all command examples omit the `%` and `$` symbols.

### Adding, Modifying, and Deleting Code
In principle, we provide the complete code of the file you'll be modifying, including any differences. However, if the code is too long, it may be abbreviated with "//omitted" or similar.

When we instruct you to "add the following code to the file you created" or to modify the code, copy the code provided below the instruction and write it entirely into the target file.

**Example of Adding the Entire Code**

```js
import Stripe from 'stripe'

export async function POST(req, res) {
  res.status(200).end()
}  
```

In cases where we provide instructions like "Modify `app/checkout_session/routes.js` as follows," the code example will show + and - symbols on the left side to indicate additions and deletions.

**Example with Modifications**


```diff js
export async function POST(req, res) {
-  return res.status(405).end()
+  res.status(200).end()
}  
```

Please remove the lines marked with - from the original code.
The lines marked with + are the new code to be added. Use the code before and after as a guide to where the additions should be placed.

Note that on Zenn, when you copy the code that includes `+` and `-` symbols, these symbols will be excluded during pasting.

## License
This material is licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0) License.
https://creativecommons.org/licenses/by/4.0/

Within the scope of the license, you are free to reproduce or modify the material, including for commercial use.

We would be delighted if you could leave a comment on the following page about the events where you used this material:

https://zenn.dev/stripe/scraps/508964ca5e6117

Any feedback will be greatly appreciated and will help us improve our materials in the future.