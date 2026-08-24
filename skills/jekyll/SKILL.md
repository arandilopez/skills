---
name: jekyll
description: "Create, run, maintain, and customize Jekyll and JekyllRB sites with the Jekyll CLI. Use this skill whenever the user mentions Jekyll or JekyllRB and wants to create or edit posts or drafts, publish content, run the local server in the background, build or troubleshoot a site, change _config.yml, manage _data files, layouts, includes, collections, or themes, or scaffold a new Jekyll site or gem-based theme."
---

# Jekyll Site Management

Use Jekyll's standard structure and Bundler commands.
Preserve the site's established conventions before adding new ones.

## Start With Inspection

Before changing content or configuration:

1. Read `Gemfile`, `_config.yml` or `_config.toml`, and relevant existing files.
2. Identify the site structure, theme, plugins, collections, and existing front matter conventions.
3. Use Bundler when the site has a `Gemfile`:

```bash
bundle exec jekyll <command>
```

Use bare `jekyll <command>` only when the project does not use Bundler.
Do not edit generated output such as `_site/`, `.jekyll-cache/`, or `.sass-cache/`.

## Commands

Use the smallest Jekyll command that verifies the requested work:

```bash
bundle exec jekyll new site-name
bundle exec jekyll new site-name --blank
bundle exec jekyll build
bundle exec jekyll serve
bundle exec jekyll clean
bundle exec jekyll doctor
bundle exec jekyll help <command>
```

`build` produces the site in `_site/` by default.
`serve` rebuilds on source changes and serves the site locally.
`clean` removes generated output and caches.
Run `doctor` for deprecations or common configuration problems.

### Run The Development Server In The Background

First check whether a server is already running and use its port if appropriate.
When a background process is supported, start it with a log file and save its PID:

```bash
mkdir -p tmp
bundle exec jekyll serve --livereload > tmp/jekyll.log 2>&1 &
echo $! > tmp/jekyll.pid
```

Report the local URL from the server log.
To stop only the server started for this task:

```bash
kill "$(cat tmp/jekyll.pid)"
```

Do not assume live reload is installed or available.
If `--livereload` fails, start `bundle exec jekyll serve` without it.
Use a non-default port only when needed:

```bash
bundle exec jekyll serve --port 4001
```

Configuration is loaded when Jekyll starts.
Restart `serve` after changes to `_config.yml` or `_config.toml`.

## Posts, Drafts, And Publishing

### Create A Post

Put posts in `_posts/` with the required filename format:

```text
YYYY-MM-DD-title.md
```

Use the site's existing front matter keys and layout.
If there is no clear convention, start with:

```markdown
---
layout: post
title: "Descriptive title"
date: 2026-08-23 10:00:00 +0000
categories: []
tags: []
---

Post content.
```

Choose the intended publication date and timezone. Do not invent categories, tags, authors, images, or layouts.
Build the site after creating or editing the post.

### Create And Preview A Draft

Put drafts in `_drafts/` without a date prefix:

```text
_drafts/my-draft.md
```

Preview drafts without publishing them:

```bash
bundle exec jekyll serve --drafts
```

Jekyll uses a draft file's modification time as its date during preview.

### Publish A Draft

Move the draft from `_drafts/` to `_posts/` and add the publication date to its filename:

```text
_drafts/my-draft.md
_posts/2026-08-23-my-draft.md
```

Set or update front matter only when the site's conventions require it.
Then run a build and check the generated URL.

### Preview Non-Public Content

Use these switches only for local review:

```bash
bundle exec jekyll serve --future
bundle exec jekyll serve --unpublished
bundle exec jekyll serve --drafts
```

Future-dated posts are excluded by default.
Posts with `published: false` are excluded by default.
Do not enable these options for a production build unless the user explicitly asks.

## Configuration And Content Maintenance

Keep site-wide settings in `_config.yml` or `_config.toml`.
Use front matter for page- and post-specific settings.

When changing configuration:

1. Change the smallest relevant setting.
2. Preserve YAML or TOML formatting and existing key order when practical.
3. Check `url`, `baseurl`, `permalink`, `theme` or `remote_theme`, `plugins`, `collections`, `defaults`, `exclude`, and `include` when they affect the request.
4. Restart the development server and run a clean build when the change affects output selection or paths.

Use front matter defaults to avoid repeated metadata across a path or collection.
Do not duplicate the same setting into each file when a suitable scoped default already exists.

### Data Files

Keep reusable structured content in `_data/` as YAML, JSON, CSV, or TSV.
CSV and TSV files need a header row.
A file such as `_data/team.yml` is available as `site.data.team`.
Subdirectories create namespaces, for example `_data/people/authors.yml` is available through `site.data.people.authors`.

Validate data syntax after editing.
Avoid duplicate data files with the same basename and different extensions because they map to the same data name.

## Layouts, Includes, And Themes

Inspect existing `_layouts/`, `_includes/`, assets, and the active theme before changing presentation.
Make a local override only for the theme file that needs modification:

```text
_layouts/<same-layout-name>.html
_includes/<same-include-name>.html
```

Jekyll prefers matching local files over theme defaults.
This preserves theme updates while keeping overrides small.

### Create A New Site

Use the standard scaffold unless the user needs a minimal starting point:

```bash
jekyll new my-site
jekyll new my-site --blank
```

Enter the new directory, review the generated `Gemfile` and configuration, then build it before adding features.

### Create A Gem-Based Theme

Create a theme scaffold with:

```bash
jekyll new-theme my-theme
```

Before editing, inspect the generated gemspec, `Gemfile`, layouts, includes, Sass files, and test setup.
Keep theme-specific defaults in the theme and site-specific content in the consuming site.
Test the theme with its documented test command and with a small consuming site when possible.

## Verification

For content changes, run a production-like build:

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

For a site with a custom configuration, use its documented configuration set, for example:

```bash
JEKYLL_ENV=production bundle exec jekyll build --config _config.yml,_config.production.yml
```

If a build fails, report the relevant source error and fix the source, not generated files.
For unexplained stale output, run `bundle exec jekyll clean` and then rebuild.

## Final Response

State:

- Files created or changed.
- Whether the item is a draft, published post, or future/unpublished preview.
- Commands run and their result.
- The local URL and PID when a server was started.
- Any required restart, deployment, or content review.
