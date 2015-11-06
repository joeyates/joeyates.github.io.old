# Repository

## Remotes

* octopress - imathis/octopress
* origin - joeyates/joeyates.github.io

## Branches:

On GitHub:

* origin/source
 * Octopress and raw blog posts
 * Checked out in root directory
* origin/master
 * The compiled site
 * Checked out in _deploy

# Configuration

* _config.yml
* layout: source/_layouts/default.html

# Editing

```
git checkout source
```

## Create a new post

`rake new_post[TITLE]`

## Static Pages

For an URL foo/bar:

```
mkdir -p foo/bar
```

Edit foo/bar/index.md:

```
---
title: The Title
layout: default
---
...
```

# Preview

Preview the site on http://localhost:4000:

```
rake preview
```

# Deploy

Deploy to GitHub pages:

```
rake gen_deploy
```

Manually:

```
rake generate        # generates in 'public'
mkdir -p _deploy
rake deploy
git checkout master
cp -r _deploy/* .
git add .
```
