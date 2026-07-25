---
name: clauding
description: Replace or extend Claude Code's loading-spinner verbs with subject-themed present-participle verbs (e.g. baking -> Kneading…, Mixing…, Proofing…). Use when the user runs /clauding with a subject, asks to change / refresh / theme their spinner verbs, or asks to reset / restore the spinner to Claude Code's built-in defaults (/clauding reset). Accepts an optional trailing mode: replace (default) or append.
argument-hint: [subject] [replace|append]
allowed-tools: Read, Write, Edit, Bash, WebSearch
disable-model-invocation: false
---

# Spinner Clauding — Subject-Themed Spinner Verbs

Your job: turn the chosen subject into a batch of short, present-participle verbs
that match the cadence of Claude Code's native whimsical spinner verbs
(`Cooking…`, `Pondering…`, `Flibbertigibbeting…`), then install them into the
user's `~/.claude/settings.json` under the `spinnerVerbs` key.

**Format facts (verified, the feature is undocumented):**
- The key shape is `"spinnerVerbs": { "mode": "replace" | "append", "verbs": [ ... ] }`.
- Claude Code appends the animated "…" itself, so store **bare** verbs — never put a
  trailing "…" in the string (it would double the dots).
- There are ~185 built-in default verbs. In `replace` mode your verbs are the *only*
  ones shown. In `append` mode your verbs join the ~185 defaults, so they become a
  minority of the pool (~N in 185+N).
- Present participles ("-ing") are the convention that makes a string read as a spinner verb.

The raw invocation is: **$ARGUMENTS**

## Step 0 — Parse the request

1. **Restore check first.** Take `$ARGUMENTS`, trim and lowercase it. If it is exactly one
   of `reset`, `restore`, `default`, `defaults`, or `off`, this is a RESTORE request — skip
   verb generation entirely and go to the **Restore** section below.
2. Otherwise, inspect the LAST whitespace-delimited token of `$ARGUMENTS`, case-insensitively:
   - if it is exactly `append` or `replace`, that token is the **explicit mode**, and the
     **subject** is everything before it (trim trailing space).
   - otherwise the mode is **replace, defaulted**, and the subject is the whole of `$ARGUMENTS`.
   Remember whether the mode was *explicit* or *defaulted* — Step 3 branches on it.
   (A subject genuinely ending in the word "append"/"replace" is vanishingly unlikely;
   if a user ever hits that, they can reorder the words.)
3. If the resulting subject is empty, do NOT proceed. Print the usage and stop:
   `/clauding <subject> [replace|append]`  — e.g. `/clauding baking`, `/clauding football append`
   `/clauding reset`                        — restore Claude Code's built-in spinner verbs

## Restore — return to Claude Code's built-in spinner verbs

Triggered when the request is `reset` / `restore` / `default` / `off`. Do NOT generate any verbs.

1. If `~/.claude/settings.json` does not exist, there is nothing to restore — tell the user
   they are already on the built-in defaults, and stop.
2. Back up first: `cp ~/.claude/settings.json ~/.claude/settings.json.bak`
3. Read `~/.claude/settings.json`. If it has **no** `spinnerVerbs` key, tell the user there
   are no custom verbs to remove (already on the built-in defaults) and stop.
4. **Delete the entire `spinnerVerbs` key**, preserving every other key exactly as-is. With
   no `spinnerVerbs` key present, Claude Code falls back to its ~185 built-in verbs — this is
   the clean "stop using custom clauding" action and does not depend on what the backup holds.
   The final file MUST stay valid JSON.
5. Write the file back.
6. Confirm, briefly: ✅ spinner restored to Claude Code's built-in verbs; takes effect
   **immediately — no restart needed**; the prior settings are backed up at
   `~/.claude/settings.json.bak`.

## Step 1 — Generate the verbs

Produce themed verbs for the subject. Follow ALL of these:

1. **Present participle.** Each entry is a gerund / "-ing" verb, or a short evocative
   "-ing" phrase. Good: `Kneading`, `Transporting`, `Blitzing`, `Boldly Going`, `Making it so`.
2. **Bare strings, no trailing "…".** The UI adds the animated dots.
3. **Short.** Keep each roughly <= 24 characters so it renders cleanly in a terminal.
   One- or two-word entries are ideal; an occasional three-word phrase is fine for flavor.
4. **In-cadence and recognizable.** Each should sound like something the agent could
   plausibly be *doing*, and land for someone who knows the subject. Mix the obvious
   (`Mixing`, `Beaming`) with the deeper cut (`Proofing`, `Mind-melding`) so it never
   feels like the same three words on a loop.
5. **Quality over count — do NOT pad.** Target **~25**. Hard floor **15**. Soft ceiling **~40**.
   Generate the strongest verbs first and stop when the next one would be a stretch or a
   near-duplicate. A tight set of 16 good verbs beats 35 padded ones. Procedural / hobby
   subjects (baking, football, woodworking, gardening) reach 30–40 easily; narrow fandoms
   often top out near 15–25 before they strain — that is expected, not a failure.
6. **No duplicates,** and no two entries that are the same verb lightly reworded.
7. **Verify uncertain proper nouns.** For fandom or technical subjects, if a term's
   spelling or in-universe accuracy is uncertain, confirm with WebSearch before including
   it. Do not ship a misspelled franchise term.

Let **N** = the number of verbs you kept.

## Step 2 — Back up current settings (safety first)

Before touching anything, if `~/.claude/settings.json` exists, copy it to a backup so the
user can always roll back. Use Bash:
`cp ~/.claude/settings.json ~/.claude/settings.json.bak`

## Step 3 — Resolve the mode (the <20 rule lives here)

Use the mode from Step 0 and N from Step 1:

| Mode (from Step 0)     | N >= 20          | N < 20                                                                 |
|------------------------|------------------|-----------------------------------------------------------------------|
| `append` (explicit)    | install `append` | install `append`                                                      |
| `replace` (explicit)   | install `replace`| install `replace`, **plus** a one-line heads-up that a sub-20 set in replace repeats within a session and they can rerun with `append` |
| `replace` (defaulted)  | install `replace`| **STOP — do not write yet.** Surface the choice below and WAIT        |

When you hit the **defaulted-replace + N < 20** cell, present this to the user
(fill in N and the subject), then stop and wait for their reply:

> Generated **N** verbs for *subject*. Below ~20, a `replace` spinner starts repeating
> within a single session. Your options:
> - reply **append** — mix these into Claude Code's ~185 built-in verbs (always fresh,
>   but your theme becomes a minority, ~N in 185+N)
> - reply **replace** — install just these anyway (fully themed, but repeats sooner)
> - reply **more** — I'll try to extend the list past 20 (may include a few stretchier verbs)

Do NOT modify `settings.json` until they answer. Then:
- **append** / **replace** → proceed to Step 4 with that mode.
- **more** → regenerate with additional verbs and re-run Step 3. If it still cannot clear
  20, offer only **append** or **replace** (drop **more**).

## Step 4 — Install into settings.json

1. Read `~/.claude/settings.json`. If it does not exist, create a minimal file containing
   only the `spinnerVerbs` key.
2. Set `spinnerVerbs` to:
   ```json
   "spinnerVerbs": {
     "mode": "<resolved mode>",
     "verbs": [ ...the N verbs... ]
   }
   ```
3. **Preserve every other key in the file exactly as-is** (permissions, statusLine,
   enabledPlugins, theme, etc.). Only `spinnerVerbs` changes — prefer editing just that key.
   The final file MUST be valid JSON; you are responsible for correct escaping.
4. Write the file back.

## Step 5 — Confirm (keep it short and upbeat)

Tell the user:
- ✅ their spinner now uses *subject* verbs in `<resolved mode>` mode (N verbs),
- show **6–8 sample verbs** so they get a taste,
- note: **new verbs take effect immediately — no restart needed** (you may need to trigger the spinner once to see them cycle),
- remind them: rerun `/clauding <subject>` to regenerate, `/clauding <other subject>` to switch,
  append `replace` or `append` to force a mode, and run `/clauding reset` to drop the custom
  verbs and return to Claude Code's built-in spinner; their previous settings are backed up
  at `~/.claude/settings.json.bak`.

---

*Adapted from [did-you-know-plugin](https://github.com/DanielPodolsky/did-you-know-plugin) by Daniel Podolsky (MIT).*
