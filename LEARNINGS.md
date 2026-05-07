# Learnings

Hard-won lessons from building and maintaining this plugin. Captured during the
2026-05 audit sweep — pull more in over time. Cite a commit or PR when you can.

---

## Plugin packaging gotchas

### `plugin.json` must not have a `skills` field
Claude Code rejects the manifest with a validation error if it does. Both
Claude Code and the Copilot CLI auto-discover skills from the `skills/`
directory — leave it out. Discovered during the v1.2.0 release
([44ef603](../../commit/44ef603)).

### Dual-CLI compatibility is mostly free, but watch the tool names
This plugin works with both Claude Code (`Skill` tool) and GitHub Copilot CLI
(`skill` tool). Skill files use Claude Code's tool names; references to other
tools (`Read`, `Edit`, `Bash`, etc.) need a platform-aware shim or a note in
the skill body. See `references/copilot-tools.md` in the wider Superpowers
ecosystem for the mapping.

### There is no build, no test, no runtime
Skills are pure Markdown. The only "validation" is human-eyeballing the
examples against the gotchas below. Adding tests would mean spinning up a
real Drupal site and asserting `<!-- wp:* -->` markup loads — out of scope
for this repo.

---

## Drupal Gutenberg authoring gotchas (the content this plugin documents)

These are facts that surprise people coming from WordPress. The skill files
treat them as authoritative; this section is the short list for orienting
yourself or a future maintainer.

### `wp:group` is broken in Drupal Gutenberg — use `wp:columns` with a single `wp:column`
The block prefix is still `wp:`, but the editor has rendering quirks the
upstream WordPress core doesn't share. `wp:group` is the most painful one;
swap it for `wp:columns` containing a single `wp:column` and style the
wrapper. See
[`patterns-and-nesting.md`](skills/drupal-gutenberg-markup/patterns-and-nesting.md)
for examples; the PR thread that surfaced this is on GitHub.

### `.gutenberg.yml`, not `block.json`
Drupal Gutenberg uses `.gutenberg.yml` with `libraries-edit:`,
`libraries-view:`, and `dynamic-blocks:`. The WordPress conventions
`block.json` and `blocks: render: true` do not work here.

### Translations: `Drupal.t`, not `wp.i18n.__`
Inside JS skills, use `const __ = Drupal.t;`. Do not import `@wordpress/i18n`
— it is not wired up in the Drupal editor stack.

### Library dependency is `gutenberg/edit-node`
Custom block modules need this in their `*.libraries.yml`. The WordPress
asset, `gutenberg/editor`, does nothing here.

### `wp.serverSideRender` is the default export
WordPress core moved it to `wp.components.ServerSideRender` long ago, but
Drupal Gutenberg 3.x still exposes it at `wp.serverSideRender` as a default
export. Server-side rendered blocks crash silently if you import the wrong
path.

### MediaUpload returns Drupal objects, not WP attachments
Expect `media.alt_text` (not `media.alt`) and `media.media_entity.id` (not
`media.id`). Field names follow Drupal conventions.

### Build tooling is Drupal-side
Custom block modules build with `drupal-js-build` and ship translations
through `drupal-gutenberg-translations`. `@wordpress/scripts` will not work
out of the box.

### List blocks require `<!-- wp:list-item -->` inner-block format
A `<!-- wp:list -->` with raw `<li>` content gets stripped at save. The
list-item inner-block wrapper is mandatory.

### Pattern APIs still carry the `__experimental` prefix in 3.x
`__experimentalBlockPatternCategories` and `__experimentalBlockPatterns` —
the prefix has been dropped upstream but Drupal Gutenberg 3.x ships
[Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/)
(September 2023) and still wants the experimental name.

### Drupal Gutenberg 3.x is not bleeding-edge Gutenberg
3.x bundles Gutenberg 16.7. Several upstream features (block bindings,
distraction-free mode toggles, newer pattern primitives) are simply not
available. Verify against the bundled version, not the latest WordPress
trunk, before recommending anything new.

---

## Conventions that look weird but are intentional

### The skills are conservative on triggers
`drupal-gutenberg-markup` only activates on requests that mention Drupal
Gutenberg by name or the `<!-- wp:` block syntax. We do not auto-fire on
plain "Gutenberg" requests because most of the world means WordPress
Gutenberg, where these gotchas would actively mislead.

### CLAUDE.md and README.md mirror each other on purpose
README serves humans installing the plugin; CLAUDE.md is editing guidance
for AI agents working inside the repo. They overlap deliberately so
contributors landing in either entry point get the right warnings.
