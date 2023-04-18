---
title: "Code&command snippets (Astro init -> CF Pages deploy)"
---


## Setup

```bash
% npm create astro@latest
```

```bash
╭─────╮  Houston:
│ ◠ ◡ ◠  Keeping the internet weird since 2021.
╰─────╯

 astro   v2.3.0 Launch sequence initiated.
```

```
   dir   Where should we create your new project?
         astro-stripe-cf-pages
```


```
  tmpl   How would you like to start your new project?
         ● Include sample files (recommended)
         ○ Use blog template 
         ○ Empty 
```

```
 ██████  Template copying...
```

```
  deps   Install dependencies? (recommended)
         ● Yes  ○ No 
```


```
  next   Liftoff confirmed. Explore your project!

         Enter your project directory using cd ./first-astro-site 
         Run npm run dev to start the dev server. CTRL+C to stop.
         Add frameworks like react or tailwind using astro add.

         Stuck? Join us at https://astro.build/chat

╭─────╮  Houston:
│ ◠ ◡ ◠  Good luck out there, astronaut!
╰─────╯
```


## Run Astro App
```
$ npm run dev
warning package.json: No license field
$ astro dev
  🚀  astro  v2.3.0 started in 26ms
  
  ┃ Local    http://127.0.0.1:3000/
  ┃ Network  use --host to expose
  
```

![](https://storage.googleapis.com/zenn-user-upload/dcbfc315c8ab-20230418.png)

## Add Cloudflare 

If you want to use Astro for Static Site, don't do this step!

```bash
npx astro add cloudflare

  Astro will run the following command:
  If you skip this step, you can always run it yourself later

 ╭────────────────────────────────────────────╮
 │ npm i @astrojs/cloudflare astro@^2.1.4     │
 ╰────────────────────────────────────────────╯

? Continue? › (Y/n)
```

```

  Astro will make the following changes to your config file:

 ╭ astro.config.mjs ──────────────────────────────╮
 │ import { defineConfig } from 'astro/config';   │
 │                                                │
 │ import cloudflare from "@astrojs/cloudflare";  │
 │                                                │
 │ // https://astro.build/config                  │
 │ export default defineConfig({                  │
 │   output: "server",                            │
 │   adapter: cloudflare()                        │
 │ });                                            │
 ╰────────────────────────────────────────────────╯

  For complete deployment options, visit
  https://docs.astro.build/en/guides/deploy/

? Continue? › (Y/n)
```

```
✔ Continue? … yes
  
   success  Added the following integration to your project:
  - @astrojs/cloudflare
```


## Deploy to Cloudflare

@TODO Setupステップ省略


### init project

```bash
$  npx wrangler pages project create ws-astro-stripe
? Enter the production branch name: › main
```
`main`は「今作業しているブランチ」になる（はず）

```
✨ Successfully created the 'ws-astro-stripe' project. It will be available at https://ws-astro-stripe.pages.dev/ once you create your first deployment.
To deploy a folder of assets, run 'wrangler pages publish [directory]'.
```

![](https://storage.googleapis.com/zenn-user-upload/2cba2360149e-20230418.png)

### Build Astro app

```
$ npm run build

> astro-stripe-cf-pages@0.0.1 build
> astro build

14:35:40 [content] No content directory found. Skipping type generation.
14:35:40 [build] output target: server
14:35:40 [build] deploy adapter: @astrojs/cloudflare
14:35:40 [build] Collecting build info...
14:35:40 [build] Completed in 35ms.
14:35:40 [build] Building server entrypoints...
14:35:41 [build] Completed in 1.01s.

 finalizing server assets 

14:35:41 [build] Rearranging server assets...
14:35:41 [build] Server built in 1.10s
14:35:41 [build] Complete!
```

### Deploy it!

```
$  npx wrangler pages publish ./dist
▲ [WARNING] Warning: Your working directory is a git repo and has uncommitted changes

  To silence this warning, pass in --commit-dirty=true


🌎  Uploading... (2/2)

✨ Success! Uploaded 2 files (1.54 sec)

✨ Compiled Worker successfully
✨ Uploading Worker bundle
✨ Uploading _routes.json
✨ Deployment complete! Take a peek over at https://d89f694f.ws-astro-stripe.pages.dev
```