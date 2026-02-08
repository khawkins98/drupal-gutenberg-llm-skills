# Block syntax reference

## Block grammar

Drupal Gutenberg stores content as HTML with block-delimiting comments. The parser (`Drupal\gutenberg\Parser\BlockParser`) reads these comments to reconstruct the block tree.

### Opening comment

```
<!-- wp:blockname {"attribute":"value"} -->
```

- `wp:` prefix is mandatory (WordPress compatibility)
- Block name follows immediately after `wp:`
- Namespaced blocks: `wp:namespace/block-name`
- Optional JSON attributes object after block name, separated by a space
- JSON must be compact (no pretty-printing) and valid

### Closing comment

```
<!-- /wp:blockname -->
```

- Forward slash before the block name
- Must match the opening block name exactly

### Self-closing comment (dynamic blocks)

```
<!-- wp:blockname {"attribute":"value"} /-->
```

- Forward slash before `-->`
- No closing comment needed
- Used for server-rendered blocks that have no saved HTML content

## Attribute types

Attributes are defined in the block's registration and serialized as JSON in the opening comment.

| Type | Example | Notes |
|------|---------|-------|
| `string` | `{"className":"my-class"}` | CSS classes, text values |
| `number` | `{"level":3}` | Heading levels, counts |
| `boolean` | `{"displayPostDate":true}` | Toggles |
| `array` | `{"ids":[1,2,3]}` | Lists of values |
| `object` | `{"style":{"color":{"text":"#ff0000"}}}` | Nested configuration |

### Attribute source

Attributes can be stored in two places:

1. **In the comment** (default) — serialized as JSON in the opening comment
2. **In the HTML** — extracted from the HTML content via `source` selectors (e.g., `innerHTML`, `attribute`, `text`)

When `source` is specified in the block definition, the value lives in the HTML markup and is not duplicated in the comment JSON. The comment JSON only contains attributes without a `source`.

## Common attribute patterns

### Alignment
```json
{"align":"wide"}
{"align":"full"}
{"align":"left"}
{"align":"center"}
{"align":"right"}
```

### Colors
```json
{"backgroundColor":"pale-pink","textColor":"vivid-red"}
```

```json
{"style":{"color":{"background":"#f0f0f0","text":"#333333"}}}
```

### Typography
```json
{"style":{"typography":{"fontSize":"24px","fontWeight":"700"}}}
```

### Spacing
```json
{"style":{"spacing":{"padding":{"top":"2em","bottom":"2em"}}}}
```

### Custom CSS class
```json
{"className":"my-custom-class another-class"}
```

### Anchor (ID)
```json
{"anchor":"my-section"}
```

## Block validation

The editor validates saved content against the block's `save` function output. If the HTML doesn't match, the block shows as "invalid" in the editor. Rules:

- HTML structure must match the block's save template exactly
- Self-closing tags (like `<img />`) must be consistent
- Attribute order in HTML tags should match save output
- Whitespace differences may or may not cause validation errors depending on the block

## Content storage

Gutenberg markup is stored as a single HTML string in:
- **Table:** `node__body`
- **Column:** `body_value`
- **Format:** `gutenberg` (set in the text format field)

The entire page content (all blocks) is one continuous HTML string with block comments as delimiters.

## Common mistakes

### Wrong prefix
```html
<!-- drupal:paragraph -->   ❌ No "drupal:" prefix exists
<!-- gb:paragraph -->        ❌ No "gb:" prefix exists
<!-- wp:paragraph -->        ✅ Always use "wp:"
```

### Missing or mismatched closing
```html
<!-- wp:paragraph -->
<p>Text</p>
<!-- /wp:heading -->         ❌ Closing name must match opening

<!-- wp:paragraph -->
<p>Text</p>
                             ❌ Missing closing comment entirely
```

### Pretty-printed or invalid JSON attributes
```html
<!-- wp:heading {
  "level": 3
} -->                        ❌ JSON must be compact (single line)

<!-- wp:heading {"level":3,} -->  ❌ Trailing comma is invalid JSON

<!-- wp:heading {"level":3} -->   ✅ Compact, valid JSON
```

### Using `<li>` without `wp:list-item` wrappers
```html
<!-- wp:list -->
<ul>
    <li>Item</li>            ❌ Raw <li> without list-item blocks (old format)
</ul>
<!-- /wp:list -->

<!-- wp:list -->
<ul><!-- wp:list-item -->
<li>Item</li>
<!-- /wp:list-item --></ul>  ✅ Each <li> wrapped in wp:list-item
<!-- /wp:list -->
```

### Self-closing syntax on non-dynamic blocks
```html
<!-- wp:paragraph /-->       ❌ Paragraph has save content; cannot self-close
<!-- wp:latest-posts /-->    ✅ Dynamic/server-rendered blocks can self-close
```

### Attributes on closing comments
```html
<!-- /wp:heading {"level":3} -->  ❌ Attributes go on the opening comment only
<!-- /wp:heading -->              ✅
```
