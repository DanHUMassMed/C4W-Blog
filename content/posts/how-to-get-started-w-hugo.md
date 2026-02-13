
+++
title = "How to Install and Use Hugo with GitHub Pages and GitHub Actions"
date = 2026-02-13T01:00:00-05:00
draft = false
+++

-------------------------------------------------------------------------------------------------------------------------

# How to Install and Use Hugo with GitHub Pages and GitHub Actions

Hugo is one of the fastest and most flexible static site generators available. Combined with GitHub Pages and GitHub Actions, it provides a free, automated, and professional publishing platform for technical and civic blogging.

This guide walks through the full process of setting up a Hugo site, deploying it with GitHub Actions, and maintaining it long-term.

---

## Why Use Hugo + GitHub Pages?

- ⚡ Extremely fast site generation
- 💸 Free hosting via GitHub Pages
- 🔁 Automatic deployment on every push
- 📝 Content managed like source code
- 🔓 Fully open-source toolchain
- 📱 Mobile-friendly and theme-driven

This workflow is ideal for civic tech groups, open-data projects, and technical blogs.

---

## Prerequisites

Before starting, install Hugo and ensure you have Git and GitHub access.

### Install Hugo

#### macOS

```bash
brew install hugo
```

#### Windows

```bash
choco install hugo-extended
```

#### Linux (Recommended: Manual Install)

```bash
sudo apt install hugo
```

> Note: Linux package managers often ship outdated versions. For modern themes like PaperMod, you may need to download a newer release from:
> [https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)

Verify installation:

```bash
hugo version
```

---

## Step 1: Create a New Hugo Site

Create your project locally:

```bash
hugo new site my-civic-blog
cd my-civic-blog
git init
```

This creates the basic Hugo directory structure.

---

## Step 2: Add the PaperMod Theme

PaperMod is a fast, modern theme suitable for technical blogging.

Add it as a Git submodule:

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Configure Hugo to use it.

Open `hugo.toml` or `config.toml` and add:

```toml
theme = "PaperMod"
```

---

## Step 3: Create Your First Post

Create a new post:

```bash
hugo new posts/my-first-article.md
```

Open the file in `content/posts/`.

You will see front matter like:

```toml
+++
date = '2026-02-13T10:21:08-05:00'
draft = true
title = 'How to Get Started with Hugo'
+++
```

### Important: Publish Your Post

For production, drafts are NOT published.

Change:

```toml
draft = true
```

To:

```toml
draft = false
```

Or remove it entirely.

If you forget this, your post will not appear on GitHub Pages.

---

## Step 4: Preview Your Site Locally

Run the development server:

```bash
hugo server -D
```

Open:

```
http://localhost:1313
```

The `-D` flag shows draft posts locally.

---

## Step 5: Push Your Site to GitHub

Create a new repository on GitHub:

* Name: `my-hugo-site` (or similar)
* Do NOT initialize with README

Then connect and push:

```bash
git add .
git commit -m "Initial Hugo site"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

---

## Step 6: Set Up GitHub Actions Deployment

GitHub Actions will build and deploy your site automatically.

### Create Workflow Folder

```bash
mkdir -p .github/workflows
```

Create:

```
.github/workflows/hugo.yml
```

---

### Add Deployment Workflow

Paste the following:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: [main]

  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      HUGO_VERSION: 0.155.3

    steps:
      - name: Install Hugo
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
          https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build Site
        env:
          HUGO_ENVIRONMENT: production
        run: |
          hugo --minify \
               --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    runs-on: ubuntu-latest
    needs: build

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy
        id: deployment
        uses: actions/deploy-pages@v4
```

---

### Important: Hugo Version

Many themes require newer Hugo.

This line is critical:

```yaml
env:
  HUGO_VERSION: 0.155.3
```

If your build fails, check your theme’s minimum version.

---

## Step 7: Enable GitHub Pages

In your repository:

1. Go to **Settings → Pages**
2. Under **Source**, select:
   **GitHub Actions**
3. Save

GitHub will now use your workflow.

---

## Step 8: Publishing Workflow

From now on, your workflow is:

### 1. Write

```bash
hugo new posts/new-post.md
```

Edit and set:

```toml
draft = false
```

---

### 2. Commit

```bash
git add .
git commit -m "Add new post"
```

---

### 3. Deploy

```bash
git push origin main
```

Within ~30 seconds, your site updates automatically.

---

## Common Problems and Fixes

### Post Not Showing Up

Check:

* `draft = false`
* Date is not in the future
* File is in `content/posts/`

---

### Hugo Version Errors

Error example:

```
Hugo v0.146+ required
```

Fix:

Update `HUGO_VERSION` in workflow.

---

### Theme Not Loading

Run:

```bash
git submodule update --init --recursive
```

Then commit.

---

### Build Fails in CI Only

Run locally:

```bash
hugo --verbose
```

Compare output.

---

## Using Data with Hugo (Civic Tech Tip)

Hugo supports structured data.

Place files in:

```
/data/
```

Example:

```
data/businesses.json
data/permits.csv
```

You can then render them in templates to build:

* Tables
* Dashboards
* Reports

This is ideal for civic data storytelling.

---

## Optional: Custom Domain

You can attach a domain like:

```
blog.codeforworcester.org
```

In:

Settings → Pages → Custom Domain

GitHub provides free HTTPS.

---

## Recommended Repository Structure

```
my-civic-blog/
├── content/
│   └── posts/
├── themes/
│   └── PaperMod/
├── static/
├── data/
├── hugo.toml
└── .github/workflows/hugo.yml
```

---

## Conclusion

By combining Hugo, GitHub Pages, and GitHub Actions, you get:

* Professional publishing
* Zero hosting cost
* Automated deployment
* Version-controlled content
* Long-term stability

This setup is well suited for civic organizations, open data projects, and technical communities that value transparency and sustainability.

Once configured, it requires minimal maintenance and scales effortlessly.

Happy publishing. 🚀

---
