# Troubleshooting

Symptom-based debugging for Drupal Gutenberg custom blocks. Each entry: **symptom → root cause → fix**.

> **Context:** Drupal Gutenberg 3.x (Gutenberg 16.7, `apiVersion: 2`). Many issues stem from accidentally using WordPress patterns instead of Drupal equivalents.

## Block not appearing in inserter

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| Block missing from inserter entirely | Library not listed in `libraries-edit:` in `.gutenberg.yml` | Add `- my_module/block-edit` under `libraries-edit:` |
| Block missing, no console errors | Module not enabled | `drush en my_module && drush cr` |
| Block missing, JS error in console | Error in `registerBlockType()` — wrong block name format, missing required fields, or syntax error | Open browser console, fix the reported JS error, rebuild |
| Block missing after code change | Build not run or stale cache | `npm run build && drush cr`, then hard refresh |

## "Invalid block" / block validation errors

These appear when you open a node with existing block content and the editor's `save()` output doesn't match what's stored.

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| "Invalid block" on every page load | `save()` output changed without adding a deprecation handler | Add the old `save()` to the `deprecated` array (see block-registration.md) |
| "Invalid block" after attribute changes | Attribute `source`/`selector` doesn't match saved HTML structure | Update the selector to match, or add a deprecation with `migrate()` |
| "Invalid block" on fresh blocks too | Missing `useBlockProps.save()` in `save()` function | Wrap the `save()` root element with `useBlockProps.save()` |
| `useBlockProps` returns empty object / block wrapper attributes missing | Missing `apiVersion: 2` in block registration | Add `apiVersion: 2` to the `registerBlockType()` options object |
| Intermittent "Invalid block" | JSON attributes in comment don't match attribute defaults | Ensure attribute `default` values match what `save()` renders when no value is set |

## ServerSideRender shows error or blank

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| SSR panel shows "Error loading block" | Block not listed in `dynamic-blocks:` in `.gutenberg.yml` | Add `my-module/my-block: {}` under `dynamic-blocks:` |
| SSR panel blank, no error | Twig template misnamed or in wrong directory | Verify: `templates/gutenberg-block--{namespace}--{block-name}.html.twig` |
| SSR panel shows PHP error | Exception in preprocess hook | Check `drush watchdog:show` for the error; fix the PHP code |
| `ServerSideRender is not a constructor` or similar | Imported from wrong location | Use `const ServerSideRender = wp.serverSideRender;` — it is the default export of `wp.serverSideRender`, NOT on `wp.components` or `wp.editor` |
| SSR renders but shows nothing useful | Block depends on runtime context (current node, route) | Replace `ServerSideRender` with a static placeholder in `edit()` — see server-side-rendering.md |

## Dynamic block renders in editor but not on front-end

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| Front-end shows nothing for the block | Missing `hook_theme()` registration | Register the block template in `hook_theme()` with `'base hook' => 'gutenberg_block'` |
| Template loads but output is empty | Preprocess hook never sets `is_visible = TRUE` | Verify the data-loading logic in the preprocess hook; `is_visible` defaults to `FALSE` |
| Template not discovered | Template in wrong directory | Must be in `MODULE/templates/`, filename: `gutenberg-block--{namespace}--{block-name}.html.twig` |
| Old content showing | Drupal render cache serving stale output | `drush cr` and verify cache tags are set in the preprocess hook |
| Cache tags not invalidating | Tags added in the wrong place or missing | Add `$variables['#cache']['tags'][]` in the preprocess hook for each referenced entity |

## Build and asset issues

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| JS changes don't appear | Stale build or cache | `npm run build && drush cr`, then hard refresh (Ctrl+Shift+R) |
| `Drupal.t is not a function` | Missing `core/drupal` dependency in `.libraries.yml` | Add `- core/drupal` under `dependencies:` for your editor library |
| `wp.blocks is undefined` | Missing `gutenberg/edit-node` dependency | Add `- gutenberg/edit-node` under `dependencies:` in `.libraries.yml` |
| `wp.serverSideRender is undefined` | Missing `gutenberg/edit-node` dependency | Same fix — `gutenberg/edit-node` provides all editor APIs |
| `npm run build` fails with Babel errors | Missing `.babelrc` or wrong presets | Ensure `.babelrc` has `@babel/preset-env` and `@babel/preset-react`; run `npm install` |
| Build succeeds but JS not loaded | Compiled file path doesn't match `.libraries.yml` | Check that `js/my-block.js` in `.libraries.yml` matches the actual output path |

## Media issues

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| `media.alt` returns `undefined` | Using WordPress media object shape | Drupal returns `media.alt_text` (not `media.alt`) |
| `media.id` returns wrong value | Using WordPress media ID | Drupal entity ID is at `media.media_entity.id` (not `media.id`) |
| Image sizes unavailable | Using `media.sizes` | Drupal uses `media.media_details.sizes` |
| Image URLs break after migration | Storing absolute URLs | Store entity IDs in attributes; generate URLs server-side in preprocess hooks |

## Translation issues

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| `__ is not a function` or `wp.i18n` undefined | Using WordPress i18n | Replace `const { __ } = wp.i18n;` with `const __ = Drupal.t;` |
| Strings not extracted for translation | Translation tool not run | Run `drupal-gutenberg-translations` (included in `npm run build` if using the standard `package.json` scripts) |

## Pattern and list block issues

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| Lists render with wrong structure / broken output | Raw `<li>` tags without inner blocks | Each `<li>` must be wrapped in `<!-- wp:list-item -->` / `<!-- /wp:list-item -->` |
| Patterns don't appear in inserter | Using stable Pattern API names | Gutenberg 16.7 requires `__experimentalBlockPatterns` and `__experimentalBlockPatternCategories` in `.gutenberg.yml` (not the stable names) |

## Twig template issues

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| XSS vulnerability / raw HTML in output | Using `{{ variable|raw }}` in Twig templates | Never use `|raw` on user-supplied content. Render entities via view modes in preprocess hooks and pass render arrays to Twig instead |
| Entity fields render as array debug output | Printing entity objects directly | Use `entity.label` for titles, or render via `\Drupal::entityTypeManager()->getViewBuilder()->view()` in preprocess |
| `attributes.addClass()` not working | Template missing `{{ attributes }}` on wrapper element | Ensure the outermost element includes `{{ attributes.addClass('my-class') }}` |

## General debugging steps

When none of the above match:

1. **Check the browser console** — most block issues produce a JavaScript error
2. **Check Drupal logs** — `drush watchdog:show` for PHP errors
3. **Clear everything** — `npm run build && drush cr` and hard refresh
4. **Verify `.gutenberg.yml`** — is the library in `libraries-edit:`? Is the block in `dynamic-blocks:` (if dynamic)?
5. **Compare with `example_block`** — the working example module shipped with Gutenberg is a reliable reference
6. **Check the block name** — must match between `registerBlockType('namespace/block')` in JS and the `dynamic-blocks:` key in `.gutenberg.yml`
