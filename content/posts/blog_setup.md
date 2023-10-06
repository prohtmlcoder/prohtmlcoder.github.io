---
title: "Blog Setup"
date: 2023-10-06T10:51:34+05:30
description: "Setup your own blog under 10 mins"
draft: false
type: "page"
tags: ["hugo","blog","github"]
---

In this blog post we will explore how to setup the a blog with [Hugo](https://gohugo.io/) and deploy it in github pages. We will use a modified version of [Hugo Ink](https://github.com/knadh/hugo-ink) theme. Please follow the instructions, I am planning to update this guide frequently. 

### Installation

Use the following commands to install the git and hugo.

```sh
sudo apt update
sudo apt install git
sudo apt install hugo
``` 
These commands will setup the git and hugo for you. 

### Setting Up

Lets create a new website with the following  command.
```
hugo new site cool_website
cd cool_website
git init
git submodule add https://github.com/174414-Rodda/hugo-ink-fork.git themes/hugo-ink-fork
```

So we are basically create a new hugo website then entered that directory. Then we initiated the git and added the submodule. So you may ask what exactly that sub module is. Well, it is more or less a set of rules for styling and stuff. This will create a new folder in `themes` with the name `hugo-ink-fork`. Now we are having a more or less working website.  Lets add content to it.

`hugo new posts/first.md`

The above command will create a file called `first.md`  under the posts directory. You can add content to the files and create more files. *Now copy the .toml in the `/themes/hugo-ink-fork/exampleSite` to the root of the website and rename with `config.toml`*.  

Well that  we more or less setup our website lets visaulize how does it looks like. In order to do so, we need a development server, fret not it comes in built  with hugo. Use the following command to spin up a local development server.

`hugo server`

 Click on the link that will shown up it will be mostly like `localhost:1313`. Please make the necessary changes as you see fit.
 
 #### Bonus
 You can add custom pages and link them in the nav bar as well. 

 `hugo create new about.md`
 
Populate the `about.md` with the stuff you like, it will be created under the content folder. In order to link it with nav bar add the following lines to the `config.toml`

```
[[menu.main]]
name = "About"
url = "/about"
weight = 2
```
`weight` is nothing but a value which defines the position, if the value is high it will be added in the end of the navbar. You can create your own custom page with whatever name you want you can even change the `url` to your own suite.

### Hosting  with Github
We use github actions to build and deploy our created website. Lets start by creating the following directory in the root directory of the website. 

```
mkdir .github && mkdir ./github/workflow
cd .github/workflow
```

Inside this create a new file with the name `hugo.yaml`. Paste the following code in it.
```
 # Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.115.4
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2

```

Create a new repository in your github repo and push this code website to the repo. Lets use the following commands to do that
```
git add .
git commit -m "first commit"
git branch -M main
git remote add origin <YOUR GITHUB REPO's HTTP URL>
git push -u origin main
```

In the previous section, we initialized the git, now we are adding files to it using `git add .` where `.` specifies to add all files. Then we are committing it with a message "first commit" and switching to the `main` branch. The last two commands were associated with pushing the repository to github. 

Hang in there, we are almost on the last leg of the deployment. Now the repository is in the go to `Settings` in your github repo and then to `pages` under it findout the `Source` and change it to github actions. Then navigate to the `actions`  where you can find out the site deployment. Check out [this](https://www.youtube.com/shorts/Kq28yBigDYw) youtube short for better understanding. 

Thats it fellas, you have your own blog post up and running. 

### References

- [JustAGuyLinux's Video on setting up Hugo](https://www.youtube.com/watch?v=s1O-8zhPQmU&pp=ygUbanVzdCBhIGd1eSBsaW51eCBodWdvIHNldHVw)

- [Hugo's QuickStart](https://gohugo.io/getting-started/quick-start/)

- [Hugo's Deployment](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
