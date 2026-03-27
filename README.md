Pavel's personal blog at https://ptimofeev.com

Built with [Zola](https://www.getzola.org/), a fast static site generator written in Rust.

## Commands

Serve locally with live reload:
```
zola serve
```

Build for production (output in `public/`):
```
zola build
```

## Structure

```
config.toml        # Site configuration
content/
  _index.md        # Homepage (paginated post list)
  about/           # About page
  posts/           # Blog posts
templates/         # Tera HTML templates
sass/              # SCSS stylesheets
static/            # Static assets (images, etc.)
```

## Adding a post

Create a new file in `content/posts/` with TOML frontmatter:

```toml
+++
title = "My Post Title"
date = 2024-01-01
description = "A short description."
path = "my-post-title"

[taxonomies]
categories = ["Category"]

[extra]
image = "/images/social/my-image.png"
+++

Post content here...

<!-- more -->

Rest of post after the fold.
```
