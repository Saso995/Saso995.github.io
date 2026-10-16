# Salvatore Bova — security research blog

A [Jekyll](https://jekyllrb.com/) site built on the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)
theme and deployed to GitHub Pages via GitHub Actions.

## 1. One-time setup

1. **Create the repo on GitHub.** For a personal site the repo **must** be named
   `Saso995.github.io` (e.g. `saso.github.io`). The site will live at
   `https://Saso995.github.io`.

2. **Your username is already set** to `Saso995` throughout `_config.yml`. Optionally
   fill in a few more fields there: `social.email`, your X/Twitter handle, and an
   `avatar:` (a URL or a local `/assets/img/avatar.png`).

3. **Push it:**

   ```bash
   git init -b main
   git add .
   git commit -m "Initial site"
   git remote add origin git@github.com:Saso995/Saso995.github.io.git
   git push -u origin main
   ```

4. **Turn on Pages:** on GitHub → **Settings → Pages → Build and deployment →
   Source: GitHub Actions**. The included workflow (`.github/workflows/pages-deploy.yml`)
   builds and deploys on every push to `main`. First build takes a minute or two.

## 2. Writing a post

Create a file in `_posts/` named `YYYY-MM-DD-title.md` with front matter:

```markdown
---
title: "Your title"
date: 2026-10-20 10:00:00 +0200
categories: [Security Research, CVE]
tags: [web, rce, writeup]
image:
  path: /assets/img/posts/your-post/cover.png   # optional cover
---

Your content…
```

Put images under `assets/img/posts/<your-post>/` and reference them with an absolute path
(`/assets/img/posts/<your-post>/file.png`). See the IUCLID post for a worked example.

## 3. Preview locally (optional)

Needs Ruby 3.x and Bundler:

```bash
bundle install
bundle exec jekyll serve   # http://127.0.0.1:4000
```

## 4. Comments (optional)

Chirpy supports Disqus, Utterances and Giscus. [Giscus](https://giscus.app) (GitHub
Discussions) fits a technical blog well — configure it under `comments:` in `_config.yml`.

---

Theme: [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) (MIT).
