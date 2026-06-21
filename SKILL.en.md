---
name: luu
version: "2.1"
updated: "2026-06-21"
description: >
  Session close protocol for a personal Obsidian-style knowledge vault. Invoke: /luu, "save session", "close session", "persist session".
  Five ordered steps: extract and sanitize -> decisions.md (idempotent) -> active.md (read-before-write, .bak, mtime-check) -> daily note -> capture the AI-OS area if the session touched it.
metadata:
  author: Le Cong Hoang
  created: "2026-05-03"
  vault: "<VAULT_ROOT>"
---

# luu - Session Close (Knowledge Vault)

Close a session cleanly: persist the vault's memory in a single command.
Invoke with: `/luu`, or "save session", or "close session".

**Frame for this run:** you are at the end of a session. The goal is to persist five memory items into the vault. What you need from the user is usually nothing, unless the mtime looks wrong or a directory is missing, in which case you stop and ask. You are done when active.md has a fresh .bak and is larger than zero bytes, decisions.md carries no duplicate day block, the daily note exists, and the final Output prints real numbers instead of empty claims.

**Vault root:** `<VAULT_ROOT>`
Every path below is relative to this root. When you run bash, use absolute paths, and if a path contains spaces, remember to quote it.

---

## Why the guardrails exist (read this before you trim for brevity)

This vault is not under version control. A lost file is gone for good, with no way back. That is the whole reason every overwrite step must take a .bak first, and every append step must be idempotent, meaning a re-run never duplicates. This is the minimum price for a session-close command that does not destroy its own vault. It is not over-engineering. Do not remove the .bak, do not remove the mtime-check, do not remove the grep just because they look redundant.

---

## When to use it
- At the end of every session.
- Just before a context limit or a compact.
- Right after an important decision that needs to be persisted.

---

## The order, and do NOT reorder the steps

### Step 0: Preflight (read-only, cheap, no writes)

Before you touch any write command, check that the three parent directories exist:
```bash
V="<VAULT_ROOT>"
test -d "$V/00_SYSTEM" && test -d "$V/40-archive/daily-notes" && test -d "$V/20-areas/AI-OS" && echo "preflight OK" || echo "preflight FAIL"
```
If it says FAIL: stop, tell the user which directory is missing, and do not create a new path yourself. The vault structure may have been renamed, and auto-creating a folder means you would be writing into a dead location.

Record the mtime of active.md so Step 3 has something to compare against:
```bash
stat -f %m "$V/00_SYSTEM/active.md" 2>/dev/null
```
Hold on to this number, call it `MTIME_START`. If the file does not exist yet, leave it empty, and Step 3 will treat it as a fresh create.

---

### Step 1: Extract from the session, with em-dash SANITIZE (do this BEFORE writing anything)

Read the whole session again. Pull out four things.

**Decisions:** what the user explicitly decided. Yes or no, chose A over B, set a new rule, killed an approach. What does not count: discussion, open options, brainstorming that reached no conclusion.

**Active state:** which projects are open, what the real blockers are, where the current focus sits.

**Daily summary:** what actually got done today. New facts, decisions, questions still open.

**AI-OS touch:** did this session touch the AI ecosystem, meaning doctrine, generators, skills, MCP, memory, or the membrane between sessions? If so, keep it aside for Step 5 to capture.

**SANITIZE (mandatory, do it right here):** before you touch any write command, scan everything you are about to write and replace every long dash (the em-dash, U+2014) with a comma, a colon, or a sentence split. Here is why. A hook (`emdash-block.py`) hard-blocks any write that contains an em-dash, exiting with code 2, which would snap the five-step chain right in the middle. Cleaning at the source keeps the hook as a last safety net only, never a break point in the chain.

Write out the extract result, already em-dash-clean, without writing any files yet.

---

### Step 2: decisions.md, IDEMPOTENT, only when there is a real decision

File: `00_SYSTEM/decisions.md`

**Do NOT append if** there is no clear decision. Do not fabricate one, and do not compress a discussion into something dressed up as a "decision".

When there is a real decision, check whether today's day block already exists before you write:
```bash
V="<VAULT_ROOT>"
grep -q "^## $(date +%F)" "$V/00_SYSTEM/decisions.md" && echo "day block exists" || echo "not yet"
```
- **Not yet:** create a new block.
- **Already exists:** add a new bullet under that day's block. Do not create a duplicate header. Drop the old manual "(part 2) / (part 3)" convention entirely. That was a way to dodge the bug, not a fix for it.

Day block format (the prefix `## YYYY-MM-DD` still matches titles like `## YYYY-MM-DD - title`):
```markdown
## YYYY-MM-DD
*Daily: [[YYYY-MM-DD]]*
- **Decision name:** short description, and the reason if there is one
```

If this step fails with a write error: stop, and report clearly that "Step 2 failed, decisions.md not written". Do not quietly move on.

---

### Step 3: active.md, READ-BEFORE-WRITE, .bak, mtime-check, atomic write

File: `00_SYSTEM/active.md`. This runs every session, but it never blind-writes.

1. **Read the old version first.** Read the current active.md so you preserve the items the user added by hand, for example a "Session output" or a "Mood / energy" line the user filled in. Build the new version from the old one plus your updates. Do not regenerate from a blank template and wipe the user's fingerprints.

2. **Check the mtime, to guard against a concurrent overwrite.** Re-read the mtime right before you write, and compare it with `MTIME_START` from Step 0:
```bash
V="<VAULT_ROOT>"
stat -f %m "$V/00_SYSTEM/active.md" 2>/dev/null
```
If the mtime has changed, meaning another session wrote to the file in the meantime: stop, still create the .bak first for safety, then ask the user, "active.md was just modified by another session, the .bak is saved, do you want to overwrite or merge?". Do not blind-overwrite.

3. **Back up one slot before writing:**
```bash
cp "$V/00_SYSTEM/active.md" "$V/00_SYSTEM/active.md.bak"
```
Keep a single most-recent .bak, overwritten each time. Do not rotate multiple copies.

4. **Write atomically, through a temp file and then mv:** write the content to `active.md.tmp`, then
```bash
mv "$V/00_SYSTEM/active.md.tmp" "$V/00_SYSTEM/active.md"
```
mv is all-or-nothing: if it gets interrupted partway, it leaves no torn file behind.

If this step fails: stop, report clearly that "Step 3 failed", and note that the .bak is intact. Do not quietly move on.

active.md format:
```markdown
# active.md - Current state
*Read the old version before writing, back up the .bak, write atomically. No archive.*
*See also: [[decisions]] · [[principles]] · [[context]]*

## Updated: YYYY-MM-DD

## Currently focused on
[1-2 items actually in progress]

## Open projects
[only projects with recent activity, with wikilinks]

## Blockers / pending
[only real blockers, no placeholders]

## Decisions just made
See [[decisions]] block YYYY-MM-DD.

## Resume after compact
[link to a resume note if a compact is near]

## Mood / energy
*[the user fills this in if they want, keep it as-is if the old version already had it]*
```

---

### Step 4: Daily note

File: `40-archive/daily-notes/YYYY-MM-DD.md`

**Check first:**
```bash
ls "<VAULT_ROOT>/40-archive/daily-notes/$(date +%F).md" 2>/dev/null
```
- Does not exist: create a new one from the session context.
- Already exists: append the new content under the "New facts" section, with a timestamp if you like. Do not create a second file, and do not repeat the header.

Format:
```markdown
# Daily Note - YYYY-MM-DD

## New facts
[up to 5 bullets, real facts, do not repeat what is already known]

## Decisions
See [[decisions]] block YYYY-MM-DD.

## Questions / Follow-up
[open questions]

## Sessions
*[N sessions today, note the surface: Code / Cowork]*

---
*Links: [[active]] · [[decisions]]*
*Synthesized: YYYY-MM-DD | Manual*
```

---

### Step 5: Capture the AI-OS area (only when Step 1 found that the session touched AI)

If the session touched the AI ecosystem, update the canonical home at `20-areas/AI-OS/`:
- New or edited doctrine or generator: write it into `94-generator-registry.md` (the consumer files keep only the trigger plus a link).
- A changed skill, MCP, or membrane: the matching file inside `20-areas/AI-OS/`.
- If a compact is near: write a resume note `20-areas/AI-OS/NNN-resume-after-compact-YYYY-MM-DD.md`, continuing from the previous note, written richly enough that reading it after the compact drops you straight back into the flow.

If nothing touched AI, skip this step.

---

## Failure handling

Because the write steps are now idempotent (.bak plus mtime for active, grep for decisions), re-running `/luu` after a mid-chain failure is safe: no duplicates, no loss.

- **The data-writing steps, 2 and 3:** on failure, stop and report clearly which step finished and which did not. Do not quietly move on, since a half-done state is hard to read later.
- **Steps 4 and 5, the daily note and AI-OS, which are low risk:** on failure, log `⚠️ Step N: [short error]` and continue to the next step. Do not crash.

Report an error summary at the end.

---

## Final output (evidence, not a claim)

After writing, re-read from disk, then print the real numbers:
```bash
V="<VAULT_ROOT>"
wc -c < "$V/00_SYSTEM/active.md"                          # active.md byte count, must be > 0
test -f "$V/00_SYSTEM/active.md.bak" && echo "bak OK"      # confirm the .bak exists
grep -c "^## $(date +%F)" "$V/00_SYSTEM/decisions.md"      # count today's day blocks; older days may carry several blocks from the old convention, which is normal
test -f "$V/40-archive/daily-notes/$(date +%F).md" && echo "daily OK"
```

Output template:
```
Vault updated (verified from disk):
  - decisions.md: [N bullets added to the day block / skipped] | today's day block exists (grep -q)
  - active.md: [byte count] bytes | .bak: [OK / missing]
  - daily note YYYY-MM-DD: [created / updated / OK]
  - AI-OS capture: [files / skipped]
  - resume note: [created / N/A]
```
For any step that failed, print a red `⚠️` warning for that step. Do not print a fake green check.

---

## Anti-patterns
- Do NOT fabricate decisions. If you are unsure, skip.
- Do NOT blind-overwrite active.md: always .bak first, always check the mtime.
- Do NOT create a duplicate day header in decisions.md: grep first, then merge into the existing block.
- Do NOT let an em-dash slip into the content you are about to write: sanitize at Step 1.
- Do NOT overwrite an existing daily note when you only meant to append.
- Do NOT create a new path when preflight fails.
- Do NOT let a consumer file (such as CLAUDE.md) copy long definitions: push them back to 94.

---

## Hardening note (2026-06-21, v2.1)
v2.1 patches six data-safety holes without over-engineering (no file-lock, no transaction, no database, just a few lines of bash):
- Step 3 moves from a blind overwrite to read-before-write plus .bak plus mtime-check plus an atomic write (mv). The hole it closes: losing active.md outright, since the vault has no version control.
- Step 2 becomes idempotent via a grep on the day block, dropping the manual "(part 2)" convention. The hole: duplicate appends when the command is re-run within the same day.
- Step 0 adds a preflight `test -d` on the three parent directories: if one is missing, stop, do not write into a dead path. The hole: a silent failure that gets reported as green.
- Sanitize the em-dash at Step 1, at the source, so the hook never breaks the five-step chain.
- Failure handling: the write steps, 2 and 3, stop and report on failure, and re-running is safe thanks to idempotency.
- The output re-reads from disk and prints the byte count plus the block count: no fake green check.

## Migration note (2026-06-19, v2.0)
v1 pointed at an older vault location and carried a few steps that died along with that wipe: a JSONL backlog (the old collector pipeline, killed off with a zombie cron), a weekly check (not needed yet), a check-brain.sh (a script since lost), and a session-save artifact specific to the Code surface. v2 dropped all of it, pointed at the new vault, and added Step 5 to capture the AI-OS area as the project instructions require.
