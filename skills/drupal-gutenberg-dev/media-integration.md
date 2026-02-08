# Media integration

## File entities vs media entities

Drupal Gutenberg creates **file entities** by default when users upload through the editor. Drupal's Media module uses **media entities** instead (a wrapper around file entities with additional metadata).

| Aspect | File Entity | Media Entity |
|--------|-------------|--------------|
| Created by | Gutenberg upload (default) | Media module |
| Admin path | `/admin/content/files` | `/admin/content/media` |
| Metadata | Filename, size, MIME type | Additional fields, thumbnails, alt text |
| View modes | Limited | Full view mode support |
| Required modules | Core (file) | Media, Media Library |

## Upload behavior

When a user drags an image or uses the upload button:

1. A **file entity** is created in Drupal's file system
2. The file is saved to the configured file storage (public/private)
3. The file is flagged as **permanent** (unlike standard Drupal uploads which start as temporary)
4. The image URL is inserted into the block markup

### File storage path

Uploaded files are stored at:
```
public://gutenberg/
```

Or as configured by the site's file system settings.

### Permanent file flag

All Gutenberg-uploaded media is marked permanent immediately. Standard Drupal behavior differs:
- Standard: files start temporary, become permanent on entity save
- Gutenberg: files are permanent from upload

Drupal's cron won't clean up Gutenberg-uploaded files, even if the referencing content is deleted. Admins should periodically review `/admin/content/files`.

## Using images in blocks

### Drupal vs WordPress media object shape

The `onSelect` callback in Drupal Gutenberg receives a Drupal-shaped media object, not the WordPress shape:

```js
// WordPress media object (NOT what Drupal returns):
// media.url, media.alt, media.id

// Drupal Gutenberg media object:
// media.url                                      — direct URL to the file
// media.alt_text                                 — alt text (note: alt_text, not alt)
// media.media_entity.id                          — Drupal media entity ID
// media.media_details.file                       — filename
// media.media_details.sizes.{style}.source_url   — URL for a specific image style
// media.media_details.sizes.{style}.width        — width for that style
// media.media_details.sizes.{style}.height       — height for that style
```

The available image styles under `media.media_details.sizes` depend on what image styles are configured in Drupal (e.g., `thumbnail`, `medium`, `large`, or custom styles).

### Image block

```js
const { MediaUpload, MediaUploadCheck } = wp.blockEditor;
const { Button } = wp.components;
const { createElement: el } = wp.element;
const __ = Drupal.t;

// In the edit function:
el(MediaUploadCheck, {},
  el(MediaUpload, {
    onSelect: (media) => {
      setAttributes({
        imageUrl: media.url,
        imageAlt: media.alt_text || '',
        mediaEntityId: media.media_entity?.id,
      });
    },
    allowedTypes: ['image'],
    value: attributes.mediaEntityId,
    render: ({ open }) => {
      return attributes.imageUrl
        ? el('div', {},
            el('img', { src: attributes.imageUrl, alt: attributes.imageAlt }),
            el(Button, { onClick: open, isSecondary: true }, __('Replace Image'))
          )
        : el(Button, { onClick: open, isPrimary: true }, __('Upload Image'));
    },
  })
);
```

### Using specific image styles

To use a specific Drupal image style (e.g., a responsive hero):

```js
onSelect: (media) => {
  // Prefer a specific image style, fall back to the full URL
  const heroUrl = media.media_details?.sizes?.hero_wide?.source_url || media.url;
  setAttributes({
    imageUrl: heroUrl,
    imageAlt: media.alt_text || '',
    mediaEntityId: media.media_entity?.id,
  });
},
```

### Attributes for media

```js
attributes: {
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
},
```

## Media Library integration

When Media and Media Library modules are installed:

1. The Gutenberg upload dialog can be configured to use Drupal's Media Library
2. Users get access to existing media entities, not just new uploads
3. Media browser provides filtering, search, and grid/table views

### Enabling Media Library module

1. Install and enable the Media Library module:
   ```bash
   drush en media_library
   ```

2. Configure Gutenberg for the content type at `/admin/config/content/gutenberg`

3. Media Library will appear as an option in the image/media block upload interface

### Known limitations

- The Media Library window may display blank with certain theme configurations
- Not all media types work in the Gutenberg media browser
- Integration between Gutenberg's JS media handling and Drupal's Media Library is still evolving

## Media block with view modes

Drupal Gutenberg has a "Media block" that embeds media entities with selectable view modes.

### Setup

1. Enable the Media block in Gutenberg configuration for the content type
2. Create custom view modes for media entities at `/admin/structure/media/manage/{type}/display`
3. When inserting a Media block, users can select which view mode to use

### Template for media blocks

```twig
{# In your theme or module template #}
{% if media_entity %}
  <div class="gutenberg-media-block gutenberg-media-block--{{ view_mode }}">
    {{ media_entity }}
  </div>
{% endif %}
```

## Image handling in server-side blocks

For dynamic blocks rendering images server-side, load the file entity in your preprocessing hook:

```php
/**
 * Implements hook_preprocess_gutenberg_block().
 */
function my_module_preprocess_gutenberg_block(&$variables) {
  if (($variables['block_name'] ?? '') !== 'my_module/my-block') {
    return;
  }

  $attrs = $variables['block_attributes'];

  if (!empty($attrs['mediaEntityId'])) {
    $file = \Drupal::entityTypeManager()
      ->getStorage('file')
      ->load($attrs['mediaEntityId']);

    if ($file) {
      $variables['image_url'] = \Drupal::service('file_url_generator')
        ->generateAbsoluteString($file->getFileUri());
      $variables['image_alt'] = $attrs['imageAlt'] ?? '';
    }
  }
}
```

### Twig template

```twig
<div{{ attributes.addClass('my-block') }}>
  {% if image_url %}
    <figure class="my-block__image">
      <img src="{{ image_url }}" alt="{{ image_alt }}" />
    </figure>
  {% endif %}

  <div class="my-block__content">
    {{ block_content }}
  </div>
</div>
```

## Gotchas

- Store file entity IDs in block attributes, not hardcoded URLs. This allows file tracking and proper URL generation.
- For dynamic blocks, generate URLs server-side via Drupal's file URL generator rather than storing absolute URLs.
- Always include alt text attributes.
- Use Drupal's image styles for responsive images in server-side blocks:
  ```php
  $style = \Drupal::entityTypeManager()
    ->getStorage('image_style')
    ->load('large');
  $variables['styled_image_url'] = $style->buildUrl($file->getFileUri());
  ```
- Gutenberg uploads are always permanent. Periodically review `/admin/content/files` for orphaned files.
