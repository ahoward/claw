# review-config prompt

Review your own Claude Code configuration for staleness or missed improvements,
using Gemini as an adversarial agent across 3 rounds.

---

## 1. gather config

Read and summarize the following into a single CONFIG block:

- `~/.claude/settings.json`
- `~/.claude/settings.local.json`
- `~/.claude/plugins/installed_plugins.json`
- any `CLAUDE.md` in `~` or `~/.claude/`
- any files in `~/.claude/commands/`

For each item note: what it is, what it does, when it was last changed.

## 2. gather recent updates

```bash
cat $(npm root -g 2>/dev/null)/@anthropic-ai/claude-code/CHANGELOG.md 2>/dev/null | head -400
```

Summarize notable recent changes as UPDATES.

## 3. self-review

Before consulting Gemini, write your own honest assessment: for each config item,
is it current, potentially stale, a workaround for something now fixed, or missing
something obvious? Call this SELF_REVIEW.

## 4. round 1 — gemini cold read

```bash
gemini -p "$(cat <<'PROMPT'
You are an adversarial config reviewer. Be specific and skeptical, not polite.

Given this Claude Code config and recent changelog, identify:
- settings that are redundant, obsolete, or have better alternatives
- missing settings that seem obviously useful
- permissions that are too broad, too narrow, or suspicious
- plugins that may be stale
- anything that looks like a workaround for a now-fixed issue

CONFIG:
__CONFIG__

RECENT_UPDATES:
__UPDATES__
PROMPT
)" 2>/dev/null
```

(substitute __CONFIG__ and __UPDATES__ before running)
Capture output as ROUND1.

## 5. round 2 — gemini challenges the self-review

```bash
gemini -p "$(cat <<'PROMPT'
Here is my self-assessment (SELF_REVIEW) and your prior critique (ROUND1).

Find:
- things my self-review got wrong or missed
- items from ROUND1 you want to retract or soften
- any new concerns not in ROUND1

SELF_REVIEW:
__SELF_REVIEW__

ROUND1:
__ROUND1__
PROMPT
)" 2>/dev/null
```

Capture as ROUND2.

## 6. round 3 — gemini final recommendations

```bash
gemini -p "$(cat <<'PROMPT'
Synthesize everything into a final prioritized action list:

CRITICAL   — broken or a security concern
RECOMMENDED — clear improvements, low risk
OPTIONAL   — matters of preference
LEAVE_ALONE — looked suspicious but is actually fine

For each item: what to change, why, and the exact edit or command.

CONFIG: __CONFIG__
ROUND1: __ROUND1__
ROUND2: __ROUND2__
PROMPT
)" 2>/dev/null
```

Capture as FINAL.

## 7. present synthesis

Show the user:

1. what gemini flagged that you agree with — proposed before/after diffs
2. what gemini flagged that you disagree with — your reasoning
3. what you caught that gemini missed

Do not change any files until the user confirms.
