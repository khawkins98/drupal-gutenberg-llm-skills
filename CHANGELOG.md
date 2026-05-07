# Changelog

All notable changes to this plugin are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[Semantic Versioning](https://semver.org/spec/v2.0.0.html). The canonical
version source is `.claude-plugin/plugin.json`.

## [Unreleased]

### Added
- `CONTRIBUTING.md` — issue / PR / maintenance guidance.
- `LEARNINGS.md` — captures Drupal Gutenberg authoring gotchas and plugin
  packaging quirks discovered while building this plugin.
- `CHANGELOG.md` — this file.

### Changed
- `patterns-and-nesting.md` — every `wp:group` example replaced with the
  Drupal-compatible `wp:columns` + single `wp:column` wrapper, matching the
  warning callout at the top of the file.

## [1.2.1] — 2026-03-26

### Added
- Drupal Gutenberg 3.x serialization quirks documented in the markup skill
  (separator `opacity:css`, button `wp-element-button` class, figcaption
  `wp-element-caption`, list-item inner blocks, card `mediaURL` in HTML
  only). [`99d0037`](../../commit/99d0037)

## [1.2.0] — 2026-03-25

### Fixed
- Removed the `skills` field from `plugin.json` — Claude Code rejected the
  manifest with a validation error. Both Claude Code and Copilot CLI
  auto-discover skills from the `skills/` directory, so the field was
  redundant. [`44ef603`](../../commit/44ef603)

## [1.1.0] — 2026-03-25

### Added
- GitHub Copilot CLI compatibility — same skill files now activate under
  Copilot's `skill` tool alongside Claude Code's `Skill` tool.
  [`57f21d8`](../../commit/57f21d8)
- Publication page patterns, centered columns, and the explicit `wp:group`
  warning that became the recurring theme of this plugin's documentation.
  [`0b11b65`](../../commit/0b11b65)

## [1.0.0] — 2026-02-08

### Added
- Two skills: `drupal-gutenberg-markup` (block markup generation) and
  `drupal-gutenberg-dev` (custom block module development).
  [`aa39bde`](../../commit/aa39bde)

[Unreleased]: ../../compare/main...HEAD
[1.2.1]: ../../releases/tag/v1.2.1
[1.2.0]: ../../releases/tag/v1.2.0
[1.1.0]: ../../releases/tag/v1.1.0
[1.0.0]: ../../releases/tag/v1.0.0
