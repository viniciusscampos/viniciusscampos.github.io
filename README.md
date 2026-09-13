# viniciusscampos.dev

Personal blog built with [Hugo](https://gohugo.io/) and the
[Hextra](https://github.com/imfing/hextra) theme.

## Requirements

- Hugo Extended 0.146.0 or newer
- Go 1.21 or newer

## Local preview

```sh
hugo server --buildDrafts
```

The first run downloads Hextra through Hugo Modules.

## Production build

```sh
hugo --gc --minify
```

## Writing

Add posts to `content/blog/` with `title` and `date` in the front matter.

## Deployment

Pushes to `main` are built and deployed to GitHub Pages by the workflow in
`.github/workflows/hugo.yaml`. The Pages deployment source must be set to
**GitHub Actions** in the repository settings.
