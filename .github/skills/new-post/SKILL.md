---
name: new-post
description: Scaffold a new Hugo blog post in this repository. Use when the user asks to create a new post, start a blog draft, or scaffold a post from a topic.
---

# Scaffold a new blog post

If the user provided a theme (typically one short sentence), use it as a starter.

Create a new entry in the `content/blog/` directory.

File name format:

`YYYY-MM-DD-title-with-dashes.md`

Use the current date unless the user specified a different date.

The content of the post should follow this structure:

```md
---
title: "Short title of the post"
date: <date in YYYY-MM-DD format>
slug: "short-title-with-dashes"
tags: [one or two relevant tags]
draft: true
---
```

Do not write anything outside the front matter except the disclosure at the end.

```md
## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
```

Title, slug, and tags should be relevant to the provided topic.

All text should be in English.

We use a slightly playful and informal tone in our blog posts.
Check the existing posts' front matter for reference. Do not introduce new tags if you can use an existing one.