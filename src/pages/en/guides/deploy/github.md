---
title: Deploy your Astro Site to GitHub Pages
description: How to deploy your Astro site to the web using GitHub Pages.
layout: ~/layouts/DeployGuideLayout.astro
i18nReady: true
---

You can use [GitHub Pages](https://pages.github.com/) to host an Astro website directly from a repository on [GitHub.com](https://github.com/).

## How to deploy

You can deploy an Astro site to GitHub Pages by using [GitHub Actions](https://github.com/features/actions) to automatically build and deploy your site. To do this, your source code must be hosted on GitHub.

Astro maintains the official `withastro/action` to deploy your project with very little configuration. Follow the instructions below to deploy your Astro site to GitHub pages, and see [the package README](https://github.com/withastro/action) if you need more information.

1. Set the [`site`](/en/reference/configuration-reference/#site) and, if needed, [`base`](/en/reference/configuration-reference/#base) options in `astro.config.mjs`.
    
    ```js title="astro.config.mjs" ins={4-5}
    import { defineConfig } from 'astro/config'

    export default defineConfig({
      site: 'https://astronaut.github.io',
      base: '/my-repo',
    })
    ```
    - `site` should be `https://<YOUR_USERNAME>.github.io` or `https://my-custom-domain.com`
    - `base` should be your repository’s name starting with a forward slash, for example `/my-repo`. This is so that Astro understands your website's root is `/my-repo`, rather than the default `/`.
    
    :::note
      Don't set a `base` parameter if:

    - Your repository is named `<YOUR_USERNAME>.github.io`.
    - You're using a custom domain.
    :::

    :::caution
        If you did not previously have a value for `base` set, and are only configuring this value so that you can deploy to GitHub, you must update your internal page links to now include your `base`.

    ```astro
    <a href="/my-repo/about">About</a>
    ```
    :::

2. Create a new file in your project at `.github/workflows/deploy.yml` and paste in the YAML below.

    ```yaml title="deploy.yml"
    name: Github Pages Astro CI

    on:
      # Trigger the workflow every time you push to the `main` branch
      # Using a different branch name? Replace `main` with your branch’s name
      push:
        branches: [ main ]
      # Allows you to run this workflow manually from the Actions tab on GitHub.
      workflow_dispatch:
      
    # Allow this job to clone the repo and create a page deployment
    permissions:
      contents: read
      pages: write
      id-token: write

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout your repository using git
            uses: actions/checkout@v2          
          - name: Install, build, and upload your site
            uses: withastro/action@v0

      deploy:
        needs: build
        runs-on: ubuntu-latest
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v1
    ```
    
    :::caution
    The official Astro [action](https://github.com/withastro/action) scans for a lockfile to detect your preferred package manager (`npm`, `yarn`, or `pnpm`). You should commit your package manager's automatically generated `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml` file to your repository.
    :::

3. On GitHub, go to your repository’s **Settings** tab and find the **Pages** section of the settings.

4. Choose **GitHub Actions** as the **Source** of your site and press **Save**.  

5. Commit the new workflow file and push it to GitHub.  

  
Your site should now be published! When you push changes to your Astro project’s repository, the GitHub Action will automatically deploy them for you.

:::tip[set up a custom domain]
You can optionally set up a custom domain by adding the following `./public/CNAME` file to your project: 

```txt title="public/CNAME"
sub.mydomain.com
```

This will deploy your site at your custom domain instead of `user.github.io`. Don't forget to also [configure DNS for your domain provider](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain).   
:::
