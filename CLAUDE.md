# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A Claude Code plugin providing two skills for the Drupal Gutenberg ecosystem. There is no build system, test suite, or runtime code — the deliverables are Markdown knowledge files consumed by Claude Code at prompt time.

## Plugin Architecture

```
.claude-plugin/plugin.json    ← Plugin manifest (name, version, description)
skills/
  drupal-gutenberg-markup/    ← Skill 1: content authoring (block markup generation)
    SKILL.md                  ← Entry point; YAML frontmatter has name/version/description
    block-syntax-reference.md
    common-blocks.md
    patterns-and-nesting.md
  drupal-gutenberg-dev/       ← Skill 2: module development (custom blocks for Drupal)
    SKILL.md                  ← Entry point
    module-structure.md
    javascript-build.md
    block-registration.md
    server-side-rendering.md
    media-integration.md
```

- **`SKILL.md`** is the entry point for each skill. Its YAML frontmatter `description` field controls auto-detection — Claude Code matches user intent against this text to decide when to load the skill.
- Supporting `.md` files in each skill directory are referenced by `SKILL.md` and provide detailed examples and rules.
- **`plugin.json`** must have `name` in kebab-case and `version` in semver.

## Drupal Gutenberg Domain Knowledge

All content targets **Drupal Gutenberg 3.x** (bundles [Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/), 27 Sep 2023 / WordPress 6.4), compatible with **Drupal 10 and 11**.

Critical Drupal-specific divergences from WordPress that content must reflect:
- Block prefix is `wp:` (not `drupal:`), but the surrounding infrastructure is all Drupal
- `.gutenberg.yml` uses `libraries-edit:`, `libraries-view:`, `dynamic-blocks:` (not `libraries:` or `blocks: render: true`)
- Translations: `const __ = Drupal.t;` (not `wp.i18n.__`)
- Library dependency: `gutenberg/edit-node` (not `gutenberg/editor`)
- `ServerSideRender` lives at `wp.serverSideRender` default export (not `wp.components`)
- MediaUpload returns Drupal objects (`media.alt_text`, `media.media_entity.id`)
- Build tooling: `drupal-js-build` + `drupal-gutenberg-translations` (not `@wordpress/scripts`)
- List blocks require `<!-- wp:list-item -->` inner-block format
- Pattern APIs still use `__experimental` prefix in 3.x

## Editing Guidelines

- When editing skill content, verify all code examples against the patterns above — incorrect examples (e.g., using WordPress APIs instead of Drupal equivalents) are the primary defect vector.
- Keep examples compact and copy-pasteable.
- The README.md mirrors key facts from the skills; keep it in sync if skill content changes.
