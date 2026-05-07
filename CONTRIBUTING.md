# Contributing

Thanks for your interest. This is a small Claude Code / GitHub Copilot CLI plugin shipping two Markdown-only skills for Drupal Gutenberg. There is no build step, no runtime code, and no test suite — skills are loaded by the AI tool at prompt time.

## Filing issues

Open an issue at https://github.com/khawkins98/drupal-gutenberg-llm-skills/issues. Most useful issue types:

- **Wrong example** — link the file/line, paste the wrong claim, give the correction.
- **Missing pattern** — what you tried, what wasn't covered, what the right answer was.
- **Drupal Gutenberg version drift** — the skills target Gutenberg 3.x (bundling WP Gutenberg 16.7, 2023). If you're seeing different behaviour on a newer release, please flag the version.

## Proposing changes

1. Fork and branch off `main`.
2. Edit the relevant `SKILL.md` or supporting `.md` under `skills/<skill-name>/`. Keep examples compact and copy-pasteable.
3. If you change a fact mirrored in `README.md`, update both.
4. Open a draft PR while you iterate.

## What to watch when editing

The Drupal-specific divergences from WordPress are the main defect vector. When adding examples, verify against:

- Block prefix is `wp:` (yes, even in Drupal).
- `.gutenberg.yml` uses `libraries-edit:` / `libraries-view:` / `dynamic-blocks:` — not the WordPress shapes.
- Translations: `const __ = Drupal.t;` (not `wp.i18n.__`).
- `wp:group` is not supported in Drupal Gutenberg — use `wp:columns` instead. The `patterns-and-nesting.md` file documents this rule, and the skills must follow it.
- Don't add a `skills` field to `.claude-plugin/plugin.json` — Claude Code rejects it.

The CLAUDE.md file lists the full set of divergences.

## Branch and commit style

- Branches: descriptive, e.g. `fix/wp-group-examples`, `feat/patterns-skill`.
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`) — match recent history.

## Review

Best-effort. Drupal-Gutenberg-specific corrections are especially welcome.

## License

GPL-2.0 (matching the upstream Drupal Gutenberg module). See [LICENSE](LICENSE).
