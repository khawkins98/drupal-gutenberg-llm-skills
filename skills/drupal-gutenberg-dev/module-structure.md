# Module structure

## File layout

A typical custom Gutenberg block module:

```
my_custom_block/
├── my_custom_block.info.yml          # Drupal module definition
├── my_custom_block.libraries.yml     # CSS/JS asset declarations
├── my_custom_block.gutenberg.yml     # Gutenberg config (libraries, dynamic blocks)
├── my_custom_block.module            # PHP hooks (required for dynamic blocks)
├── package.json                      # npm build dependencies
├── .babelrc                          # Babel configuration
├── src/
│   └── my-block.es6.js              # ES6/JSX source (edit/save functions)
├── js/
│   └── my-block.js                  # Compiled output (referenced in .libraries.yml)
├── css/
│   └── edit.css                     # Editor-only styles (optional)
├── js/theme/
│   └── my-block-frontend.js         # Front-end JS for hydration (optional)
└── templates/
    └── gutenberg-block--my-custom-block--my-block.html.twig  # SSR template (optional)
```

## `.info.yml`

Standard Drupal module info file:

```yaml
name: My Custom Block
type: module
description: 'A custom Gutenberg block for My Site.'
core_version_requirement: ^10 || ^11
package: Custom
dependencies:
  - gutenberg:gutenberg
```

## `.libraries.yml`

Declares CSS and JS assets via Drupal's Libraries API. Define separate libraries for editor and front-end assets:

```yaml
# Editor library — loaded only in Gutenberg edit screens
block-edit:
  js:
    js/my-block.js: {}
  css:
    theme:
      css/edit.css: {}
  dependencies:
    - gutenberg/edit-node

# Front-end library — loaded on node view (optional)
block-view:
  js:
    js/theme/my-block-frontend.js: {}
```

`gutenberg/edit-node` pulls in everything needed for block registration (blocks API, editor components, element, components, data store). You rarely need individual sub-libraries.

### Library dependencies

Common Gutenberg library dependencies:

| Library | Purpose |
|---------|---------|
| `gutenberg/edit-node` | All editor APIs; the standard dependency for editor JS |
| `gutenberg/blocks` | Block registration API (`wp.blocks`) — included by `edit-node` |
| `gutenberg/components` | UI components (`wp.components`) — included by `edit-node` |
| `gutenberg/element` | React wrapper (`wp.element`) — included by `edit-node` |
| `core/drupal` | Drupal JS API (includes `Drupal.t` for translations) |
| `core/drupalSettings` | Server-to-client settings |

> **Note:** In real-world Drupal Gutenberg modules, `gutenberg/edit-node` is the standard dependency. The older `gutenberg/editor` also works but `edit-node` is preferred as it matches what the Gutenberg module actually loads for the edit screen.

### Attaching libraries

Libraries in `libraries-edit:` and `libraries-view:` in `.gutenberg.yml` are attached automatically. You can also attach them manually in Twig or hooks:

```twig
{{ attach_library('my_custom_block/block-view') }}
```

```php
function my_custom_block_page_attachments(array &$attachments) {
  $attachments['#attached']['library'][] = 'my_custom_block/block-view';
}
```

## `.gutenberg.yml`

The central configuration file for a Gutenberg module. Registers libraries, dynamic blocks, and configures editor behavior.

### Library attachment

`libraries-edit:` loads on the Gutenberg edit screen; `libraries-view:` loads on the front-end node view:

```yaml
# Libraries loaded in the Gutenberg editor
libraries-edit:
  - my_custom_block/block-edit

# Libraries loaded on node view (front-end)
libraries-view:
  - my_custom_block/block-view
```

### Client-side blocks (JS-only)

Blocks that save their HTML output (non-dynamic) only need their editor library listed. No `blocks:` key required. The block is registered in JavaScript via `registerBlockType()`:

```yaml
libraries-edit:
  - my_custom_block/block-edit
```

### Dynamic blocks (server-side rendered)

Blocks rendered via PHP/Twig are declared under `dynamic-blocks:`:

```yaml
libraries-edit:
  - my_custom_block/block-edit

dynamic-blocks:
  my-custom-block/hero-banner: {}
  my-custom-block/featured-content: {}
```

Each key under `dynamic-blocks:` is the block name (namespace/block-name). `{}` is the default; you can pass configuration options if needed.

### Full example

```yaml
# Editor JS/CSS
libraries-edit:
  - my_custom_block/block-edit

# Front-end JS/CSS
libraries-view:
  - my_custom_block/block-view

# Server-rendered blocks
dynamic-blocks:
  my-custom-block/hero-banner: {}
  my-custom-block/featured-content: {}

# Custom block categories
custom-categories:
  - slug: my-site
    title: "My Site Blocks"
```

> **Note:** The namespace in `dynamic-blocks:` uses hyphens (`my-custom-block`), not underscores. This matches the block name registered in JavaScript.

### Theme `.gutenberg.yml`

Themes can also have `.gutenberg.yml` to register patterns and configure the editor:

```yaml
# Configure allowed blocks per content type
content-type-config:
  article:
    allowed-blocks:
      - core/paragraph
      - core/heading
      - core/image
      - core/list
      - my_custom_block/hero-banner

# Block patterns (see skills/drupal-gutenberg-markup/patterns-and-nesting.md)
__experimentalBlockPatternCategories:
  - name: my-patterns
    label: "My Patterns"

__experimentalBlockPatterns:
  - name: mytheme/intro-section
    title: "Intro Section"
    categories: ["my-patterns"]
    content: |
      <!-- wp:heading {"level":2} -->
      <h2>Section Title</h2>
      <!-- /wp:heading -->
      <!-- wp:paragraph -->
      <p>Introduction text here.</p>
      <!-- /wp:paragraph -->
```

## `package.json`

`drupal-js-build` is the standard build tool, mirroring Drupal core's JS compilation. It compiles `.es6.js` files to `.js`:

```json
{
  "name": "my_custom_block",
  "version": "1.0.0",
  "scripts": {
    "start": "cross-env NODE_ENV=production drupal-js-build watch --css",
    "build": "cross-env NODE_ENV=production drupal-gutenberg-translations && drupal-js-build --css"
  },
  "devDependencies": {
    "cross-env": "^7.0.3",
    "drupal-gutenberg-translations": "^1.1.0",
    "drupal-js-build": "^1.0.0"
  }
}
```

`drupal-gutenberg-translations` extracts translatable strings. `--css` enables CSS compilation alongside JS.

### Alternative: manual Babel

For simpler setups, use Babel directly:

```json
{
  "name": "my_custom_block",
  "version": "1.0.0",
  "scripts": {
    "build": "babel src --out-dir js",
    "start": "babel src --out-dir js --watch"
  },
  "devDependencies": {
    "@babel/cli": "^7.0.0",
    "@babel/core": "^7.0.0",
    "@babel/preset-env": "^7.0.0",
    "@babel/preset-react": "^7.0.0"
  }
}
```

## `.babelrc`

```json
{
  "presets": [
    ["@babel/preset-env", {
      "modules": false
    }],
    "@babel/preset-react"
  ]
}
```

`"modules": false` prevents Babel from converting ES6 imports to `require()` calls, which don't work in browsers. Gutenberg APIs are on `wp.*` globals, accessed directly.

## `.module` File (Required for dynamic blocks)

For dynamic blocks rendered entirely server-side via Twig (declared under `dynamic-blocks:` in `.gutenberg.yml`), the `.module` file registers theme hooks and provides preprocess logic. Client-side blocks that save HTML in `save()` or rehydrate via front-end JS do not need this. `hook_theme()` is required to enable block-specific preprocess hooks:

```php
<?php

/**
 * @file
 * Contains my_custom_block.module.
 */

/**
 * Implements hook_theme().
 *
 * Registers Gutenberg block templates for block-specific preprocessing.
 */
function my_custom_block_theme() {
  return [
    'gutenberg_block__my_custom_block__hero_banner' => [
      'base hook' => 'gutenberg_block',
    ],
  ];
}

/**
 * Implements hook_preprocess_gutenberg_block__MODULE__BLOCK().
 */
function my_custom_block_preprocess_gutenberg_block__my_custom_block__hero_banner(&$variables) {
  // Add variables for Twig templates.
}
```

See [server-side-rendering.md](server-side-rendering.md) for full SSR patterns including the `is_visible` guard, cache tags, and `drupalSettings` context injection.
