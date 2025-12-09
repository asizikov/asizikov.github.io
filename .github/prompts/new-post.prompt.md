---
agent: agent
---

Scafold a new blog post.

If the user provided a theme (typically one short sentense), use it as a starter.

You need to create a new entry in the /content/blog directory. 

File name format: 

YYYY-MM-DD-title-with-dashes.md

Use the current date unless the user specified a different date.

The content of the post should follow this structure:

```md
---
title: "Short title of the post"
date: <date in YYYY-MM-DD format>
slug: "short-title-with-dashes"
tags: [one or two relevant tags]
draft: false
---
```

Do not write anything outside the front matter. Exept the Disclosurer at the end.

```
## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
```

Title, description, and tags should be relevant to the post content. Derive them from the post content.

All text should be in English.

We use a bit playful and informal tone in our blog posts.
Check the existing posts front matters for reference, do not introduce new tags if you can use an existing one.