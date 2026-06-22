---
name: rails-i18n
description: "Correctly manage internationalization in Ruby on Rails apps. Use this skill whenever working on Rails views, controllers, helpers, mailers, forms, validations, flash messages, page titles, navigation, button labels, error messages, or any feature that adds or changes user-facing text. Be strict: when touching Rails UI code, verify there are no hardcoded user-facing strings left in the changed area, add proper locale keys, and run or recommend i18n verification with i18n-tasks when available."
---

# Rails I18n Management

Use this skill to keep Rails user-facing text fully internationalized. The goal is not only to add translations for new strings, but to prevent regressions: changed Rails UI code should not leave visible English or other natural-language strings directly in Ruby, ERB, component templates, mailers, JavaScript rendered by Rails, or specs that assert UI text without translation awareness.

## Core Workflow

1. Identify every user-facing string in the files you touch.
2. Replace visible strings with Rails translation calls.
3. Add or update locale entries for every supported locale when enough information exists.
4. Preserve the project's existing locale organization; if there is no clear convention, prefer Rails lazy lookup keys scoped to the view/controller path.
5. Verify lookups, interpolation, pluralization, YAML validity, and missing/unused keys before finishing.
6. Report any translations that need human review instead of inventing production-quality copy in languages you cannot verify.

## What Counts As User-Facing

Treat these as requiring i18n:

- ERB/Haml/Slim text nodes, headings, labels, placeholders, buttons, links, titles, empty states, tooltips, ARIA labels, meta titles, and confirmation prompts.
- Controller and mailer text such as `flash`, `notice`, `alert`, redirects with messages, e-mail subjects, and error messages.
- Helper, decorator, presenter, ViewComponent, partial, Turbo Stream, and Rails-rendered JavaScript strings.
- Active Record model names, attribute names, validation messages, enum labels, status labels, and form option labels.
- Test expectations for translated UI, when the expected text is part of the user experience.

Do not translate purely internal strings such as log messages, exception class names, debug output, metric names, CSS classes, DOM ids, test fixture identifiers, or API machine values unless they are shown to users.

## Rails Translation Patterns

Use the Rails helpers already available in the context:

- In views, helpers, controllers, and mailers, prefer `t(".key")` for local text scoped to the current template or action.
- Use absolute keys for shared text, model labels, navigation, or text reused across multiple templates.
- In model or service objects where `t` is not available, use `I18n.t("scope.key")`.
- For dates and times, use `l(value, format: :name)` or `I18n.l(value, format: :name)` rather than formatting strings manually.
- For numbers, currency, percentages, and distances, prefer Rails number helpers or locale-aware formatting rather than assembling symbols and values manually.

Default to lazy lookup when adding view/controller-local copy:

```erb
<h1><%= t(".title") %></h1>
<%= link_to t(".new_user"), new_user_path %>
```

```yaml
en:
  users:
    index:
      title: "Users"
      new_user: "New user"
```

Use explicit shared keys when the same text is reused:

```erb
<%= button_tag t("actions.save") %>
```

```yaml
en:
  actions:
    save: "Save"
```

## Interpolation And Grammar

Do not concatenate translated fragments or assume English word order. Put the sentence structure in the locale file and pass variables through interpolation.

Prefer this:

```erb
<%= t(".price", price: number_to_currency(@product.price)) %>
```

```yaml
en:
  products:
    show:
      price: "Price: %{price}"
```

Avoid this:

```erb
<%= t(".currency") + @product.price.to_s %>
```

When a count affects grammar, use pluralization keys and pass `count:`:

```erb
<%= t(".results", count: @results.size) %>
```

```yaml
en:
  searches:
    show:
      results:
        zero: "No results"
        one: "1 result"
        other: "%{count} results"
```

For languages with different plural rules, do not assume English `one`/`other` is enough unless the project only supports English-compatible locales.

## HTML Safety

Avoid `raw`, `html_safe`, and string-built HTML for translated copy. Rails marks keys ending in `_html` or named `html` as HTML safe in views while still escaping interpolated user values.

Prefer this:

```erb
<%= t(".terms_html", link: link_to(t(".terms_link"), terms_path)) %>
```

```yaml
en:
  signups:
    new:
      terms_html: "By continuing, you agree to the %{link}."
      terms_link: "terms of service"
```

Only pass safe HTML values that you intentionally generated, such as `link_to`. Never mark user-provided content as safe to make a translation work.

## Locale Files

First inspect the project convention:

- Existing files under `config/locales/`.
- `config.i18n.load_path` customizations.
- `config/i18n-tasks.yml` routing and search configuration.

If there is no clear convention, use keys that mirror Rails paths and actions, e.g. `users.index.title`, `admin.reports.show.empty_state`, or `mailers.invitation_mailer.invite.subject`.

Keep YAML valid and easy to review:

- Quote strings containing `%{...}`, `:`, leading/trailing spaces, or YAML-sensitive values.
- Quote keys such as `true`, `false`, `on`, `off`, `yes`, and `no`.
- Keep interpolation variable names consistent across locales.
- Do not duplicate the same key in multiple locale files unless the project deliberately uses isolated sidecar files.

If adding a new locale file, remember Rails loads `.rb` and `.yml` files in `config/locales` by default, but nested directories require the app's load path to include them. Check before placing files in new subdirectories.

## Active Record, Active Model, And Errors

Use Rails' built-in translation scopes for model and validation text:

```yaml
en:
  activerecord:
    models:
      user:
        one: "User"
        other: "Users"
    attributes:
      user:
        email: "Email address"
    errors:
      models:
        user:
          attributes:
            email:
              blank: "Enter an email address"
```

Use `Model.model_name.human` and `Model.human_attribute_name(:attribute)` when rendering model names and attribute labels outside Rails form helpers.

For non-Active Record models that include Active Model, use `activemodel` scopes instead of `activerecord`.

## Verification

Before finishing, verify both code and locale data.

1. Search the changed files for hardcoded user-facing strings.
2. Confirm every new translation key exists in the base locale and any required target locales.
3. Confirm interpolation variables in code match the locale value placeholders.
4. Confirm pluralized keys include the needed forms for the supported locales.
5. Confirm HTML translations use `_html` or `.html` keys and do not rely on unsafe `raw`/`html_safe` shortcuts.
6. Run the project's normal tests if relevant.

If `i18n-tasks` is available, prefer these checks:

```bash
bundle exec i18n-tasks health
bundle exec i18n-tasks missing
bundle exec i18n-tasks unused
bundle exec i18n-tasks normalize
```

Use `normalize` only when the project expects locale files to be auto-sorted or when the user requested cleanup. It can rewrite YAML formatting.

If `i18n-tasks` is not installed but the project uses Ruby Bundler, consider recommending:

```ruby
gem "i18n-tasks", "~> 1.1", group: :development
```

Then initialize configuration with:

```bash
cp $(bundle exec i18n-tasks gem-path)/templates/config/i18n-tasks.yml config/
```

For modern Rails apps, consider enabling the Prism Rails scanner in `config/i18n-tasks.yml` when static detection misses controller, nested method, model name, or `human_attribute_name` usage:

```yaml
search:
  prism: "rails"
```

Use `i18n-tasks-use` hints for dynamic keys that static analysis cannot infer. Prefer reducing dynamic keys when a stable explicit key is practical.

## Handling Missing Information

Do not silently fabricate translations for languages you do not know. If the app has target locales and you cannot produce reliable copy, add clear placeholders only if the project uses that workflow, or ask the user. In final notes, call out exactly which locale entries need translator review.

When the task scope is small, still perform a strict audit of nearby changed files. If unrelated hardcoded strings are widespread, fix the changed area and report the broader issue rather than rewriting the entire application without approval.

## Final Response Checklist

Summarize:

- Which user-facing strings were internationalized.
- Which locale keys/files were added or changed.
- Which verification commands ran and their results.
- Any remaining translations or dynamic keys needing human review.
