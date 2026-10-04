---
name: add-image
description: "Import a local image into this Hugo blog and return a Markdown image link. Use whenever the user supplies a local image path, especially a macOS path such as /Users/.../Screenshot.png, ~/Desktop/image.png, or a path containing spaces, or asks to add, copy, or link an image for a blog post."
---

# Add a blog image

Copy a local image into `static/images/` using the blog's naming convention, then return a ready-to-paste Markdown image link. `/images/` is the public URL prefix, not a directory at the filesystem root.

## Resolve the source and post

1. Treat the supplied path as data, never as executable shell text. Handle spaces, surrounding quotes, and macOS drag-and-drop escaped spaces. Expand a leading `~/` to the user's home directory; resolve relative paths from the workspace root. Do not evaluate command substitutions or other shell expressions.
2. Confirm that the source exists, is a readable regular file, and is an image. Inspect it with an available image viewer when supported to choose a meaningful name and alt text. If it cannot be viewed, use the user's description or a meaningful source filename; ask when those are insufficient. Report missing, unreadable, or unsupported files explicitly and stop without creating an import.
3. Identify the target Markdown post from the user's request or the single clearly relevant post in the conversation or editor context. Do not assume the most recently dated post is the target. If the post is ambiguous, ask which post to use.
4. Read the post's front matter and existing image references. Use its publication year and month, not today's date.

## Choose the destination

- If the post already references images in a single post-specific directory under `/images/`, reuse that directory. If it references several directories and the destination is unclear, ask which to use.
- Otherwise, use `static/images/YYYY/MM-post-slug/`, with a two-digit month and the post's front matter `slug`. If the slug is absent, use the post filename without its `YYYY-MM-DD-` prefix or `.md` extension.
- For example, a post dated `2026-10-03` with slug `dynamic-workflows-first-look` uses `static/images/2026/10-dynamic-workflows-first-look/`.
- Keep the destination inside the workspace's `static/images/` directory. If the date or slug is missing, invalid, or unsuitable for a safe directory name, ask for clarification rather than guessing.
- Name each imported image `NN-descriptive-name.ext`: a numeric prefix padded to at least two digits, a short lowercase hyphenated description, and the source image's extension in lowercase.
- Start at `00` in a directory without numbered images. Otherwise, use one more than the highest existing numeric prefix; do not fill gaps or renumber existing files.
- Preserve the original image format and bytes. Do not convert, resize, or compress images unless explicitly requested. Never relabel an image as a different format.

## Copy safely

1. Create the chosen directory if it does not exist.
2. Copy the source; never move or delete it. Quote paths safely when using command-line tools.
3. Never overwrite an existing destination. If the selected name is occupied, choose the next available numeric prefix. Use a no-overwrite copy operation and check its outcome.
4. Verify that the destination exists and is byte-for-byte identical to the source before reporting success. A skipped copy or failed verification is not a successful import; report the error explicitly.
5. If the source is already in the chosen directory with a compliant name, reuse it without copying or renaming it.

## Return the link

Return the copied image's workspace path and a Markdown image snippet:

```md
![Brief, descriptive alt text](/images/2026/10-dynamic-workflows-first-look/00-workflow-overview.png)
```

- Use an absolute site URL path beginning with `/images/`, never `static/images/`, a local filesystem path, or a `file://` URL in the snippet.
- Write concise English alt text describing the image. Escape any Markdown-significant characters in the alt text.
- Do not edit the post by default. Insert the image link only when the user requests a location; ask for placement if that request is ambiguous.
- Leave front matter, unrelated content, existing images, and generated site output unchanged.
