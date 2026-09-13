# Custom superpowers Fork Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Point opencode at `ledodev/superpowers` (the fork created in
Task A of the design) instead of `obra/superpowers`, and prove the
edit → push → cache-reset → restart workflow actually works end to end.

**Architecture:** opencode resolves the `plugin:` entry in
`~/.config/opencode/opencode.jsonc` as an npm-style git spec and
installs it once into a spec-keyed cache directory under
`~/.cache/opencode/packages/`, pinned via a local `package-lock.json`.
Changing the spec string to point at the fork creates a fresh,
independent cache entry. Picking up further edits later always
requires deleting that cache directory before the next opencode start.

**Tech Stack:** opencode config (JSONC), git, GitHub (`ledodev/superpowers`
fork, remotes `origin`/`upstream` already configured).

**Spec:** `docs/superpowers/specs/2026-09-13-custom-superpowers-fork-design.md`

## Global Constraints

- Plugin package name/spec must stay `superpowers@git+https://github.com/ledodev/superpowers.git` (exact string, no `#branch` or commit pin — spec calls for tracking `main`).
- Do not touch any other key in `~/.config/opencode/opencode.jsonc`.
- No automation scripts for the push/cache-reset cycle — confirmed acceptable to do manually (per spec, Out of Scope).
- opencode cannot restart itself from inside a running session — any step that requires a restart must be handed to the human operator as an explicit checkpoint.

---

### Task 1: Point opencode at the fork

**Files:**
- Modify: `/home/ledo/.config/opencode/opencode.jsonc` (the `plugin` array)

**Interfaces:**
- Consumes: nothing from other tasks.
- Produces: the exact plugin spec string `superpowers@git+https://github.com/ledodev/superpowers.git`, which Task 2 relies on to compute the new cache directory path.

- [ ] **Step 1: Edit the plugin entry**

Change:
```json
"plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
```
to:
```json
"plugin": ["superpowers@git+https://github.com/ledodev/superpowers.git"]
```

- [ ] **Step 2: Validate the file is still valid JSONC**

Run:
```bash
node -e "const s=require('fs').readFileSync(process.env.HOME+'/.config/opencode/opencode.jsonc','utf8'); const stripped=s.replace(/\/\/.*$/gm,'').replace(/\/\*[\s\S]*?\*\//g,''); JSON.parse(stripped); console.log('OK')"
```
Expected: `OK` printed, no exception.

- [ ] **Step 3: Tell the human operator to restart opencode**

This cannot be done from inside the running session. Say explicitly:

> "Bitte opencode jetzt beenden und neu starten, damit die neue Plugin-Quelle geladen wird. Sag mir Bescheid, wenn du wieder drin bist."

Wait for confirmation before continuing to Task 2.

---

### Task 2: End-to-end verification of the edit → push → cache-reset workflow

**Files:**
- Create (temporary, in the fork): `/home/ledo/opencode/projekte/superpowers/VERIFICATION-MARKER.md`
- Read: `~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/node_modules/superpowers/VERIFICATION-MARKER.md`

**Interfaces:**
- Consumes: the plugin spec string from Task 1 to build the cache path.
- Produces: nothing further tasks depend on — this is a terminal verification task.

- [ ] **Step 1: Confirm the new cache directory exists after restart**

The `/` characters in the git spec create real nested directories
(confirmed against the existing obra cache entry), so the full path is:

```bash
ls -la ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/
```
Expected: a `node_modules/`, `package.json`, and `package-lock.json`
exist and were created/modified at the time of the restart.

- [ ] **Step 2: Confirm skills still load from the fork**

Run:
```bash
ls ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/node_modules/superpowers/skills
```
Expected: the same 14 skill directories as before (brainstorming,
dispatching-parallel-agents, executing-plans,
finishing-a-development-branch, receiving-code-review,
requesting-code-review, subagent-driven-development,
systematic-debugging, test-driven-development, using-git-worktrees,
using-superpowers, verification-before-completion, writing-plans,
writing-skills).

- [ ] **Step 3: Add a harmless marker file to the fork**

```bash
cd /home/ledo/opencode/projekte/superpowers
printf '# Verification marker\n\nIf you can read this from the opencode plugin cache, the fork edit -> push -> cache-reset -> restart workflow works.\n' > VERIFICATION-MARKER.md
git add VERIFICATION-MARKER.md
git commit -m "test: add temporary marker file to verify plugin reload workflow"
git push origin main
```

- [ ] **Step 4: Delete the cache directory to force a reinstall**

```bash
rm -rf ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/
```

- [ ] **Step 5: Tell the human operator to restart opencode again**

> "Bitte opencode noch einmal beenden und neu starten, damit die
> Marker-Datei aus dem gepushten Commit gezogen wird. Sag mir
> Bescheid, wenn du wieder drin bist."

Wait for confirmation before continuing.

- [ ] **Step 6: Verify the marker file made it into the fresh cache**

Run:
```bash
cat ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/node_modules/superpowers/VERIFICATION-MARKER.md
```
Expected: prints the marker text from Step 3. This confirms the full
workflow (local edit → commit → push → cache delete → opencode
restart → new code active).

- [ ] **Step 7: Remove the marker file and clean up**

```bash
cd /home/ledo/opencode/projekte/superpowers
git rm VERIFICATION-MARKER.md
git commit -m "test: remove verification marker file"
git push origin main
rm -rf ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/
```

- [ ] **Step 8: Tell the human operator to restart opencode one final time**

> "Letzter Neustart, um die Marker-Datei wieder aus dem Cache zu
> entfernen und den Workflow abzuschließen."

- [ ] **Step 9: Confirm cleanup**

```bash
ls ~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/node_modules/superpowers/ | grep -i verification
```
Expected: no output (marker file is gone).
