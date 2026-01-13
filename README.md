# KI Documentation

This repo contains the documentation for Ki languages.

The docs site is published to [Kixi Docs](https://kixi-io.github.io/Ki.Docs/)

## Set up and deployment

The docs site is build using GitHub Pages with Jekyll.  It uses the [Just the Docs](https://github.com/just-the-docs/just-the-docs) remote theme.

`_config.yml` contains the configuration for the site.

Publishing is done via GitHub settings under Pages.  It builds off of the `master` branch using the `/docs` folder as the source.

The site is built when changes are pushed to `master`.

## Branches

- `main` - main branch, contains the docs site in `/docs` folder

### Workflows

Use one of the following workflows to update content.

1. Directly edit files in `main` and commit.  Changes will be published on commit.
2. Create a feature branch off of `main` then merge the feature branch into `main` via PR.

## Editing content

Content files are in markdown format (`.md`) and are located in the `/docs` folder.

Images are placed in the `/docs/assets/images` folder.  They can be referenced in markdown files using relative paths, for example:

```markdown
![KD Schema Diagram](/assets/images/KD_Schema_Diagram.png)
```

Markdown process is using Kramdown, which is the default for Jekyll.  Kramdown support GitHub Flavored Markdown (GFM) features such as fenced code blocks, tables, and strikethrough, however it is more strict with syntax than GitHub. For example tables must have an empty line above them.  They cannot immdiately start after a line like a header.  The same with code blocks.

### Front matter

Front matter is used to set page-specific variables for Jekyll.  Each markdown file should have front matter at the top of the file enclosed in `---` lines.  At a minimum, each file should have a `title` variable.

Example front matter:

```frontmatter
---
title: Home
nav_order: 1
---
```

`nav_order` is used to set the order of pages in the left navigation.  Lower numbers appear first.  If not set the page appears at the end of the navigation in alphabetical order.

## Left navigation

The left navigation is automatically generated from the markdown files in the `/docs` folder.  The order is determined by the `nav_order` variable in the front matter of each file.

Appended to the bottom of the left nav are external links defined in `_config.yml` under `nav_external_links:`.


## Customizations

`_sass/color_schemes/wider.scss` contains a customization to adjust the width of the site.  It is used in `_config.yml` under `color_scheme: wider`.

`_sass/custom/custom.scss` contains css overrides and customizations.  It is referenced automatically by Just the Docs.

## emoji code support

The site uses the `jemoji` plugin to support emoji code like `:smile:`. It is enabled in `_config.yml` under `plugins:`.

`jemoji` does not support GitHub custom emoji codes such as `:octocat:`.
