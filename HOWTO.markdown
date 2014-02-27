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

# Editing

```
git checkout source
```

## Create a new post:

```
rake new_post[TITLE]
```

# Preview

Preview the site on http://localhost:4000:

```
rake preview
```

# Deploy

Deploy to GitHub pages:

```
rake push
```
