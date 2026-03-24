---
name: drupal-gutenberg-markup
version: 1.1.0
description: >
  Generate correct Drupal Gutenberg HTML block markup. Use when the user asks to
  create, edit, or compose Drupal Gutenberg content, HTML blocks, block patterns,
  page layouts, or any content that will be stored in Gutenberg's block format.
---

# Drupal Gutenberg Block Markup

Generate valid Drupal Gutenberg block markup (the HTML + comment format stored in `node__body.body_value`).

> **Version target:** Drupal Gutenberg 3.x, which bundles [Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/) (27 Sep 2023) / WordPress 6.4. Compatible with Drupal 10 and 11.

## Rules

1. **Use the `wp:` prefix** — Drupal Gutenberg retains the WordPress `wp:` prefix for block compatibility. Never use `drupal:` or `gb:`.
2. **Three block formats exist:**
   - **Wrapper blocks**: `<!-- wp:blockname -->` ... content ... `<!-- /wp:blockname -->`
   - **Self-closing dynamic blocks**: `<!-- wp:blockname {"attr":"val"} /-->`
   - **Blocks with attributes**: `<!-- wp:blockname {"key":"value"} -->` ... content ... `<!-- /wp:blockname -->`
3. **Attributes are JSON** in the opening comment — compact, no trailing comma.
4. **Closing comments** use a forward slash prefix: `<!-- /wp:blockname -->`.
5. **Self-closing blocks** use a forward slash before `-->`: `<!-- wp:blockname /-->`.
6. **Nested blocks** are placed between the opening wrapper HTML tag and the closing comment of the parent block.
7. **Custom/namespaced blocks** use `wp:namespace/block-name` format.

## Before returning markup

Self-check every piece of generated markup against this list:

1. Every `<!-- wp:blockname -->` has a matching `<!-- /wp:blockname -->`
2. Block names use the `wp:` prefix (never `drupal:` or `gb:`)
3. JSON attributes are compact single-line with no trailing commas
4. Closing comments have no attributes (`<!-- /wp:heading -->`, not `<!-- /wp:heading {"level":3} -->`)
5. List blocks use `<!-- wp:list-item -->` inner blocks (not raw `<li>` inside `<!-- wp:list -->`)
6. Column layouts: each `<!-- wp:column -->` is nested inside `<!-- wp:columns -->`
7. HTML tags match the block type (`<p>` for paragraph, `<h2>` for `{"level":2}` heading, `<h3>` for `{"level":3}`, etc.)
8. Custom/namespaced blocks use `wp:namespace/block-name` format
9. Self-closing syntax (`<!-- wp:blockname /-->`) is only used for dynamic/server-rendered blocks with no saved HTML content

## Quick reference

### Paragraph
```html
<!-- wp:paragraph -->
<p>Your text here.</p>
<!-- /wp:paragraph -->
```

### Heading
```html
<!-- wp:heading {"level":2} -->
<h2>Section Title</h2>
<!-- /wp:heading -->
```

### Image
```html
<!-- wp:image {"sizeSlug":"large"} -->
<figure class="wp-block-image size-large">
    <img src="/sites/default/files/image.jpg" alt="Description" />
</figure>
<!-- /wp:image -->
```

### Two-column layout
```html
<!-- wp:columns -->
<div class="wp-block-columns">
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:paragraph -->
        <p>Left column content.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:paragraph -->
        <p>Right column content.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Video embed (YouTube)
```html
<!-- wp:embed {"url":"https://www.youtube.com/watch?v=VIDEO_ID","type":"rich","providerNameSlug":"youtube","responsive":true,"className":"wp-embed-aspect-16-9 wp-has-aspect-ratio"} -->
<figure class="wp-block-embed is-type-rich is-provider-youtube wp-block-embed-youtube wp-embed-aspect-16-9 wp-has-aspect-ratio">
    <div class="wp-block-embed__wrapper">
        https://www.youtube.com/watch?v=VIDEO_ID
    </div>
</figure>
<!-- /wp:embed -->
```

## Page archetypes

Two main page types exist:

1. **Index/reference pages** -- Card grids, 5-column book card rows, full-width content. Use the page header with TOC sidebar pattern.
2. **Publication landing pages** -- Narrative flow for annual reports, GAR, key publications. Uses centered reading column (15/70/15), sticky navigation bar, truncated story previews with read-more links to PDF, stats cards, quote highlights. See "Publication landing page archetype" in patterns-and-nesting.md.

## Detailed references

For full examples and syntax rules, see these files in this skill's directory:

- **block-syntax-reference.md** -- block grammar, attribute types, serialization rules
- **common-blocks.md** -- copy-paste examples for every common block type
- **patterns-and-nesting.md** -- nested blocks, column layouts, block patterns, `.gutenberg.yml` pattern definitions, centered reading columns, sticky navigation, publication page archetype
