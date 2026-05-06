# Patterns and nesting

> **Drupal Gutenberg note:** The `wp:group` block causes rendering issues in Drupal Gutenberg and should not be used. Some examples in this file show generic Gutenberg patterns that include `wp:group` for illustration — in Drupal, replace `wp:group` wrappers with `wp:columns` containing a single `wp:column` child.

## Nested blocks

Blocks that accept inner content use `InnerBlocks` in their edit/save functions. In serialized HTML, inner blocks sit between the parent's wrapper HTML tag and its closing block comment.

### Nesting structure

```html
<!-- wp:parent-block {"parentAttr":"value"} -->
<div class="wp-block-parent-block">
    <!-- wp:child-block -->
    <p>Child content</p>
    <!-- /wp:child-block -->
    <!-- wp:another-child -->
    <p>More child content</p>
    <!-- /wp:another-child -->
</div>
<!-- /wp:parent-block -->
```

### Deeply nested example

The outer wrapper here uses `wp:columns` with a single `wp:column` child rather than `wp:group` — see the Drupal Gutenberg note at the top of this file. Inside it, a second `wp:columns` provides the two-up service layout.

```html
<!-- wp:columns {"className":"page-section"} -->
<div class="wp-block-columns page-section">
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:heading {"level":2} -->
        <h2>Our Services</h2>
        <!-- /wp:heading -->
        <!-- wp:columns -->
        <div class="wp-block-columns">
            <!-- wp:column -->
            <div class="wp-block-column">
                <!-- wp:image {"sizeSlug":"medium"} -->
                <figure class="wp-block-image size-medium">
                    <img src="/sites/default/files/service1.jpg" alt="Service 1" />
                </figure>
                <!-- /wp:image -->
                <!-- wp:heading {"level":3} -->
                <h3>Web Development</h3>
                <!-- /wp:heading -->
                <!-- wp:paragraph -->
                <p>Custom Drupal development services.</p>
                <!-- /wp:paragraph -->
            </div>
            <!-- /wp:column -->
            <!-- wp:column -->
            <div class="wp-block-column">
                <!-- wp:image {"sizeSlug":"medium"} -->
                <figure class="wp-block-image size-medium">
                    <img src="/sites/default/files/service2.jpg" alt="Service 2" />
                </figure>
                <!-- /wp:image -->
                <!-- wp:heading {"level":3} -->
                <h3>Design</h3>
                <!-- /wp:heading -->
                <!-- wp:paragraph -->
                <p>UX and visual design services.</p>
                <!-- /wp:paragraph -->
            </div>
            <!-- /wp:column -->
        </div>
        <!-- /wp:columns -->
    </div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

## Parent-child relationships

Blocks can enforce parent-child constraints:

- **`parent`** — Block can only be inserted as a direct child of specified blocks
- **`ancestor`** — Block can appear anywhere nested within specified blocks
- **`allowedBlocks`** — Restricts which blocks can be inserted as children
- **`templateLock`** — Controls whether inner blocks can be added, moved, or removed:
  - `"all"` — No blocks can be added, moved, or removed
  - `"insert"` — Blocks can be moved but not added or removed
  - `false` — No restrictions (default)

### Template (default inner blocks)

Blocks can specify a default inner block template:

```js
const TEMPLATE = [
    ['core/heading', { level: 3, placeholder: 'Card Title' }],
    ['core/paragraph', { placeholder: 'Card description...' }],
    ['core/button', {}],
];

// In edit function:
<InnerBlocks template={TEMPLATE} templateLock="insert" />
```

This creates the inner blocks automatically when the parent block is inserted.

## Block patterns

Block patterns are pre-built layouts that users insert as a starting point. In Drupal Gutenberg, patterns are defined in `.gutenberg.yml`.

### Defining patterns in `.gutenberg.yml`

```yaml
# In your theme's or module's .gutenberg.yml

__experimentalBlockPatternCategories:
  - name: hero
    label: "Hero Sections"
  - name: footer
    label: "Footer"
  - name: testimonials
    label: "Testimonials"

__experimentalBlockPatterns:
  - name: mytheme/hero-with-cta
    title: "Hero with Call to Action"
    description: "A full-width hero section with heading, text, and button"
    categories: ["hero"]
    content: |
      <!-- wp:cover {"url":"/themes/mytheme/images/hero-default.jpg","dimRatio":60,"align":"full"} -->
      <div class="wp-block-cover alignfull">
          <span aria-hidden="true" class="wp-block-cover__background has-background-dim-60 has-background-dim"></span>
          <img class="wp-block-cover__image-background" alt="" src="/themes/mytheme/images/hero-default.jpg" />
          <div class="wp-block-cover__inner-container">
              <!-- wp:heading {"level":1,"textColor":"white","align":"center"} -->
              <h1 class="has-white-color has-text-color has-text-align-center">Welcome to Our Site</h1>
              <!-- /wp:heading -->
              <!-- wp:paragraph {"align":"center","textColor":"white"} -->
              <p class="has-text-align-center has-white-color has-text-color">Discover what we have to offer.</p>
              <!-- /wp:paragraph -->
              <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
              <div class="wp-block-buttons">
                  <!-- wp:button -->
                  <div class="wp-block-button">
                      <a class="wp-block-button__link" href="/get-started">Get Started</a>
                  </div>
                  <!-- /wp:button -->
              </div>
              <!-- /wp:buttons -->
          </div>
      </div>
      <!-- /wp:cover -->

  - name: mytheme/two-column-text-image
    title: "Two Column: Text and Image"
    description: "Text on the left, image on the right"
    categories: ["hero"]
    content: |
      <!-- wp:columns -->
      <div class="wp-block-columns">
          <!-- wp:column -->
          <div class="wp-block-column">
              <!-- wp:heading {"level":2} -->
              <h2>About Us</h2>
              <!-- /wp:heading -->
              <!-- wp:paragraph -->
              <p>Tell your story here. This section pairs text with an image for visual interest.</p>
              <!-- /wp:paragraph -->
              <!-- wp:buttons -->
              <div class="wp-block-buttons">
                  <!-- wp:button -->
                  <div class="wp-block-button">
                      <a class="wp-block-button__link" href="/about">Learn More</a>
                  </div>
                  <!-- /wp:button -->
              </div>
              <!-- /wp:buttons -->
          </div>
          <!-- /wp:column -->
          <!-- wp:column -->
          <div class="wp-block-column">
              <!-- wp:image {"sizeSlug":"large"} -->
              <figure class="wp-block-image size-large">
                  <img src="/sites/default/files/placeholder.jpg" alt="" />
              </figure>
              <!-- /wp:image -->
          </div>
          <!-- /wp:column -->
      </div>
      <!-- /wp:columns -->

  - name: mytheme/testimonial-card
    title: "Testimonial Card"
    description: "A styled quote with attribution"
    categories: ["testimonials"]
    content: |
      <!-- wp:group {"backgroundColor":"pale-cyan-blue","className":"testimonial-card"} -->
      <div class="wp-block-group testimonial-card has-pale-cyan-blue-background-color has-background">
          <!-- wp:quote -->
          <blockquote class="wp-block-quote"><!-- wp:paragraph -->
          <p>This product changed how we work. Highly recommended!</p>
          <!-- /wp:paragraph -->
          <cite>Jane Smith, CEO of Example Corp</cite></blockquote>
          <!-- /wp:quote -->
      </div>
      <!-- /wp:group -->
```

### Pattern content rules

- The `content` field contains the exact block markup that will be inserted
- Use YAML `|` (literal block scalar) for multi-line content
- The content must be valid Gutenberg block markup
- Patterns can reference images/assets from the theme or default files directory
- Users can modify the pattern content after insertion

## Reusable blocks / synced patterns

Drupal Gutenberg has two types of reusable content:

### Synced patterns (formerly reusable blocks)
- Changes propagate to every instance across the site
- Stored as separate entities
- Inserted via `<!-- wp:block {"ref":123} /-->` where `123` is the entity ID

### Non-synced patterns
- Content is copied on insertion
- Each instance is independently editable after insertion
- These are the standard block patterns defined in `.gutenberg.yml`

## Common layout patterns

### Card grid (3 columns)

```html
<!-- wp:columns -->
<div class="wp-block-columns">
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:group {"className":"card"} -->
        <div class="wp-block-group card">
            <!-- wp:image {"sizeSlug":"medium"} -->
            <figure class="wp-block-image size-medium">
                <img src="/sites/default/files/card1.jpg" alt="" />
            </figure>
            <!-- /wp:image -->
            <!-- wp:heading {"level":3} -->
            <h3>Card Title</h3>
            <!-- /wp:heading -->
            <!-- wp:paragraph -->
            <p>Card description text.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:group -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:group {"className":"card"} -->
        <div class="wp-block-group card">
            <!-- wp:image {"sizeSlug":"medium"} -->
            <figure class="wp-block-image size-medium">
                <img src="/sites/default/files/card2.jpg" alt="" />
            </figure>
            <!-- /wp:image -->
            <!-- wp:heading {"level":3} -->
            <h3>Card Title</h3>
            <!-- /wp:heading -->
            <!-- wp:paragraph -->
            <p>Card description text.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:group -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column -->
    <div class="wp-block-column">
        <!-- wp:group {"className":"card"} -->
        <div class="wp-block-group card">
            <!-- wp:image {"sizeSlug":"medium"} -->
            <figure class="wp-block-image size-medium">
                <img src="/sites/default/files/card3.jpg" alt="" />
            </figure>
            <!-- /wp:image -->
            <!-- wp:heading {"level":3} -->
            <h3>Card Title</h3>
            <!-- /wp:heading -->
            <!-- wp:paragraph -->
            <p>Card description text.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:group -->
    </div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Full-width section with centered content

```html
<!-- wp:group {"align":"full","backgroundColor":"black"} -->
<div class="wp-block-group alignfull has-black-background-color has-background">
    <!-- wp:heading {"textAlign":"center","textColor":"white"} -->
    <h2 class="has-text-align-center has-white-color has-text-color">Section Title</h2>
    <!-- /wp:heading -->
    <!-- wp:paragraph {"align":"center","textColor":"white"} -->
    <p class="has-text-align-center has-white-color has-text-color">Description text centered below the heading.</p>
    <!-- /wp:paragraph -->
    <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
    <div class="wp-block-buttons">
        <!-- wp:button {"backgroundColor":"white","textColor":"black"} -->
        <div class="wp-block-button">
            <a class="wp-block-button__link has-black-color has-white-background-color has-text-color has-background">Call to Action</a>
        </div>
        <!-- /wp:button -->
    </div>
    <!-- /wp:buttons -->
</div>
<!-- /wp:group -->
```

### Sidebar layout (two-thirds / one-third)

```html
<!-- wp:columns -->
<div class="wp-block-columns">
    <!-- wp:column {"width":"66.66%"} -->
    <div class="wp-block-column" style="flex-basis:66.66%">
        <!-- wp:heading {"level":2} -->
        <h2>Main Content Area</h2>
        <!-- /wp:heading -->
        <!-- wp:paragraph -->
        <p>The main content of the page goes here with a wider column.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"33.33%"} -->
    <div class="wp-block-column" style="flex-basis:33.33%">
        <!-- wp:heading {"level":3} -->
        <h3>Sidebar</h3>
        <!-- /wp:heading -->
        <!-- wp:paragraph -->
        <p>Sidebar content, links, or widgets.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Centered reading column (15% / 70% / 15%)

For longer-form pages with a single narrative flow (annual reports, key publications), use a narrower reading column. Three columns — 15% empty, 70% content, 15% empty — center body text while full-width elements (hero, sticky nav, stats cards) break out of this pattern.

```html
<!-- wp:columns -->
<div class="wp-block-columns">
    <!-- wp:column {"width":"15%"} -->
    <div class="wp-block-column" style="flex-basis:15%"></div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"70%"} -->
    <div class="wp-block-column" style="flex-basis:70%">
        <!-- wp:heading {"level":2} -->
        <h2>Section Title</h2>
        <!-- /wp:heading -->
        <!-- wp:paragraph -->
        <p>Body text in the narrower center column for improved readability.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"15%"} -->
    <div class="wp-block-column" style="flex-basis:15%"></div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

This is responsive — on mobile the side columns collapse and content fills the viewport. Don't use this for index/reference pages where full-width content density is preferred.

### Combined background color + centered reading column

For sections that need both a background color AND a centered reading column, combine them in a single `wp:columns` block. This is cleaner than double-nesting a background wrapper inside centering columns.

```html
<!-- wp:columns {"backgroundColor":"undrr-light-grey","undrrUtilityClasses":["mg-container-full-width"]} -->
<div class="wp-block-columns has-undrr-light-grey-background-color has-background mg-container-full-width">
    <!-- wp:column {"width":"15%"} -->
    <div class="wp-block-column" style="flex-basis:15%"></div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"70%"} -->
    <div class="wp-block-column" style="flex-basis:70%">
        <!-- wp:heading -->
        <h2>Section with Background</h2>
        <!-- /wp:heading -->
        <!-- wp:paragraph -->
        <p>Content inside a full-width grey background with centered reading column.</p>
        <!-- /wp:paragraph -->
    </div>
    <!-- /wp:column -->
    <!-- wp:column {"width":"15%"} -->
    <div class="wp-block-column" style="flex-basis:15%"></div>
    <!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Video embed (YouTube)

```html
<!-- wp:embed {"url":"https://www.youtube.com/watch?v=VIDEO_ID","type":"rich","providerNameSlug":"youtube","responsive":true,"className":"wp-embed-aspect-16-9 wp-has-aspect-ratio"} -->
<figure class="wp-block-embed is-type-rich is-provider-youtube wp-block-embed-youtube wp-embed-aspect-16-9 wp-has-aspect-ratio">
    <div class="wp-block-embed__wrapper">
        https://www.youtube.com/watch?v=VIDEO_ID
    </div>
</figure>
<!-- /wp:embed -->
```

The URL appears both in the JSON attributes and as text content inside the wrapper div.

## Publication landing page archetype

For annual reports and key publications that companion a downloadable PDF, use a distinct page structure:

1. **Hero header** with video or cover image (full-width)
2. **Sticky navigation** bar (full-width `wp:html` block)
3. **Foreword/introduction** in centered reading column (15/70/15)
4. **Key statistics** using stats-card blocks (full-width)
5. **Thematic sections** (H2) with story previews truncated to 2-3 paragraphs, each followed by a "Read more" link to the PDF
6. **Quote highlights** for official quotes
7. **Cross-cutting themes** as vertical content cards linking to thematic pages
8. **Download footer** with book cards for PDF and annexes
9. **"More" footer** with vertical content cards linking to related pages
10. Alternating white/grey section backgrounds for visual rhythm

Story truncation pattern:

```html
<!-- wp:paragraph -->
<p>First 2-3 paragraphs of the story from the report...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><a href="/media/XXXXX/download">Read more in the full report (PDF) →</a></p>
<!-- /wp:paragraph -->
```
