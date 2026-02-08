# Server-side rendering

Dynamic blocks in Drupal Gutenberg are rendered server-side using Twig templates, using Drupal's theme system. Declared under `dynamic-blocks:` in `.gutenberg.yml`.

## Declaring a dynamic block

### `.gutenberg.yml`

```yaml
libraries-edit:
  - my_module/block-edit

dynamic-blocks:
  my-module/my-block: {}
```

### JavaScript (edit only)

For server-rendered blocks, `save` returns `null`. All output comes from the Twig template:

```js
const { registerBlockType } = wp.blocks;
const { createElement: el } = wp.element;
const { useBlockProps, InspectorControls } = wp.blockEditor;
const { PanelBody, TextControl, SelectControl } = wp.components;
const ServerSideRender = wp.serverSideRender;
const __ = Drupal.t;

registerBlockType('my-module/my-block', {
  apiVersion: 2,
  title: __('My Block'),
  icon: 'admin-site',
  category: 'common',
  attributes: {
    heading: {
      type: 'string',
      default: '',
    },
    displayMode: {
      type: 'string',
      default: 'full',
    },
  },

  edit({ attributes, setAttributes }) {
    const blockProps = useBlockProps();

    return el('div', blockProps,
      el(InspectorControls, {},
        el(PanelBody, { title: __('Block Settings') },
          el(TextControl, {
            label: __('Heading'),
            value: attributes.heading,
            onChange: (val) => setAttributes({ heading: val }),
          }),
          el(SelectControl, {
            label: __('Display Mode'),
            value: attributes.displayMode,
            options: [
              { label: __('Full'), value: 'full' },
              { label: __('Compact'), value: 'compact' },
            ],
            onChange: (val) => setAttributes({ displayMode: val }),
          })
        )
      ),
      el(ServerSideRender, {
        block: 'my-module/my-block',
        attributes: attributes,
      })
    );
  },

  save() {
    return null;
  },
});
```

`ServerSideRender` calls a Drupal route that renders the block via the Twig template and returns the HTML for live preview in the editor.

> **Note:** In Drupal Gutenberg 3.x (Gutenberg 16.7), `ServerSideRender` is the default export of `wp.serverSideRender`. It is **not** on `wp.components`. The legacy re-export `wp.editor.ServerSideRender` also works but is deprecated.

### Placeholder editor preview

When a block depends on runtime context that doesn't exist during editing (e.g., the current node, a taxonomy term from the URL, or external API data), `ServerSideRender` can't render meaningful output. Use a static placeholder in the `edit()` function instead:

```js
edit({ attributes }) {
  const blockProps = useBlockProps();

  return el('div', blockProps,
    el(InspectorControls, {},
      el(PanelBody, { title: __('Settings') },
        // ... attribute controls
      )
    ),
    el('div', { className: 'my-block-placeholder' },
      el('p', {}, __('Country Header')),
      el('small', {}, __('Content is rendered from the current page context.'))
    )
  );
},
```

Use this approach when:
- The block reads entity context from `drupalSettings` or the route
- `ServerSideRender` would render empty/broken output in the editor
- The block's value comes from runtime data, not stored attributes

## Theme hook registration

> **Scope:** This section applies only to dynamic blocks rendered entirely server-side via Twig (declared under `dynamic-blocks:` in `.gutenberg.yml`). Client-side blocks that save their HTML in `save()` — including those that rehydrate via front-end JS — do not need `hook_theme()` or preprocess hooks.

To use block-specific preprocess hooks (the preferred pattern), each dynamic block template must be registered in `hook_theme()`. Without this, Drupal's theme system won't recognize the block-specific template suggestion, and targeted preprocess hooks won't fire.

```php
/**
 * Implements hook_theme().
 */
function my_module_theme() {
  return [
    // Register each dynamic block template.
    // Naming: gutenberg_block__{namespace}__{block_name}
    // (slashes become __, hyphens become _)
    'gutenberg_block__my_module__featured_content' => [
      'base hook' => 'gutenberg_block',
    ],
    'gutenberg_block__my_module__hero_banner' => [
      'base hook' => 'gutenberg_block',
    ],
  ];
}
```

**Naming convention:** The theme hook name uses double underscores (`__`) as separators and underscores for hyphens. For block `my-module/featured-content`:
- Block ID: `my-module/featured-content`
- Theme hook: `gutenberg_block__my_module__featured_content`
- Template file: `gutenberg-block--my-module--featured-content.html.twig`

## Twig template naming

Templates follow this convention:

```
gutenberg-block--{namespace}--{block-name}.html.twig
```

Where:
- `{namespace}` is the module name (underscores become hyphens)
- `{block-name}` is the block name (underscores become hyphens)

### Examples

| Block ID | Template File |
|----------|---------------|
| `my_module/my-block` | `gutenberg-block--my-module--my-block.html.twig` |
| `my_module/hero_banner` | `gutenberg-block--my-module--hero-banner.html.twig` |
| `example_block/example` | `gutenberg-block--example-block--example.html.twig` |

### Template location

Place templates in the module's `templates/` directory:

```
my_module/
└── templates/
    └── gutenberg-block--my-module--my-block.html.twig
```

Themes can override these templates by placing a file with the same name in their `templates/` directory.

## Template variables

Available variables in Gutenberg block Twig templates:

| Variable | Type | Description |
|----------|------|-------------|
| `block_content` | string | The rendered inner HTML content of the block |
| `block_attributes` | object | All block attributes as defined in `registerBlockType` |
| `block_name` | string | The full block identifier (e.g., `my_module/my-block`) |
| `block_renderer` | string | Rendering context: `'ssr'` when rendered for the editor preview via `ServerSideRender`, absent on front-end |
| `attributes` | object | HTML attributes for the wrapper element |

### Example template

```twig
{# templates/gutenberg-block--my-module--my-block.html.twig #}

<div{{ attributes.addClass('my-block') }}>
  {% if block_attributes.heading %}
    <h2 class="my-block__heading">{{ block_attributes.heading }}</h2>
  {% endif %}

  <div class="my-block__content my-block--{{ block_attributes.displayMode|default('full') }}">
    {{ block_content }}
  </div>
</div>
```

> **Security:** Never use `|raw` on user-supplied content in Twig templates. To render entity fields safely, build render arrays in preprocess hooks (e.g., via `$view_builder->view($entity, 'teaser')`) and pass them to Twig as variables — Twig will render them with proper escaping.

### Differentiating editor preview from front-end

Use `block_renderer` to show different output when the block is rendered inside the editor (via `ServerSideRender`) vs. the front-end:

```twig
{% if block_renderer|default('') == 'ssr' %}
  <div class="my-block my-block--preview">
    <p>{{ block_attributes.heading|default('Preview') }}</p>
  </div>
{% else %}
  <div{{ attributes.addClass('my-block') }}>
    {# Full front-end rendering with all markup #}
  </div>
{% endif %}
```

### Accessing attributes

Block attributes from the JSON comment are available via `block_attributes`:

```twig
{# String attribute #}
{{ block_attributes.title }}

{# Boolean check #}
{% if block_attributes.showImage %}
  <img src="{{ block_attributes.imageUrl }}" alt="{{ block_attributes.imageAlt }}" />
{% endif %}

{# Default value #}
{{ block_attributes.alignment|default('left') }}

{# Array/object attribute #}
{% for item in block_attributes.items %}
  <li>{{ item.label }}</li>
{% endfor %}
```

## PHP hooks

### `hook_gutenberg_alter()`

Called after the block is assembled into a render array. Use it to modify the block before rendering:

```php
/**
 * Implements hook_gutenberg_alter().
 */
function my_module_gutenberg_alter(array &$build, $block_name, $block_attributes, $block_content) {
  if ($block_name === 'my_module/my-block') {
    // Add cache tags.
    $build['#cache']['tags'][] = 'my_custom_tag';

    // Add additional variables for the template.
    $build['#extra_data'] = _my_module_load_extra_data($block_attributes);
  }
}
```

### Block-specific preprocess hooks (preferred)

When you register templates in `hook_theme()`, you can use Drupal's standard suggestion-based preprocess hooks — one per block, no `if` branching required:

```php
/**
 * Implements hook_preprocess_gutenberg_block__MODULE__BLOCK().
 */
function my_module_preprocess_gutenberg_block__my_module__featured_content(&$variables) {
  // Default to hidden — template wraps output in {% if is_visible %}.
  $variables['is_visible'] = FALSE;

  $attrs = $variables['block_attributes'];
  $count = $attrs['count'] ?? 3;

  $nids = \Drupal::entityQuery('node')
    ->condition('type', 'article')
    ->condition('status', 1)
    ->sort('created', 'DESC')
    ->range(0, $count)
    ->accessCheck(TRUE)
    ->execute();

  if (!empty($nids)) {
    $nodes = \Drupal::entityTypeManager()
      ->getStorage('node')
      ->loadMultiple($nids);
    $variables['nodes'] = $nodes;
    $variables['is_visible'] = TRUE;

    // Cache tags — invalidate when any article changes.
    $variables['#cache']['tags'][] = 'node_list:article';
    foreach ($nodes as $node) {
      $variables['#cache']['tags'][] = 'node:' . $node->id();
    }
  }
}
```

**Key patterns:**
- **`is_visible` guard:** Default to `FALSE`, set `TRUE` only when data is valid. The Twig template wraps all output in `{% if is_visible %}` to prevent broken markup when context is missing.
- **Cache tags in preprocess:** Add `$variables['#cache']['tags'][]` directly. This is the most common location (works alongside `hook_gutenberg_alter()` cache tags).
- **Hook naming:** `hook_preprocess_gutenberg_block__{namespace}__{block}` — double underscores match the `hook_theme()` key.

> **Requires:** `hook_theme()` registration for the block (see [Theme hook registration](#theme-hook-registration) above).

### Generic `hook_preprocess_gutenberg_block()` (fallback)

Without `hook_theme()` registration, use the generic hook with conditional branching:

```php
/**
 * Implements hook_preprocess_gutenberg_block().
 */
function my_module_preprocess_gutenberg_block(&$variables) {
  $block_name = $variables['block_name'] ?? '';

  if ($block_name === 'my_module/my-block') {
    // Add computed variables.
    $variables['computed_class'] = 'block--' . str_replace('/', '-', $block_name);
  }
}
```

This works but becomes unwieldy with multiple blocks. Prefer block-specific hooks for production modules.

### `#post_render` callbacks

To transform rendered HTML after Twig processing:

```php
function my_module_gutenberg_alter(array &$build, $block_name, $block_attributes, $block_content) {
  if ($block_name === 'my_module/my-block') {
    $build['#post_render'][] = '_my_module_post_render_block';
  }
}

function _my_module_post_render_block($markup, $element) {
  // Transform the rendered HTML string.
  return str_replace('placeholder', 'actual-value', $markup);
}
```

## Injecting runtime context

Blocks that depend on entity or route context can inject data into `drupalSettings` via `hook_page_attachments()`, making it available to front-end JavaScript. Preprocess hooks access the same underlying data directly via PHP APIs like `\Drupal::routeMatch()`.

```php
/**
 * Implements hook_page_attachments().
 */
function my_module_page_attachments(array &$attachments) {
  $route_match = \Drupal::routeMatch();
  $node = $route_match->getParameter('node');

  if ($node && $node->getType() === 'country_page') {
    $attachments['#attached']['drupalSettings']['my_module'] = [
      'countryId' => $node->id(),
      'countryName' => $node->getTitle(),
    ];
  }
}
```

Access in front-end JavaScript: `drupalSettings.my_module.countryId`. In preprocess hooks, use `\Drupal::routeMatch()` or entity loading directly — `drupalSettings` is a client-side mechanism only.

Use this when blocks need:
- Front-end JS that depends on the current node or taxonomy term
- Data from external services that front-end JS needs to fetch or display
- Configuration that varies by route and must be available client-side

## Complete dynamic block example

This example demonstrates the production patterns: `hook_theme()` registration, block-specific preprocess hooks, `is_visible` guard, and cache tags.

### `.gutenberg.yml`

```yaml
libraries-edit:
  - my_module/block-edit

dynamic-blocks:
  my-module/featured-content: {}
```

### `src/featured-content.es6.js`

```js
const { registerBlockType } = wp.blocks;
const { createElement: el } = wp.element;
const { useBlockProps, InspectorControls } = wp.blockEditor;
const { PanelBody, TextControl, RangeControl } = wp.components;
const ServerSideRender = wp.serverSideRender;
const __ = Drupal.t;

registerBlockType('my-module/featured-content', {
  apiVersion: 2,
  title: __('Featured Content'),
  icon: 'star-filled',
  category: 'common',
  attributes: {
    heading: { type: 'string', default: 'Featured' },
    count: { type: 'number', default: 3 },
    contentType: { type: 'string', default: 'article' },
  },

  edit({ attributes, setAttributes }) {
    const blockProps = useBlockProps();

    return el('div', blockProps,
      el(InspectorControls, {},
        el(PanelBody, { title: __('Settings') },
          el(TextControl, {
            label: __('Heading'),
            value: attributes.heading,
            onChange: (val) => setAttributes({ heading: val }),
          }),
          el(RangeControl, {
            label: __('Number of items'),
            value: attributes.count,
            onChange: (val) => setAttributes({ count: val }),
            min: 1,
            max: 10,
          })
        )
      ),
      el(ServerSideRender, {
        block: 'my-module/featured-content',
        attributes: attributes,
      })
    );
  },

  save() {
    return null;
  },
});
```

### `templates/gutenberg-block--my-module--featured-content.html.twig`

```twig
{# templates/gutenberg-block--my-module--featured-content.html.twig #}

{% if is_visible %}
<div{{ attributes.addClass('featured-content') }}>
  {% if block_attributes.heading %}
    <h2 class="featured-content__heading">{{ block_attributes.heading }}</h2>
  {% endif %}

  <div class="featured-content__grid featured-content__grid--{{ block_attributes.count }}">
    {% for node in nodes %}
      <article class="featured-content__item">
        <h3>{{ node.label }}</h3>
        {{ node_teasers[node.id] }}
      </article>
    {% endfor %}
  </div>
</div>
{% endif %}
```

### `my_module.module`

```php
<?php

/**
 * Implements hook_theme().
 */
function my_module_theme() {
  return [
    'gutenberg_block__my_module__featured_content' => [
      'base hook' => 'gutenberg_block',
    ],
  ];
}

/**
 * Implements hook_preprocess_gutenberg_block__MODULE__BLOCK().
 */
function my_module_preprocess_gutenberg_block__my_module__featured_content(&$variables) {
  $variables['is_visible'] = FALSE;

  $attrs = $variables['block_attributes'];
  $count = $attrs['count'] ?? 3;
  $content_type = $attrs['contentType'] ?? 'article';

  $nids = \Drupal::entityQuery('node')
    ->condition('type', $content_type)
    ->condition('status', 1)
    ->sort('created', 'DESC')
    ->range(0, $count)
    ->accessCheck(TRUE)
    ->execute();

  if (!empty($nids)) {
    $nodes = \Drupal::entityTypeManager()
      ->getStorage('node')
      ->loadMultiple($nids);
    $view_builder = \Drupal::entityTypeManager()->getViewBuilder('node');

    // Render each node safely via view modes instead of using |raw in Twig.
    $teasers = [];
    foreach ($nodes as $node) {
      $teasers[$node->id()] = $view_builder->view($node, 'teaser');
    }

    $variables['nodes'] = $nodes;
    $variables['node_teasers'] = $teasers;
    $variables['is_visible'] = TRUE;

    // Invalidate when articles change.
    $variables['#cache']['tags'][] = 'node_list:' . $content_type;
    foreach ($nodes as $node) {
      $variables['#cache']['tags'][] = 'node:' . $node->id();
    }
  }
}
```
