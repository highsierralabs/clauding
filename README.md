# Clauding — Subject-Themed Spinner Verbs 🌀

Theme Claude Code's loading spinner around any subject you like. Run
`/clauding baking` and the spinner stops saying `Pondering…` and starts saying
`Kneading…`, `Proofing…`, `Blitzing…`. Works for hobbies, fandoms, sports,
sciences — anything with a vocabulary.

No API keys, no external servers: the skill generates a tight batch of
present-participle verbs for your subject and installs them into Claude Code's
(undocumented) `spinnerVerbs` key in `~/.claude/settings.json`.

## Install

**As a plugin (recommended):**

```
/plugin marketplace add highsierralabs/clauding
/plugin install clauding@highsierralabs
```

The skill is then invoked as `/clauding:clauding <subject>`.

**Manual copy (bare `/clauding` name):**

Copy `plugins/clauding/skills/clauding/` into `~/.claude/skills/` (user-wide) or
`<project>/.claude/skills/` (per-project):

```
~/.claude/skills/clauding/SKILL.md
```

The skill is then invoked as plain `/clauding <subject>`.

## Usage

```
/clauding <subject> [replace|append]   # theme the spinner
/clauding baking                       # e.g. Kneading…, Proofing…, Mixing…
/clauding "star trek" append           # mix themed verbs into the built-in pool
/clauding reset                        # restore Claude Code's built-in verbs
```

- **replace** (default) — your themed verbs are the only ones shown.
- **append** — your verbs join the ~185 built-in defaults (always fresh, but
  your theme becomes a minority of the pool).
- If a subject yields fewer than ~20 strong verbs in defaulted-replace mode, the
  skill pauses and asks whether you'd rather append, replace anyway, or extend
  the list — a small pool in replace mode starts repeating within a session.
- Changes take effect immediately, no restart needed. Your previous
  `settings.json` is backed up to `~/.claude/settings.json.bak` before every
  change.

## How it works

Claude Code reads an undocumented `spinnerVerbs` key from
`~/.claude/settings.json`:

```json
"spinnerVerbs": {
  "mode": "replace",
  "verbs": ["Kneading", "Proofing", "Blitzing"]
}
```

Verbs are stored bare — the UI appends the animated `…` itself. The skill
handles verb generation (quality-gated, ~15–40 per subject), mode resolution,
backup, and safe JSON editing; `/clauding reset` deletes the key to fall back to
the built-in defaults.

## Credits

Adapted from
[did-you-know-plugin](https://github.com/DanielPodolsky/did-you-know-plugin) by
**Daniel Podolsky**, which turns the spinner into "Did You Know" facts. The
spinner-text-via-`spinnerVerbs` mechanism and the theme-the-spinner concept
originate there — this project re-aims it at subject-themed verbs that keep the
native spinner cadence. Thank you, Daniel.

## License

[MIT](LICENSE) — portions adapted from did-you-know-plugin (MIT).
