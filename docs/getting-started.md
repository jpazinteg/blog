> NOTE: This document is adapted from [cssjpn/blog-example](https://github.com/cssjpn/blog-example/blob/main/docs/getting-started.md)

# Getting Started

This guide covers getting started with the `jpazinteg-blog`.

## Installation

In this _Getting Started_, you need `git`, `docker` and `docker-compose`.

### Git

* Windows: Download & install [git](https://git-scm.com/download/win).
* Mac: Install it with [Homebrew](https://brew.sh/), [MacPorts](http://www.macports.org/) or [installer](http://sourceforge.net/projects/git-osx-installer/).
* Linux (Ubuntu): `sudo apt-get install git-core`

### Docker

* Windows / Mac
  * https://www.docker.com/products/docker-desktop
* Linux (Ubuntu)
  * `docker`: https://docs.docker.com/engine/install/ubuntu/
  * `docker-compose`: https://docs.docker.com/compose/install/

Make sure the following commands are successful:

```shell
$ git
$ docker
$ docker-compose
```

## Setup

### Cloning a repository

```shell
# Move to your working directory
$ cd {YOUR_WORKING_DIR}

# Cloning a blog repository
$ git clone git@github.com:jpazinteg/blog.git
$ cd blog
```

### Initialize blog theme

Blog theme (.css, .js, etc...) is configured as git submodule.

```shell
# Initialize and Update themes
$ git submodule update -i
```

### Customize site configuration file

You can configure most settings in `_config.yml`.

## Writing blog articles

```shell
# Create new branch
$ git checkout -b add_article

# Create or Edit articles
$ vim articles/information/test.md
```

### Start / Stop local-preview server

You can start a local-preview server. This is at `http://localhost:4000/`.

```shell
# Run server
$ docker-compose up
```

To stop server, press `Ctrl+C` and `docker-compose down`.

```shell
$ docker-compose down
```

## Publish Blog

### With Azure Pipelines (Recommended)

When new changes are merged to `main` branch via Pull Request, the Azure Pipelines production pipeline will:

1. Build the Hexo blog (generate static HTML/CSS/JS)
2. Push the generated files to the `gh-pages` branch on GitHub

The PR pipeline will also:
- Build the blog for validation
- Generate an HTML preview of changed articles
- Post a link to the preview in the PR discussion

### With GitHub Actions (Legacy)

The repository also has a GitHub Actions workflow (`.github/workflows/upload-gh-pages.yml`) that can be used as a fallback.

### Configure GitHub Pages

Navigate **Settings** > **Pages** on GitHub. Select `gh-pages` branch as source and click **Save**.

Now you can access your blog at: `https://jpazinteg.github.io/blog/`
