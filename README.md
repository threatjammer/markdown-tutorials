# markdown-tutorials repo

Threatjammer tutorials in markdown format.

## How to use this repo

This repo is a collection of markdown files that are used to generate the user and developer guides of [ThreatJammer.com](https://threatjammer.com). These files will be included in the website under the `/tutorials`folder.

## File structure

Each file in this repo is a markdown file with the following structure:
- `metadata`: contains the information about the file, such as the title, description, author, etc. It also contains the different paths y `slugs` that are used to generate the page and connect to other content.
- `content`: contains the actual content of the file.
- Reference to an image file of PNG format and 800x400 pixels.

### Metadata
It should follow the following structure:
```
---
title: TITLE OF THE PAGE (MANDATORY)
excerpt: SHORT DESCRIPTION OF THE PAGE (MANDATORY)
coverImage: URL OF THE COVER IMAGE
created: DATE OF CREATION (MANDATORY)
updated: DATE OF UPDATION (MANDATORY)
navigation:
  github: LINK TO THE BLOB OF THE FILE ON GITHUB (MANDATORY)
  home: LINK TO THE HOME PAGE OF THE DOCS (MANDATORY)
  previous: LINK TO THE PREVIOUS PAGE IN THE SAME SECTION (OPTIONAL)
  next: LINK TO THE NEXT PAGE IN THE SAME SECTION (OPTIONAL)
authors: (AT LEAST ONE AUTHOR)
  - name: NAME OF THE AUTHOR (MANDATORY)
    link: LINK TO THE AUTHOR'S GITHUB PROFILE, IF ANY (OPTIONAL)
ogImage:
  url: URL OF THE IMAGE TO USE AS THE OG IMAGE (OPTIONAL)
---
```

The developer should populate the mandatory fields always, and try to populate the optional ones if possible.

### Content

The content can be a standard Markdown format, or a [Github Flavored Markdown](https://github.github.com/gfm/) syntax. The developer should use the Github syntax if possible, as it will be easier to maintain and to use.

When linking to other pages in the same section, the developer should use the `slug` obtained removing the suffix `.md` from the file name. For example, the file `introduction-user-api.md` should be linked to the page `/tutorials/introduction-user-api`.

When linking to external resources, the developer should use `https://` if possible.

## Folder structure

The only mandatory file is `index.md`, which is the home page of the tutorials. This file should reference ALL the other files in the repo and link to them if they are the first page of a tutorial. Example:

```
+- index.md
  +- what-is-threat-jammer.md
  +- introduction-user-api.md
  +- introduction-user-api-1.md
  +- introduction-user-api-2.md
  +- introduction-user-api-3.md
  +- introduction-report-api.md
  +- introduction-report-api-1.md
  +- introduction-report-api-2.md
```

In this example the `what-is-threat-jammer` page is a single file with the content of the tutorial. The other pages are the different parts of the tutorial split in different files.

## Pull requests

You can fork this repo and create a pull request to fix errors or add new content.

## Licensing

The code is licensed under the [MIT license](/LICENSE).