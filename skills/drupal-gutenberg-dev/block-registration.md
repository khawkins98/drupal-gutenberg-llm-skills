# Block registration

## Registering blocks

Blocks are registered in JavaScript via `registerBlockType()`. The `.gutenberg.yml` file tells Drupal which libraries to load and which blocks are server-rendered.

For **client-side blocks** (blocks that save their own HTML), you only need:
1. **`.gutenberg.yml`** — Lists the editor library under `libraries-edit:`
2. **`registerBlockType()`** — Registers the block in JavaScript

For **dynamic blocks** (server-rendered via PHP/Twig), you also need:
3. **`dynamic-blocks:`** entry in `.gutenberg.yml`

## Server-side: `.gutenberg.yml`

### Client-side block (JS saves HTML)

```yaml
libraries-edit:
  - my_module/block-edit
```

No `blocks:` or `dynamic-blocks:` entry needed; the block is fully defined in JS.

### Dynamic block (server-side rendered)

```yaml
libraries-edit:
  - my_module/block-edit

dynamic-blocks:
  my-module/my-block: {}
```

### Multiple blocks

```yaml
libraries-edit:
  - my_module/block-edit

libraries-view:
  - my_module/block-view

dynamic-blocks:
  my-module/hero: {}
  my-module/featured-content: {}
```

Client-side blocks (like a CTA card) don't need a `dynamic-blocks:` entry; they're registered in JavaScript and included via the `block-edit` library.

## Client-side: `registerBlockType()`

### Minimal registration

```js
const { registerBlockType } = wp.blocks;
const { useBlockProps } = wp.blockEditor;
const { createElement: el } = wp.element;
const __ = Drupal.t;

registerBlockType('my-module/my-block', {
  apiVersion: 2,
  title: __('My Block'),
  icon: 'smiley',
  category: 'common',

  edit() {
    const blockProps = useBlockProps();
    return el('div', blockProps, __('Block editor view'));
  },

  save() {
    const blockProps = useBlockProps.save();
    return el('div', blockProps, 'Block saved output');
  },
});
```

### Full registration with attributes

```js
const { registerBlockType } = wp.blocks;
const { createElement: el } = wp.element;
const { useBlockProps, RichText, InspectorControls, MediaUpload } = wp.blockEditor;
const { PanelBody, TextControl, ToggleControl, Button } = wp.components;
const __ = Drupal.t;

registerBlockType('my-module/feature-card', {
  apiVersion: 2,
  title: __('Feature Card'),
  description: __('A card with image, title, and description.'),
  icon: 'index-card',
  category: 'common',
  keywords: [__('card'), __('feature'), __('box')],
  supports: {
    align: ['wide', 'full'],
    html: false,
  },
  attributes: {
    title: {
      type: 'string',
      source: 'html',
      selector: 'h3',
    },
    description: {
      type: 'string',
      source: 'html',
      selector: 'p',
    },
    imageUrl: {
      type: 'string',
      default: '',
    },
    imageAlt: {
      type: 'string',
      default: '',
    },
    mediaEntityId: {
      type: 'number',
    },
    showBorder: {
      type: 'boolean',
      default: false,
    },
  },

  edit({ attributes, setAttributes }) {
    const { title, description, imageUrl, imageAlt, showBorder } = attributes;
    const blockProps = useBlockProps();

    return el('div', blockProps,
      el(InspectorControls, {},
        el(PanelBody, { title: __('Card Settings') },
          el(ToggleControl, {
            label: __('Show border'),
            checked: showBorder,
            onChange: (val) => setAttributes({ showBorder: val }),
          })
        )
      ),
      el('div', { className: `feature-card${showBorder ? ' has-border' : ''}` },
        el(MediaUpload, {
          onSelect: (media) => setAttributes({
            // Drupal Gutenberg returns Drupal-shaped media objects; see media-integration.md
            imageUrl: media.url,
            imageAlt: media.alt_text || media.alt || '',
            mediaEntityId: media.media_entity?.id,
          }),
          allowedTypes: ['image'],
          render: ({ open }) => el('img', {
            src: imageUrl || 'data:image/svg+xml,...',
            alt: imageAlt,
            onClick: open,
            style: { cursor: 'pointer' },
          }),
        }),
        el(RichText, {
          tagName: 'h3',
          value: title,
          onChange: (val) => setAttributes({ title: val }),
          placeholder: __('Card title...'),
        }),
        el(RichText, {
          tagName: 'p',
          value: description,
          onChange: (val) => setAttributes({ description: val }),
          placeholder: __('Card description...'),
        })
      )
    );
  },

  save({ attributes }) {
    const { title, description, imageUrl, imageAlt, showBorder } = attributes;
    const blockProps = useBlockProps.save({
      className: `feature-card${showBorder ? ' has-border' : ''}`,
    });

    return el('div', blockProps,
      imageUrl && el('img', { src: imageUrl, alt: imageAlt }),
      el(RichText.Content, { tagName: 'h3', value: title }),
      el(RichText.Content, { tagName: 'p', value: description })
    );
  },
});
```

## Block categories

### Default categories

| Slug | Label |
|------|-------|
| `common` | Common blocks |
| `formatting` | Formatting |
| `layout` | Layout elements |
| `widgets` | Widgets |
| `embed` | Embeds |

### Custom categories

Register custom categories in `.gutenberg.yml`:

```yaml
custom-categories:
  - slug: my-site-blocks
    title: "My Site"
```

Or dynamically in JavaScript via the data store:

```js
const { dispatch, select } = wp.data;

// Add a custom category at the beginning of the list
const currentCategories = select('core/blocks').getCategories();
dispatch('core/blocks').setCategories([
  { slug: 'my-site', title: __('My Site Blocks'), icon: 'admin-site' },
  ...currentCategories,
]);
```

Then use the slug in block registration:

```js
registerBlockType('my-module/promo', {
  title: __('Promo Block'),
  category: 'my-site',
  // ...
});
```

## Block styles

Block styles change only the visual appearance, not structure or attributes:

```js
wp.blocks.registerBlockStyle('core/paragraph', {
  name: 'fancy',
  label: 'Fancy',
});

wp.blocks.registerBlockStyle('core/quote', {
  name: 'modern',
  label: 'Modern',
});
```

The selected style adds a CSS class `is-style-{name}` to the block wrapper. You provide CSS for these classes in your theme or module.

### Unregistering default styles

```js
wp.blocks.unregisterBlockStyle('core/quote', 'large');
```

## Block variations

Variations are different configurations of the same block type. Unlike styles, variations can change attributes and inner blocks:

```js
wp.blocks.registerBlockVariation('core/columns', {
  name: 'two-equal',
  title: 'Two Equal Columns',
  description: 'Two columns of equal width',
  innerBlocks: [
    ['core/column', { width: '50%' }],
    ['core/column', { width: '50%' }],
  ],
  scope: ['inserter'],
});

wp.blocks.registerBlockVariation('core/columns', {
  name: 'sidebar-right',
  title: 'Sidebar Right',
  description: 'Main content with right sidebar',
  innerBlocks: [
    ['core/column', { width: '66.66%' }],
    ['core/column', { width: '33.33%' }],
  ],
  scope: ['inserter'],
});
```

### Variation properties

| Property | Description |
|----------|-------------|
| `name` | Unique identifier |
| `title` | Display name in inserter |
| `description` | Tooltip text |
| `attributes` | Default attribute values |
| `innerBlocks` | Default inner block template |
| `icon` | Custom icon |
| `scope` | Where it appears: `['inserter']`, `['block']`, `['transform']` |
| `isDefault` | If `true`, used when the block is inserted |
| `isActive` | Function or array to detect if this variation is active |

## Block supports

`supports` controls editor features for your block:

```js
registerBlockType('my-module/my-block', {
  // ...
  supports: {
    align: ['wide', 'full'],        // Specific alignments (use `true` for all)
    anchor: true,                   // Enable HTML anchor field
    className: true,                // Auto-add className (default: true)
    color: {
      background: true,
      text: true,
      gradients: true,
    },
    html: false,                    // Disable HTML editing
    inserter: true,                 // Show in block inserter (default: true)
    multiple: true,                 // Allow multiple instances (default: true)
    reusable: true,                 // Allow conversion to reusable block
    typography: {
      fontSize: true,
      lineHeight: true,
    },
    spacing: {
      padding: true,
      margin: true,
    },
  },
});
```

## Block deprecations

When you change a block's saved markup or attributes, existing content breaks. Use the `deprecated` array to define migration paths:

```js
// v1 of the block saved content with a different class
const v1 = {
  attributes: {
    content: { type: 'string', source: 'html', selector: 'p' },
  },
  save({ attributes }) {
    return el('div', { className: 'old-card' },
      el(RichText.Content, { tagName: 'p', value: attributes.content })
    );
  },
};

registerBlockType('my-module/card', {
  apiVersion: 2,
  title: __('Card'),
  icon: 'index-card',
  category: 'common',
  attributes: {
    content: { type: 'string', source: 'html', selector: 'p' },
    variant: { type: 'string', default: 'default' },
  },

  // Current version
  edit({ attributes, setAttributes }) {
    const blockProps = useBlockProps();
    return el('div', blockProps,
      el(RichText, {
        tagName: 'p',
        value: attributes.content,
        onChange: (val) => setAttributes({ content: val }),
      })
    );
  },

  save({ attributes }) {
    const blockProps = useBlockProps.save({
      className: 'card card--' + attributes.variant,
    });
    return el('div', blockProps,
      el(RichText.Content, { tagName: 'p', value: attributes.content })
    );
  },

  // Previous versions — Gutenberg tries each in order to parse old content
  deprecated: [v1],
});
```

Gutenberg tries each deprecated version's `save` output against existing content. When it finds a match, it migrates to the current format. Add a `migrate(attributes)` function to transform old attributes to new ones.

## Data-attribute hydration pattern

For blocks with interactive front-end behavior (search widgets, maps, etc.), save configuration as `data-*` attributes in `save`. Front-end JS reads these and hydrates:

```js
// In the block's save function
save({ attributes }) {
  const blockProps = useBlockProps.save({
    className: 'my-interactive-widget',
    'data-config': JSON.stringify({
      endpoint: attributes.endpoint,
      filters: attributes.filters,
      limit: attributes.limit,
    }),
  });
  return el('div', blockProps);
},
```

Front-end JS (loaded via `libraries-view:`) reads and hydrates:

```js
// js/theme/my-widget-frontend.js
document.querySelectorAll('.my-interactive-widget').forEach((container) => {
  const config = JSON.parse(container.dataset.config);
  // Initialize your interactive component with the config
});
```

## Existing Drupal blocks

All existing Drupal blocks (Views blocks, custom blocks, etc.) appear automatically in the Gutenberg inserter under "Drupal Blocks". No registration needed; the Gutenberg module discovers them.
