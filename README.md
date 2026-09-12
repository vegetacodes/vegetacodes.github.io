# vegetacodes.github.io

My personal blog, built with [Hugo](https://gohugo.io/) using the [Hyde](https://github.com/spf13/hyde) theme, deployed to GitHub Pages.

## Prerequisites

- [Hugo (extended version)](https://gohugo.io/installation/), v0.111.3 or newer
  ```sh
  brew install hugo
  ```
- Git

## Installation

Clone the repository along with its submodules (the `hyde` theme is a git submodule):

```sh
git clone --recurse-submodules https://github.com/vegetacodes/vegetacodes.github.io.git
cd vegetacodes.github.io
```

If you already cloned the repo without `--recurse-submodules`, initialize the theme submodule with:

```sh
git submodule update --init --recursive
```

## Usage

### Run the local dev server

```sh
hugo server -D
```

- `-D` includes draft content.
- The site will be available at `http://localhost:1313/`.
- The server live-reloads on content, layout, and static asset changes.

### Create a new post

```sh
hugo new posts/my-new-post.md
```

This scaffolds a new file under `content/posts/` using the archetype in [archetypes/default.md](archetypes/default.md). Set `draft: false` in the front matter when it's ready to publish.

### Build for production

```sh
hugo --minify
```

Generates the static site into the `public/` directory (ignored by git).

## Project Structure

```
config.toml           # Site configuration (menus, params, services)
content/               # Markdown content (posts, about page)
layouts/               # Site-specific template overrides
  _default/            # Overrides for base/single templates
  partials/            # Overrides for theme partials (e.g. sidebar)
  shortcodes/          # Custom shortcodes
static/                # Static assets (CSS, icons, images)
themes/hyde/           # Hyde theme (git submodule)
```

Files under `layouts/` take precedence over the same-named files in `themes/hyde/layouts/`, so theme behavior is customized without modifying the submodule directly.

## Deployment

Pushing to `main` triggers the [`Deploy Hugo site to Pages`](.github/workflows/hugo.yml) GitHub Actions workflow, which builds the site with Hugo and publishes it to GitHub Pages automatically. No manual deployment steps are needed.

