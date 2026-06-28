# Japan Azure Integration Support Blog

Japan Azure Integration Support Team のブログリポジトリです。

## Getting Started

[Getting Started](./docs/getting-started.md)

## Init / Update blog theme

https://github.com/jpazureid/hexo-theme-jpazure

```shell
git submodule update -i
```

## Start / Stop Hexo server (local-preview)

```shell
docker-compose up

# Ctrl+C
docker-compose down
```

## Directory structure

```
jpazinteg-blog
├── .azuredevops
│   └── pull_request_template  # PR templates
│       ├── add.md
│       └── fix.md
├── .gitignore
├── .textlintrc
├── README.md
├── _config.yml                # Site configuration
├── articles                   # Blog articles
│   └── information
│       └── test.md            # Example post
├── docker-compose.yaml        # Configuration for containers (local-preview)
├── docs                       # Documents
├── github-issue-template.md
├── pipelines                  # Azure Pipelines
│   ├── prod-pipeline.yml
│   ├── pr-pipeline.yml
│   ├── stages
│   │   ├── generate-blog.yml
│   │   └── publish-to-ghpages.yml
│   └── scripts
│       └── prepare-preview.js
├── scaffolds
├── source
└── themes                     # Blog themes
    └── jpazure (git submodule)
```
