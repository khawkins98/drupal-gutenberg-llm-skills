# JavaScript build process

## ES6 to ES5 workflow

Drupal Gutenberg uses Babel to transpile ES6+ JavaScript to ES5. Source files use the `.es6.js` extension by convention.

### File naming convention

```
src/my-block.es6.js    →    js/my-block.js      (manual Babel: babel src --out-dir js)
js/my-block.es6.js     →    js/my-block.js       (drupal-js-build: compiles in-place)
```

The `.es6.js` extension signals that the file needs transpilation. The compiled `.js` output is what `.libraries.yml` references. The output location depends on your build tool (see sections below).

### Build commands

```bash
npm run build    # One-time compilation
npm start        # Watch mode (recompile on changes)
```

## Translations

Drupal Gutenberg modules use `Drupal.t` for translatable strings, **not** `wp.i18n.__`:

```js
// Use Drupal's translation system
const __ = Drupal.t;

registerBlockType('my-module/my-block', {
  title: __('My Block'),
  description: __('A custom block.'),
  // ...
});
```

This ensures strings are extracted by Drupal's translation pipeline and available at `/admin/config/regional/translate`.

## Basic block source file

```js
// src/my-block.es6.js

const { registerBlockType } = wp.blocks;
const { createElement: el, Fragment } = wp.element;
const { useBlockProps, RichText, InspectorControls } = wp.blockEditor;
const { PanelBody, TextControl } = wp.components;
const __ = Drupal.t;

registerBlockType('my-module/my-block', {
  apiVersion: 2,
  title: __('My Block'),
  icon: 'smiley',
  category: 'common',
  attributes: {
    content: {
      type: 'string',
      source: 'html',
      selector: 'p',
    },
    subtitle: {
      type: 'string',
      default: '',
    },
  },

  edit: function(props) {
    const { attributes, setAttributes } = props;
    const blockProps = useBlockProps();

    return el(Fragment, {},
      el(InspectorControls, {},
        el(PanelBody, { title: __('Settings') },
          el(TextControl, {
            label: __('Subtitle'),
            value: attributes.subtitle,
            onChange: (value) => setAttributes({ subtitle: value }),
          })
        )
      ),
      el('div', blockProps,
        el(RichText, {
          tagName: 'p',
          value: attributes.content,
          onChange: (content) => setAttributes({ content }),
          placeholder: __('Enter text...'),
        })
      )
    );
  },

  save: function(props) {
    const blockProps = useBlockProps.save();

    return el('div', blockProps,
      el(RichText.Content, {
        tagName: 'p',
        value: props.attributes.content,
      })
    );
  },
});
```

> **`useBlockProps`**: Always wrap your root element with `useBlockProps()` (edit) and `useBlockProps.save()` (save). This applies block wrapper attributes (class names, IDs, data attributes) that Gutenberg expects. Without it, block selection and toolbars won't work. Requires `apiVersion: 2` in the block registration.

## JSX alternative

If your Babel config includes `@babel/preset-react`, you can use JSX:

```jsx
// src/my-block.es6.js

const { registerBlockType } = wp.blocks;
const { Fragment } = wp.element;
const { useBlockProps, RichText, InspectorControls } = wp.blockEditor;
const { PanelBody, TextControl } = wp.components;
const __ = Drupal.t;

registerBlockType('my-module/my-block', {
  apiVersion: 2,
  title: __('My Block'),
  icon: 'smiley',
  category: 'common',
  attributes: {
    content: {
      type: 'string',
      source: 'html',
      selector: 'p',
    },
    subtitle: {
      type: 'string',
      default: '',
    },
  },

  edit({ attributes, setAttributes }) {
    const blockProps = useBlockProps();

    return (
      <Fragment>
        <InspectorControls>
          <PanelBody title={__('Settings')}>
            <TextControl
              label={__('Subtitle')}
              value={attributes.subtitle}
              onChange={(subtitle) => setAttributes({ subtitle })}
            />
          </PanelBody>
        </InspectorControls>
        <div {...blockProps}>
          <RichText
            tagName="p"
            value={attributes.content}
            onChange={(content) => setAttributes({ content })}
            placeholder={__('Enter text...')}
          />
        </div>
      </Fragment>
    );
  },

  save({ attributes }) {
    const blockProps = useBlockProps.save();

    return (
      <div {...blockProps}>
        <RichText.Content tagName="p" value={attributes.content} />
      </div>
    );
  },
});
```

## Accessing Gutenberg APIs

Drupal Gutenberg exposes the same `wp` global as WordPress:

| Global | Purpose | Common Imports |
|--------|---------|----------------|
| `wp.blocks` | Block registration | `registerBlockType`, `createBlock` |
| `wp.element` | React wrapper | `createElement`, `Fragment`, `useState` |
| `wp.blockEditor` | Editor components | `useBlockProps`, `RichText`, `InnerBlocks`, `InspectorControls`, `MediaUpload` |
| `wp.components` | UI components | `PanelBody`, `TextControl`, `ToggleControl`, `SelectControl`, `Button` |
| `wp.data` | Data store | `select`, `dispatch`, `useSelect`, `useDispatch` |
| `wp.compose` | Higher-order components | `withSelect`, `withDispatch`, `compose` |
| `wp.serverSideRender` | SSR preview component | `ServerSideRender` (default export) |
| `Drupal.t` | Translations | Alias as `const __ = Drupal.t;` (see above) |

### Do not use ES6 imports for wp packages

Drupal provides Gutenberg APIs as globals on the `wp` object. Do not use ES6 import statements for WordPress packages:

```js
// WRONG — will not work without a bundler
import { registerBlockType } from '@wordpress/blocks';

// CORRECT — use the wp global
const { registerBlockType } = wp.blocks;
```

If you want ES6 imports, you need a bundler (webpack). See the webpack section below.

## Babel configuration

### Default `.babelrc`

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

Set `"modules": false` to prevent Babel from transforming `import`/`export` to `require()`. Without a bundler, `require()` doesn't work in browsers. Since we use `wp.*` globals, this setting just ensures nothing breaks.

### Required npm packages

```json
{
  "devDependencies": {
    "@babel/cli": "^7.0.0",
    "@babel/core": "^7.0.0",
    "@babel/preset-env": "^7.0.0",
    "@babel/preset-react": "^7.0.0"
  }
}
```

## drupal-js-build (recommended)

`drupal-js-build` is the standard build tool for Drupal Gutenberg modules. It compiles `.es6.js` files in-place (creating `.js` siblings) and handles CSS:

```bash
npm install --save-dev drupal-js-build cross-env drupal-gutenberg-translations
```

```json
{
  "scripts": {
    "start": "cross-env NODE_ENV=production drupal-js-build watch --css",
    "build": "cross-env NODE_ENV=production drupal-gutenberg-translations && drupal-js-build --css"
  }
}
```

`drupal-js-build` finds `.es6.js` files and compiles them to `.js` siblings in the same directory. Reference the compiled `.js` files in `.libraries.yml`.

## Alternative: webpack

For projects that need module bundling, code splitting, or ES6 import support:

### `webpack.config.js`

```js
const path = require('path');

module.exports = {
  entry: './src/my-block.es6.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-block.js',
  },
  module: {
    rules: [
      {
        test: /\.es6\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react'],
          },
        },
      },
    ],
  },
  externals: {
    // Map ES6 imports to wp globals
    '@wordpress/blocks': 'wp.blocks',
    '@wordpress/element': 'wp.element',
    '@wordpress/block-editor': 'wp.blockEditor',
    '@wordpress/components': 'wp.components',
    '@wordpress/data': 'wp.data',
    '@wordpress/compose': 'wp.compose',
    // NOTE: Do not add @wordpress/i18n — use `const __ = Drupal.t;` instead
  },
};
```

With webpack, you can use standard ES6 imports:

```js
import { registerBlockType } from '@wordpress/blocks';
import { RichText } from '@wordpress/block-editor';
```

The `externals` config maps these imports to the `wp.*` globals at runtime.

### Webpack npm packages

```json
{
  "devDependencies": {
    "webpack": "^5.0.0",
    "webpack-cli": "^5.0.0",
    "babel-loader": "^9.0.0",
    "@babel/core": "^7.0.0",
    "@babel/preset-env": "^7.0.0",
    "@babel/preset-react": "^7.0.0"
  },
  "scripts": {
    "build": "webpack --mode production",
    "start": "webpack --mode development --watch"
  }
}
```

## Build toolchain comparison

| Tool | Pros | Cons |
|------|------|------|
| **drupal-js-build** (recommended) | Standard for Drupal Gutenberg, handles translations + CSS | Limited configuration |
| **Babel (manual)** | Simple, full control, lightweight | No bundling, must use `wp.*` globals |
| **webpack** | Full bundling, ES6 imports, code splitting, CSS modules | More setup |
| **esbuild** | Fast, good for bundling React hydration code | Requires externals config for wp/React globals |

For most custom blocks, `drupal-js-build` is the right choice. Use webpack or esbuild when bundling third-party dependencies (charting libraries, design systems) or when you want ES6 import syntax for `@wordpress/*` packages.
