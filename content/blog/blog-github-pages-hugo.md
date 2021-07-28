---
title: "Creating a blog for free using GitHub Pages and Hugo"
date: 2021-06-21T10:55:13+01:00
draft: true
tags: ["Blog", "Hugo", "Github Pages", "Github", "Pages"]
showToc: true
---

Welcome :wave:

In this blog post, I will be showing you how to create a blog for **free** using GitHub Pages and Hugo.

I will walk you through the entire process of creating the GitHub repository (where your blog will live), creating your Hugo site, adding a theme for your blog :art:, creating your first blog post and automating the publishing of the blog!

# What is GitHub Pages and Hugo?

[GitHub Pages](https://pages.github.com/) allows you to create a website which is hosted directly from a repository on GitHub.

[Hugo](https://gohugo.io/) is a fast and highly customisable static site generator.

# Prerequisites

You're going to need a couple of things before you start creating your blog:

1. A [GitHub](https://github.com/signup) account.

2. [Git](https://docs.github.com/en/get-started/quickstart/set-up-git#setting-up-git).

    - Follow the instructions for installing Git, setting up your username, commit email address and caching your GitHub credentials using a credential helper.

3. Hugo.

    - I recommend installing the **extended** version of Hugo as some themes require it.

    - Follow the instructions for [Windows](https://gohugo.io/getting-started/installing/#windows) or [Linux](https://gohugo.io/getting-started/installing/#binary-cross-platform).

    > Most likely your Operating System will be 64-bit architecture. So you would download: `hugo_extended_{version}_Windows-64bit.zip`

    > If you have [choco](https://chocolatey.org/) installed, run the following command to install Hugo: `choco install hugo-extended -y`

# Step 1 - Creating and cloning the GitHub repository

Create a [new](https://github.com/new) **public** GitHub repository named *username.github.io*. Where *username* is your GitHub username. For example, if your GitHub username was *bumblebee*, then you would enter *bumblebee.github.io*.

Enter the following command in a terminal to clone the repository to your machine (providing your GitHub username instead of *username*): `git clone https://github.com/username/username.github.io.git`

Now, create a new branch using the following command: ` cd username.github.io.git; git checkout -b source`

**For the rest of this post, substitute *username* for your GitHub username.**

# Step 2 - Initalising your Hugo site

Run the following command to initalise your site: `hugo new site -f yml .`

You should see the following output:

`Congratulations! Your new Hugo site is created in /path/to/your/hugo/site/username.github.io.`

# Step 3 - Adding and configuring a site theme 🎨

Next, go to https://themes.gohugo.io/ and browse the list of themes that are available. There are a lot... :smile:

Many themes provide a demo so you can see for yourself whether you like a theme. Furthermore, many themes have their own configuration so make sure you read the documentation.

For this blog post, I'm going to use the [Tania](https://themes.gohugo.io/hugo-tania/) theme.

Following the installation instructions, run the command: `git submodule add https://github.com/WingLim/hugo-tania themes/hugo-tania` to install the theme.

In the site's root directory (`username.github.io`), open the `config.yml` file and paste the [Tania theme's configuration](https://raw.githubusercontent.com/WingLim/hugo-tania/main/exampleSite/config.yaml). Edit it to your liking and save the file.

> **NOTE**: Make sure you edit `baseurl: "https://example.com"` to `baseurl: "https://username.github.io"` - Remembering to substitute *username* for your GitHub username!

Additional configuration for the Tania theme can be found [here](https://github.com/WingLim/hugo-tania#configuration).

The Tania theme also requires an `articles.md` file to be created if you want to have an archive page of blog posts as shown in the [demo site](https://hugo-tania.netlify.app/articles/).

Create the `articles.md` file inside the site's content directory (`username.github.io/content`), paste the following block into it and save the file:

```
---
title: Articles
# Edit subtitle and date!
subtitle: Posts, tutorials, snippets, musings, and everything else.
date: 2020-11-26
type: section
layout: "archives"
---
```

# Step 4 - Creating your first blog post :pencil2:

Now it's time to create your first blog post!

Run the following command to create the blog post file (changing `your-blog-post` to the name of the post): `hugo new post/your-blog-post.md`

You should see the following output verifying that a markdown file was created for the post: `username.github.io/content/post/your-blog-post.md created`.

Next, open the file, write your post's content and save the file.

To see your site rendered run the command: `hugo server -D` and paste the following URL into your browser: `http://localhost:1313/`

> **NOTE**: Once you have finished writing your post, set `draft: true` at the top of the file to `draft: false`

# Step 5 - Automating the publishing process ⚙️

To automate the publishing process of your blog, create a `gh-pages.yml` file located at: `username.github.io/.github/workflows/gh-pages.yml`, paste the following block into the file and save the file:

```yaml
name: Github Pages

on:
  push:
    branches:
      - source

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/source'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: main
          publish_dir: ./public
```

# Step 6 - Push the site's content to the GitHub repository

Finally, you need to push all the site's content to the GitHub repository. To do this, run the following commands from the root directory (`username.github.io`) of the site:

1. `git add -A`

2. `git commit -m "Publishing my first blog post."`

3. `git push`

# Step 7 - Configuring GitHub Pages for your GitHub repository

Go to https://github.com/username/username.github.io/settings/pages and perform the following steps:

1. Make sure the source branch is set to **main**.

2. Make sure the source folder is set to **/ (root)**.

3. Press **Save**.

4. Enable **Enforce HTTPS** so your blog is served over HTTPS.

If all goes well the automated workflow will trigger and your site will be published at: `https://username.github.io` :tada:

Enjoy! :smile:
