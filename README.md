# drupal-gutenberg-llm-skills

Skills for [Drupal Gutenberg](https://www.drupal.org/project/gutenberg), Drupal's implementation of the WordPress Gutenberg block editor. Installable as a plugin in both [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli).

## Skills

### `drupal-gutenberg-markup`

Generates valid Drupal Gutenberg HTML block markup. Covers `<!-- wp:blockname -->` comment syntax, attributes, nested blocks, column layouts, and block patterns.

Both tools auto-detect this skill when you ask about creating Gutenberg content. You can also invoke it explicitly:

- **Claude Code:** `/drupal-gutenberg:drupal-gutenberg-markup`
- **Copilot CLI:** include `/drupal-gutenberg-markup` in your prompt, e.g. _"Use the /drupal-gutenberg-markup skill to create a hero section"_

Example prompts:
- "Create a Drupal Gutenberg page with a hero image, two columns, and a call-to-action button"
- "Write Gutenberg block markup for a testimonials section with three cards"
- "Convert this HTML into valid Gutenberg block markup"

### `drupal-gutenberg-dev`

Helps build custom Gutenberg blocks and modules for Drupal. Covers module structure, `.gutenberg.yml` configuration, ES6/Babel build toolchain, block registration, Twig server-side rendering, and media integration.

Both tools auto-detect this skill when you're working on Gutenberg module code. You can also invoke it explicitly:

- **Claude Code:** `/drupal-gutenberg:drupal-gutenberg-dev`
- **Copilot CLI:** include `/drupal-gutenberg-dev` in your prompt

Example prompts:
- "Create a custom Gutenberg block module that displays a featured content carousel"
- "Set up server-side rendering for my custom block with a Twig template"
- "What's the Twig template naming convention for Drupal Gutenberg?"

## Installation

### Claude Code

**Quick start (single session):**

```bash
claude --plugin-dir /path/to/drupal-gutenberg-llm-skills
```

**Permanent install via local marketplace:**

```bash
# Register the repo as a local marketplace (one-time setup)
/plugin marketplace add /path/to/drupal-gutenberg-llm-skills

# Install the plugin
/plugin install drupal-gutenberg@drupal-gutenberg-llm-skills
```

### GitHub Copilot CLI

**Install from a local clone:**

```bash
copilot plugin install /path/to/drupal-gutenberg-llm-skills
```

**Install directly from GitHub** (no local clone needed):

```bash
copilot plugin install owner/drupal-gutenberg-llm-skills
```

**Verify the install:**

```bash
copilot plugin list
```

Or inside an interactive session:

```
/skills list
```

## Development

If you're working on the skill content in this repo:

1. **Clone the repo** and make your changes to the Markdown files under `skills/`.

2. **Test locally** — load directly from your working copy without an install step:

   - Claude Code: `claude --plugin-dir ./`
   - Copilot CLI: `copilot --plugin-dir ./`

3. **Iterate** — edit skill files, restart the tool, and verify the skills activate and produce correct output.
   - In Copilot CLI you can also run `/skills reload` during a session after adding or editing a skill.

4. **Publish** — push to the repo. To pick up changes in Copilot CLI after a pull: `copilot plugin install /path/to/repo` (re-run install to refresh the cache).

---

## Maintaining this plugin

> **Note (March 2026):** Both Claude Code and Copilot CLI converged on a very similar plugin/skill structure, which is why this repo works for both without duplication. Both systems are actively evolving — treat the specifics below as current best-effort, not permanent truth.

### What both tools share

Because of deliberate format alignment between the two tools, a single set of files serves both:

| File | Used by | Notes |
|---|---|---|
| `.claude-plugin/plugin.json` | Both | Both tools recognize this path as a valid plugin manifest location |
| `.claude-plugin/marketplace.json` | Both | Both tools recognize this path for marketplace registration |
| `skills/*/SKILL.md` | Both | Default skill convention for both; the `name` and `description` frontmatter fields drive auto-detection in both |
| Supporting `.md` files in `skills/*/` | Both | Referenced from `SKILL.md` body; loaded into context by both tools |
| `CLAUDE.md` | Both | Claude Code uses it as plugin-level instructions; Copilot CLI uses it as project-level instructions — keep it accurate for both contexts |

### What to keep in sync when editing

When you change any of the following, check that both tools still behave correctly:

- **`skills/*/SKILL.md` — `description` field:** This is the auto-detection trigger for both tools. If it's vague or wrong, neither tool will load the skill at the right moment. Write it as a trigger condition: _"Use when the user asks to…"_
- **`skills/*/SKILL.md` — `name` field:** Must be unique across all installed plugins. Changing it is a breaking rename — users of the old name will lose auto-detection.
- **`.claude-plugin/plugin.json` — `name` field:** Changing this is a breaking rename for installed users of both tools.
- **Supporting `.md` files:** Both tools inject these into context when the skill loads. Inaccurate examples (especially Drupal-vs-WordPress API differences) are the primary defect vector.

### Key differences between the two tools

| Aspect | Claude Code | Copilot CLI |
|---|---|---|
| Explicit skill invocation | `/plugin-name:skill-name` | `/skill-name` inline in a prompt |
| List available skills | `/plugin list` | `/skills list` |
| Reload local edits | Restart with `--plugin-dir` | `copilot plugin install ./path` (refreshes cache) or `/skills reload` in session |
| `version` in `SKILL.md` | Recognized | Ignored (extra fields are safe) |
| Agents (`.agent.md`) | Not supported | Supported |
| Hooks (`hooks.json`) | Not supported | Supported |
| MCP server config | Not supported | Supported |

---

## Porting a Claude Code skill to Copilot CLI

> These notes reflect the state of both systems as of **March 24, 2026**. Both plugin systems are actively evolving — check the current documentation for each tool before relying on these steps.

If you have an existing Claude Code skill plugin and want it to also work in Copilot CLI, here is what to check. In most cases no content changes are needed — only metadata and file placement matter.

### 1. Verify the manifest is in a shared location

Copilot CLI looks for `plugin.json` in these locations (in order):

1. Root `plugin.json`
2. `.github/plugin/plugin.json`
3. `.claude-plugin/plugin.json` ← Claude Code's default

If your manifest is already at `.claude-plugin/plugin.json`, **no move is needed**.

### 2. Add an explicit `skills` field to `plugin.json`

Copilot CLI defaults to `skills/` if the field is absent, but being explicit avoids ambiguity:

```json
{
  "name": "my-plugin",
  "description": "…",
  "version": "1.0.0",
  "skills": "skills/"
}
```

### 3. Check `SKILL.md` frontmatter

Copilot CLI requires `name` and `description`. The `version` field is Claude Code–specific but harmless to leave in place:

```yaml
---
name: my-skill           # required by both; lowercase, hyphens only
description: >           # required by both — drives auto-detection in both tools
  Use when the user asks to …
version: 1.0.0           # Claude Code only; Copilot CLI ignores it safely
---
```

The `description` field is the most important thing to get right. Both tools use it to decide whether to load the skill for a given prompt.

### 4. Test the install

```bash
# Install from a local directory
copilot plugin install /path/to/your-plugin

# Confirm it loaded
copilot plugin list

# In an interactive session, confirm skills are visible
/skills list
```

### 5. What won't automatically cross over

These Copilot CLI features have no Claude Code equivalent — they are safe to add for Copilot users without affecting Claude Code:

- **Custom agents** (`.agent.md` files in `agents/`) — Copilot CLI only
- **Hooks** (`hooks.json`) — Copilot CLI only
- **MCP server config** (`.mcp.json`) — Copilot CLI only

`CLAUDE.md` is read by both tools, but with slightly different scope: Claude Code treats it as plugin-level instructions, while Copilot CLI treats it as project-level custom instructions. Write content that makes sense in both contexts.

### Reference docs

- [Creating skills for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-skills)
- [Creating a plugin for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)
- [GitHub Copilot CLI plugin reference](https://docs.github.com/en/copilot/reference/cli-plugin-reference)
- [Claude Code plugins](https://docs.anthropic.com/en/docs/claude-code)

---

## Technical details

These skills target **Drupal Gutenberg 3.x** (bundles [Gutenberg 16.7](https://make.wordpress.org/core/2023/09/28/whats-new-in-gutenberg-16-7-27-september/) (27 Sep 2023) / WordPress 6.4). Compatible with Drupal 10 and 11.

- Blocks use the `wp:` prefix (not `drupal:`), keeping WordPress block compatibility
- Content is stored as HTML in `node__body.body_value`
- Custom blocks are declared in `.gutenberg.yml` (not WordPress's `block.json`)
- Server-side rendering uses Twig templates named `gutenberg-block--namespace--block-name.html.twig`
- Dynamic blocks need `hook_theme()` registration and block-specific preprocess hooks (`hook_preprocess_gutenberg_block__MODULE__BLOCK`) — this is only for blocks rendered entirely server-side via Twig, not client-side blocks that save HTML or rehydrate via JS
- ES6 source files use the `.es6.js` extension, compiled via Babel to `.js`
- Start by copying `gutenberg/modules/example_block` as boilerplate

## Resources

- [Drupal Gutenberg module](https://www.drupal.org/project/gutenberg)
- [Drupal Gutenberg documentation](https://www.drupal.org/docs/contributed-modules/gutenberg)
- [Create Custom Blocks guide](https://www.drupal.org/docs/contributed-modules/gutenberg/create-custom-blocks)
- [WordPress Block API reference](https://developer.wordpress.org/block-editor/developers/block-api/) (underlying API used by Drupal Gutenberg)
