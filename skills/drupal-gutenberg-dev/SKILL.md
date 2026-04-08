---
name: drupal-gutenberg-dev
version: 1.1.0
description: >
  Develop custom Gutenberg blocks and modules for Drupal. Use when the user is
  creating, modifying, or debugging Drupal Gutenberg modules, custom blocks,
  JavaScript source files, PHP integration code, Twig templates, or build
  configuration for the Drupal Gutenberg ecosystem.
---

# Drupal Gutenberg development

Build custom Gutenberg blocks and modules for Drupal.

> **Version target:** Drupal Gutenberg 3.x, which bundles [Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/) (27 Sep 2023) / WordPress 6.4. Compatible with Drupal 10 and 11.

## Getting started

The quickest way to a custom block is copying the `example_block` module included with Gutenberg:

1. Copy `gutenberg/modules/example_block` to `modules/custom/my_block`
2. Rename all `example_block.*` files to `my_block.*`
3. Rename `package.example.json` to `package.json`
4. Update all `example_block` references in `.gutenberg.yml`, `.info.yml`, `.libraries.yml`, and `package.json`
5. Run `npm install`
6. Enable the module: `drush en my_block`
7. Run `npm run build` to compile ES6 to JS
8. Run `npm start` for watch mode during development

## Development workflow

Follow these steps in order when building a block. Choose the workflow that matches your block type.

### Client-side blocks (save HTML)

Use this when the block saves its own HTML output — no server-side rendering needed.

1. **Scaffold module structure** — copy `gutenberg/modules/example_block` or create `MODULE.info.yml`, `MODULE.libraries.yml`, `MODULE.gutenberg.yml`, `package.json`, `.babelrc`, and `src/` directory
2. **Configure `.gutenberg.yml`** — add your editor library under `libraries-edit:`
3. **Configure `.libraries.yml`** — declare the compiled JS with `gutenberg/edit-node` as a dependency
4. **Register block in `.es6.js`** — implement `registerBlockType()` with `edit()` and `save()` functions; use `useBlockProps()` / `useBlockProps.save()`
5. **Build** — `npm install && npm run build`
6. **Enable** — `drush en MODULE && drush cr`
7. **Verify** — run through the [verification checklist](#verification-checklist) below

### Dynamic blocks (server-rendered via Twig)

Use this when block output is rendered by PHP/Twig — the block saves no HTML (`save()` returns `null`).

1. **Scaffold module structure** — same as client-side, plus create `MODULE.module` and `templates/` directory
2. **Configure `.gutenberg.yml`** — add editor library under `libraries-edit:` AND add the block under `dynamic-blocks:`
3. **Configure `.libraries.yml`** — declare the compiled JS with `gutenberg/edit-node` as a dependency
4. **Register block in `.es6.js`** — implement `registerBlockType()` with `edit()` function; `save()` returns `null`
5. **Choose editor preview approach:**
   - `ServerSideRender` — if the block can render without page context (use `const ServerSideRender = wp.serverSideRender;`)
   - Static placeholder — if the block depends on runtime context (current node, route data)
6. **Register theme hook** — in `MODULE.module`, implement `hook_theme()` with `'base hook' => 'gutenberg_block'`
7. **Create Twig template** — in `templates/gutenberg-block--{namespace}--{block-name}.html.twig`; wrap output in `{% if is_visible %}`
8. **Implement preprocess hook** — block-specific `hook_preprocess_gutenberg_block__MODULE__BLOCK()` with data loading, `is_visible` guard, and cache tags
9. **Build** — `npm install && npm run build`
10. **Enable** — `drush en MODULE && drush cr`
11. **Verify** — run through the [verification checklist](#verification-checklist) below

## Verification checklist

After implementing a block, verify each item before considering the task complete:

- [ ] Block appears in the Gutenberg inserter
- [ ] No JavaScript errors in browser console
- [ ] Edit controls (InspectorControls, RichText, etc.) work correctly
- [ ] Save and reload the node: no "Invalid block" warning
- [ ] `npm run build` completes without errors
- [ ] `drush cr` and hard refresh show latest changes
- [ ] **Client-side blocks:** `save()` output produces valid HTML markup
- [ ] **Dynamic blocks:** `ServerSideRender` preview renders (or placeholder displays)
- [ ] **Dynamic blocks:** front-end renders correctly
- [ ] **Dynamic blocks:** cache tags invalidate when referenced entities change

## Troubleshooting

If something isn't working, see [troubleshooting.md](troubleshooting.md) for symptom-based debugging organized by category (block not appearing, validation errors, SSR issues, build problems, media, translations, and more).

## File types

| File | Purpose |
|------|---------|
| `my_block.info.yml` | Standard Drupal module info |
| `my_block.libraries.yml` | Declares CSS/JS assets via Libraries API |
| `my_block.gutenberg.yml` | Registers custom blocks, categories, patterns |
| `src/*.es6.js` | ES6 source files (edit/save functions, block registration) |
| `dist/*.js` | Compiled JavaScript output |
| `templates/*.html.twig` | Twig templates for server-side rendered blocks |

## Differences from WordPress

| Aspect | WordPress | Drupal |
|--------|-----------|--------|
| Content storage | Post meta | HTML in `node__body.body_value` |
| Block declaration | `register_block_type()` + `block.json` | `.gutenberg.yml` + Twig |
| SSR | `render_callback` | Twig templates + hooks |
| Assets | `wp_enqueue_script/style()` | Libraries API (`.libraries.yml`) |
| Build | `@wordpress/scripts` (webpack) | Babel/drupal-js-build (`.es6.js`) |
| Media | Native media library | Drupal file entities + optional media module |

## Installation

```bash
composer require 'drupal/gutenberg:^3.0'
drush en gutenberg
```

Then enable Gutenberg for specific content types at `/admin/config/content/gutenberg`.

## Block deprecation

When modifying a block's `save()` function, **always add a deprecation entry** to prevent "Attempt Block Recovery" warnings on existing content.

Gutenberg compares stored HTML against the current `save()` output. A mismatch — even a single added class — triggers a validation warning on every existing instance of that block. The `deprecated` API tells Gutenberg how to recognize old markup and silently migrate it.

### Steps

1. **Before changing `save()`**, copy the current `save` function and `attributes` object into a versioned constant:
   ```javascript
   const v1 = {
     attributes: { /* current attributes, before your change */ },
     save({ attributes }) {
       // Current save() body, before your change
     },
   };
   ```
2. Add the constant to the block's `deprecated` array: `deprecated: [v1]`
3. If attribute shapes changed, include a `migrate(attributes)` function that maps old to new
4. Stack deprecations in reverse chronological order: `deprecated: [v2, v1]`
5. **Test:** load a page with existing instances — no recovery warning should appear

### When to add a deprecation

- New or removed CSS classes in save output
- Changed wrapper elements or HTML structure
- Added, removed, or renamed attributes that affect save output
- Changed default attribute values that alter the rendered HTML

### When deprecation doesn't help

- Core Gutenberg blocks whose markup changed due to a Drupal Gutenberg module upgrade — these are outside your control
- Blocks broken by translation workflows or content import

### Drupal Gutenberg i18n note

Drupal Gutenberg replaces `wp.i18n` with a `Drupal.t()` wrapper (`gutenberg/js/i18n.js`). The vendored i18n filter hooks (`i18n.gettext`) and `setLocaleData()` are inoperative. To override Gutenberg UI strings, monkey-patch `wp.i18n.__` directly.

### Reference

- [WordPress Block Deprecation API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-deprecation/)

## Detailed references

See these files in this skill's directory:

- **module-structure.md** -- file structure, `.info.yml`, `.libraries.yml`, `.gutenberg.yml`
- **javascript-build.md** -- ES6 to ES5 workflow, Babel, webpack, React/JSX
- **block-registration.md** -- registering blocks, categories, variations, block styles
- **server-side-rendering.md** -- Twig templates, hooks, dynamic blocks, `ServerSideRender`
- **media-integration.md** -- file entities, media library, media blocks, upload handling
- **troubleshooting.md** -- symptom-based debugging for common block development issues
