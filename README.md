# NorthernLightsDevel Blog

Hugo source for https://northernlightsdevel.github.io running PaperMod + a small custom theme. GitHub Pages builds and deploys the site via `.github/workflows/publish.yml` every time `main` is updated, so you only need Hugo locally to preview and add content.

## Prerequisites
- [Hugo Extended](https://gohugo.io/getting-started/installing/) v0.128.0 or newer (`hugo version` should report `extended`).
- Go-standard toolchain to run Hugo binaries (bundled with the download).

## Typical Workflow
```bash
# 1. Clone and enter the repo
git clone git@github.com:NorthernLightsDevel/NorthernLightsDevel.github.io.git
cd NorthernLightsDevel.github.io

# 2. Start a preview server (rebuilds on save, includes drafts)
hugo server -D --disableFastRender
```
Visit `http://localhost:1313` while writing; stop the server with `Ctrl+C`.

## Add a Post
```bash
# Create content/content/posts/my-new-post.md from archetypes/default.md
hugo new posts/my-new-post.md
```
Next steps:
1. Open the generated file in `content/posts/`.
2. Fill in the front matter (`title`, `date`, optional `description`, `tags`).
3. Change `draft = true` to `false` when you are ready to publish so GitHub Pages includes it (drafts are excluded from the production build).

## Useful Commands
- `hugo server -D` – Run the live-reload server, including drafts. Drop `-D` to preview the production view.
- `hugo --minify` – Build the site into `public/` (ignored by git); handy to sanity-check before committing.
- `hugo list drafts` – Show any posts that are still marked as drafts.

## Deploy
```bash
git add content/posts/my-new-post.md
git commit -m "Add new post"
git push origin main
```
Pushing triggers the “Deploy Hugo site to Pages” workflow, which runs `hugo --minify` and publishes the `public/` output to GitHub Pages automatically—no manual upload required.
