---
layout: post
title: Building this site in Github Pages
date: 2018-02-24 11:44:00 +0000
---
The following are instructions to build this site. Largely it is an exercise in both verifying the steps for myself and in using the site building tools to create and publish this post.

These instructions are based on building a user or organisation site, rather than a project site.

## Create initial site

* Create Repository: `kemitix.github.io`
* Browse to `https://github.com/kemitix/kemitix.githubo/settings`
* Scroll down to *GitHub Pages*
* Tick *Enforce HTTPS*

## Create local Jekyll project

```bash
sudo apt install ruby-dev
sudo gem install jekyll bundler
jekyll new kemitix.github.io
cd kemitix.github.io
```

## Configure So Simple theme

Add plugin to *Gemfile*:

```yaml
gem "jekyll-remote-theme"`
```

Install plugin:

```bash
bundle install
```

Enable plugin and select theme in *_config.yml*:

```yaml
plugins:
  - jekyll-remote-theme
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-gist
remote_theme: mmistakes/so-simple-theme
```

## Review site locally

```bash
bundle exec jekyll serve --incremental
```

Open browser: `http://localhost:4000/`

## Publish to github

```bash
git init
git add .
git commit -m"Initial commit"
git push -u origin master
```

* Open browser: `https://kemitix.github.io/`

## Refs

* http://pages.github.com/
* https://jekyllrb.com/docs/installation/
