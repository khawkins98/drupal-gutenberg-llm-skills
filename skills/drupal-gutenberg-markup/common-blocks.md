# Common block examples

Copy-paste examples for all common Gutenberg blocks in Drupal.

## Text blocks

### Paragraph

```html
<!-- wp:paragraph -->
<p>A simple paragraph of text.</p>
<!-- /wp:paragraph -->
```

With custom class:
```html
<!-- wp:paragraph {"className":"lead-text"} -->
<p class="lead-text">A styled paragraph.</p>
<!-- /wp:paragraph -->
```

With drop cap:
```html
<!-- wp:paragraph {"dropCap":true} -->
<p class="has-drop-cap">This paragraph starts with a large drop capital letter.</p>
<!-- /wp:paragraph -->
```

### Heading

```html
<!-- wp:heading -->
<h2>Default H2 Heading</h2>
<!-- /wp:heading -->
```

```html
<!-- wp:heading {"level":1} -->
<h1>H1 Heading</h1>
<!-- /wp:heading -->
```

```html
<!-- wp:heading {"level":3,"anchor":"my-section"} -->
<h3 id="my-section">H3 With Anchor</h3>
<!-- /wp:heading -->
```

### List

Unordered:
```html
<!-- wp:list -->
<ul><!-- wp:list-item -->
<li>First item</li>
<!-- /wp:list-item -->

<!-- wp:list-item -->
<li>Second item</li>
<!-- /wp:list-item -->

<!-- wp:list-item -->
<li>Third item</li>
<!-- /wp:list-item --></ul>
<!-- /wp:list -->
```

Ordered:
```html
<!-- wp:list {"ordered":true} -->
<ol><!-- wp:list-item -->
<li>Step one</li>
<!-- /wp:list-item -->

<!-- wp:list-item -->
<li>Step two</li>
<!-- /wp:list-item -->

<!-- wp:list-item -->
<li>Step three</li>
<!-- /wp:list-item --></ol>
<!-- /wp:list -->
```

> **Note:** Each `<li>` is wrapped in a `<!-- wp:list-item -->` inner block. This format was introduced in Gutenberg 14.1+ and is required by Drupal Gutenberg 3.x (which bundles Gutenberg 16.7).

### Quote

```html
<!-- wp:quote -->
<blockquote class="wp-block-quote"><!-- wp:paragraph -->
<p>The only way to do great work is to love what you do.</p>
<!-- /wp:paragraph -->
<cite>Steve Jobs</cite></blockquote>
<!-- /wp:quote -->
```

> **Note:** Like lists, the Quote block uses inner blocks in Gutenberg 16.7. Each paragraph inside the blockquote is wrapped in `<!-- wp:paragraph -->` blocks.

### Pullquote

```html
<!-- wp:pullquote -->
<figure class="wp-block-pullquote">
    <blockquote><!-- wp:paragraph -->
        <p>A highlighted quote for emphasis.</p>
        <!-- /wp:paragraph -->
        <cite>Attribution</cite>
    </blockquote>
</figure>
<!-- /wp:pullquote -->
```

> **Note:** Like Quote, the Pullquote block uses inner `<!-- wp:paragraph -->` blocks in Gutenberg 16.7.

### Preformatted text

```html
<!-- wp:preformatted -->
<pre class="wp-block-preformatted">Preformatted text
    preserves whitespace
        and line breaks.</pre>
<!-- /wp:preformatted -->
```

### Code

```html
<!-- wp:code -->
<pre class="wp-block-code"><code>function hello() {
  return "world";
}</code></pre>
<!-- /wp:code -->
```

### Verse

```html
<!-- wp:verse -->
<pre class="wp-block-verse">A poem or verse
with preserved
line breaks.</pre>
<!-- /wp:verse -->
```

## Media blocks

### Image

Basic:
```html
<!-- wp:image -->
<figure class="wp-block-image">
    <img src="/sites/default/files/photo.jpg" alt="Photo description" />
</figure>
<!-- /wp:image -->
```

With size, link, and caption:
```html
<!-- wp:image {"sizeSlug":"large","linkDestination":"media"} -->
<figure class="wp-block-image size-large">
    <a href="/sites/default/files/photo-full.jpg">
        <img src="/sites/default/files/styles/large/public/photo.jpg" alt="Photo" />
    </a>
    <figcaption class="wp-element-caption">Photo caption text</figcaption>
</figure>
<!-- /wp:image -->
```

> **Note:** Drupal Gutenberg 3.x adds `class="wp-element-caption"` to `<figcaption>` elements. Always include this class to avoid block validation warnings.

With Drupal media entity attributes (for images uploaded via the Drupal media library):
```html
<!-- wp:image {"id":12345,"sizeSlug":"full","linkDestination":"none","mediaAttrs":{"data-entity-type":"file","data-entity-uuid":"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","data-image-style":"original"}} -->
<figure class="wp-block-image size-full"><img src="/sites/default/files/2026-01/photo.jpg" alt="Description" class="wp-image-12345" data-entity-type="file" data-entity-uuid="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" data-image-style="original"/></figure>
<!-- /wp:image -->
```

Aligned:
```html
<!-- wp:image {"align":"center","sizeSlug":"medium"} -->
<figure class="wp-block-image aligncenter size-medium">
    <img src="/sites/default/files/styles/medium/public/photo.jpg" alt="" />
</figure>
<!-- /wp:image -->
```

### Gallery

```html
<!-- wp:gallery {"columns":3,"linkTo":"none"} -->
<figure class="wp-block-gallery has-nested-images columns-3 is-cropped">
    <!-- wp:image {"sizeSlug":"large"} -->
    <figure class="wp-block-image size-large">
        <img src="/sites/default/files/image1.jpg" alt="" />
    </figure>
    <!-- /wp:image -->
    <!-- wp:image {"sizeSlug":"large"} -->
    <figure class="wp-block-image size-large">
        <img src="/sites/default/files/image2.jpg" alt="" />
    </figure>
    <!-- /wp:image -->
    <!-- wp:image {"sizeSlug":"large"} -->
    <figure class="wp-block-image size-large">
        <img src="/sites/default/files/image3.jpg" alt="" />
    </figure>
    <!-- /wp:image -->
</figure>
<!-- /wp:gallery -->
```

### Video

```html
<!-- wp:video -->
<figure class="wp-block-video">
    <video controls src="/sites/default/files/video.mp4"></video>
</figure>
<!-- /wp:video -->
```

### Audio

```html
<!-- wp:audio -->
<figure class="wp-block-audio">
    <audio controls src="/sites/default/files/audio.mp3"></audio>
</figure>
<!-- /wp:audio -->
```

### File

```html
<!-- wp:file {"href":"/sites/default/files/document.pdf"} -->
<div class="wp-block-file">
    <a href="/sites/default/files/document.pdf">Download Document</a>
    <a href="/sites/default/files/document.pdf" class="wp-block-file__button" download>Download</a>
</div>
<!-- /wp:file -->
```

### Cover

```html
<!-- wp:cover {"url":"/sites/default/files/hero.jpg","dimRatio":50} -->
<div class="wp-block-cover">
    <span aria-hidden="true" class="wp-block-cover__background has-background-dim"></span>
    <img class="wp-block-cover__image-background" alt="" src="/sites/default/files/hero.jpg" />
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"level":1,"textColor":"white"} -->
        <h1 class="has-white-color has-text-color">Hero Title</h1>
        <!-- /wp:heading -->
        <!-- wp:paragraph {"textColor":"white"} -->
        <p class="has-white-color has-text-color">Subtitle text over image.</p>
        <!-- /wp:paragraph -->
    </div>
</div>
<!-- /wp:cover -->
```

## Layout blocks

### Columns

Two columns:
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

Three columns with custom widths:
```html
<!-- wp:columns -->
<div class="wp-block-columns">
    <!-- wp:column {"width":"25%"} -->
    <div class="wp-block-column" style="flex-basis:25%">
        <!-- wp:paragraph -->
        <p>Sidebar</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"50%"} -->
    <div class="wp-block-column" style="flex-basis:50%">
        <!-- wp:paragraph -->
        <p>Main content</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"25%"} -->
    <div class="wp-block-column" style="flex-basis:25%">
        <!-- wp:paragraph -->
        <p>Sidebar</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Group

> **Drupal Gutenberg:** `wp:group` causes rendering issues and should not be used. Use `wp:columns` with a single `wp:column` child as a wrapper instead. The examples below are for reference only — see patterns-and-nesting.md for the Drupal-compatible alternatives.

```html
<!-- wp:group {"className":"my-section"} -->
<div class="wp-block-group my-section">
    <!-- wp:heading {"level":2} -->
    <h2>Group Title</h2>
    <!-- /wp:heading -->
    <!-- wp:paragraph -->
    <p>Grouped content.</p>
    <!-- /wp:paragraph -->
</div>
<!-- /wp:group -->
```

With background color:
```html
<!-- wp:group {"backgroundColor":"pale-cyan-blue"} -->
<div class="wp-block-group has-pale-cyan-blue-background-color has-background">
    <!-- wp:paragraph -->
    <p>Content with background.</p>
    <!-- /wp:paragraph -->
</div>
<!-- /wp:group -->
```

### Separator

Default (CSS opacity — the Drupal Gutenberg serialized form):
```html
<!-- wp:separator {"opacity":"css"} -->
<hr class="wp-block-separator has-css-opacity"/>
<!-- /wp:separator -->
```

Wide separator:
```html
<!-- wp:separator {"className":"is-style-wide","opacity":"css"} -->
<hr class="wp-block-separator has-css-opacity is-style-wide"/>
<!-- /wp:separator -->
```

> **Note:** Drupal Gutenberg 3.x serializes the separator with `{"opacity":"css"}` and the `has-css-opacity` class. Omitting these will still render but may cause a block validation warning in the editor.

### Spacer

```html
<!-- wp:spacer {"height":"50px"} -->
<div style="height:50px" aria-hidden="true" class="wp-block-spacer"></div>
<!-- /wp:spacer -->
```

## Interactive blocks

### Buttons

Single button:
```html
<!-- wp:buttons -->
<div class="wp-block-buttons"><!-- wp:button -->
<div class="wp-block-button"><a class="wp-block-button__link wp-element-button" href="/contact">Contact Us</a></div>
<!-- /wp:button --></div>
<!-- /wp:buttons -->
```

Multiple buttons with styles:
```html
<!-- wp:buttons -->
<div class="wp-block-buttons"><!-- wp:button {"backgroundColor":"vivid-cyan-blue"} -->
<div class="wp-block-button"><a class="wp-block-button__link wp-element-button has-vivid-cyan-blue-background-color has-background" href="/signup">Sign Up</a></div>
<!-- /wp:button -->

<!-- wp:button {"className":"is-style-outline"} -->
<div class="wp-block-button is-style-outline"><a class="wp-block-button__link wp-element-button" href="/learn-more">Learn More</a></div>
<!-- /wp:button --></div>
<!-- /wp:buttons -->
```

> **Note:** Drupal Gutenberg 3.x adds `wp-element-button` to the button `<a>` tag. Always include it alongside `wp-block-button__link` to avoid block validation warnings.

### Table

```html
<!-- wp:table -->
<figure class="wp-block-table">
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>Role</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Alice</td>
                <td>Developer</td>
                <td>Active</td>
            </tr>
            <tr>
                <td>Bob</td>
                <td>Designer</td>
                <td>Active</td>
            </tr>
        </tbody>
    </table>
</figure>
<!-- /wp:table -->
```

With stripes:
```html
<!-- wp:table {"className":"is-style-stripes"} -->
<figure class="wp-block-table is-style-stripes">
    <table>
        <tbody>
            <tr><td>Row 1</td><td>Data</td></tr>
            <tr><td>Row 2</td><td>Data</td></tr>
        </tbody>
    </table>
</figure>
<!-- /wp:table -->
```

## Embed blocks

### Generic embed

```html
<!-- wp:embed {"url":"https://www.youtube.com/watch?v=dQw4w9WgXcQ","type":"video","providerNameSlug":"youtube"} -->
<figure class="wp-block-embed is-type-video is-provider-youtube">
    <div class="wp-block-embed__wrapper">
        https://www.youtube.com/watch?v=dQw4w9WgXcQ
    </div>
</figure>
<!-- /wp:embed -->
```

## Raw HTML block

For freeform HTML that doesn't fit any block type:

```html
<!-- wp:html -->
<div class="custom-embed">
    <iframe src="https://example.com/widget" width="600" height="400"></iframe>
</div>
<!-- /wp:html -->
```

The HTML block stores its content verbatim with no validation or transformation.

## Dynamic blocks (self-closing)

Dynamic blocks are server-rendered and use self-closing syntax:

```html
<!-- wp:latest-posts {"postsToShow":5,"displayPostDate":true} /-->
```

```html
<!-- wp:categories {"displayAsDropdown":true} /-->
```

### Drupal Views shortcode

```html
<!-- wp:shortcode -->
[drupal_view:my_view_name:my_display_id]
<!-- /wp:shortcode -->
```

> **Note:** The `[drupal_view:...]` shortcode requires the Gutenberg module's Views integration. The view and display must exist and be enabled.

## Custom blocks

Custom blocks from Drupal modules use namespaced names:

```html
<!-- wp:my-module/my-block {"field1":"value1","field2":"value2"} -->
<div class="wp-block-my-module-my-block">
    <p>Custom block content</p>
</div>
<!-- /wp:my-module/my-block -->
```

Dynamic custom block (server-rendered):
```html
<!-- wp:my-module/my-dynamic-block {"entityId":42,"viewMode":"teaser"} /-->
```
