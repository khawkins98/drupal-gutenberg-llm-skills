# drupal-gutenberg-llm-skills

A [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code) with skills for [Drupal Gutenberg](https://www.drupal.org/project/gutenberg), Drupal's implementation of the WordPress Gutenberg block editor.

## Skills

### `drupal-gutenberg-markup`

Generates valid Drupal Gutenberg HTML block markup. Covers `<!-- wp:blockname -->` comment syntax, attributes, nested blocks, column layouts, and block patterns.

Invoke with `/drupal-gutenberg:drupal-gutenberg-markup` or auto-detected when you ask Claude to create Gutenberg content.

Example prompts:
- "Create a Drupal Gutenberg page with a hero image, two columns, and a call-to-action button"
- "Write Gutenberg block markup for a testimonials section with three cards"
- "Convert this HTML into valid Gutenberg block markup"

### `drupal-gutenberg-dev`

Helps build custom Gutenberg blocks and modules for Drupal. Covers module structure, `.gutenberg.yml` configuration, ES6/Babel build toolchain, block registration, Twig server-side rendering, and media integration.

Invoke with `/drupal-gutenberg:drupal-gutenberg-dev` or auto-detected when you're working on Gutenberg module code.

Example prompts:
- "Create a custom Gutenberg block module that displays a featured content carousel"
- "Set up server-side rendering for my custom block with a Twig template"
- "What's the Twig template naming convention for Drupal Gutenberg?"

## Installation

### For development/testing

```bash
claude --plugin-dir /path/to/drupal-gutenberg-llm-skills
```

### As an installed plugin

```bash
claude plugin add /path/to/drupal-gutenberg-llm-skills
```

## Technical details

These skills target **Drupal Gutenberg 3.x** (bundles [Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/) (27 Sep 2023) / WordPress 6.4). Compatible with Drupal 10 and 11.

- Blocks use the `wp:` prefix (not `drupal:`), keeping WordPress block compatibility
- Content is stored as HTML in `node__body.body_value`
- Custom blocks are declared in `.gutenberg.yml` (not WordPress's `block.json`)
- Server-side rendering uses Twig templates named `gutenberg-block--namespace--block-name.html.twig`
- Dynamic blocks need `hook_theme()` registration and block-specific preprocess hooks (`hook_preprocess_gutenberg_block__MODULE__BLOCK`) — this is only for blocks rendered entirely server-side via Twig, not client-side blocks that save HTML or rehydrate via JS
- ES6 source files use the `.es6.js` extension, compiled via Babel to `.js`
- Start by copying `gutenberg/modules/example_block` as boilerplate

## Resources

- [Drupal Gutenberg module](https://www.drupal.org/project/gutenberg)
- [Drupal Gutenberg documentation](https://www.drupal.org/docs/contributed-modules/gutenberg)
- [Create Custom Blocks guide](https://www.drupal.org/docs/contributed-modules/gutenberg/create-custom-blocks)
- [WordPress Block API reference](https://developer.wordpress.org/block-editor/developers/block-api/) (underlying API used by Drupal Gutenberg)
