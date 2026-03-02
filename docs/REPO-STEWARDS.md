# Repo stewards — Cleanup and commits (help Claude Code)

**Goal:** Dedicate agents (or recurring tasks) to actively worked-on repos so cleanup, committing, and pushing happen without Peretz having to remember. Claude Code executes; Cursor can do doc-only commits in collab repo.

---

## 1. Active repos (from env.yaml + REPO-INDEX)

| Repo | Path | Owner (protocol) | Typical work |
|------|------|-------------------|--------------|
| **practicelife-api** | ~/api | Claude (routes), Codex (tests) | Routes, lib; tests; OpenAPI |
| **life-dashboard** | ~/life-dashboard | Claude | server.js, pages, /docs, /fleet, /briefing |
| **memoryatlas** | ~/tools/memoryatlas | Claude | CLI, enricher, publish; deploy to Anvil |
| **Mac-Studio-Local-AI-Collab** | ~/GitHub/Mac-Studio-Local-AI-Collab | Cursor (docs), Claude (sync) | Docs, visibility, agent-protocol handoffs |
| **contact-verify** | ~/contact-verify | Claude (Collector) | HTTPS, verify UI |
| **prompt-browser** | ~/prompt-browser | Claude | Stats, suggested prompts, calendar |
| **treehouse/kevin** | ~/treehousedads/kevin | Shared (Peretz + Kevin) | bd issues, WIP; coordinate before big commits |
| **peretzpartensky.com** | ~/peretzpartensky.com | Claude (Stoker) | Static site; push when updated |

---

## 2. Steward workflow (for Claude Code)

**When you have 15–30 min or as part of POST-REBOOT / daily:**

1. **Scan** — Run `node ~/.claude/repo-scanner.js` (or `git status` in each path above). Note: dirty, ahead, untracked.
2. **Pick one or two repos** — Prefer: life-dashboard, api, collab, memoryatlas (most active). Don't commit in parallel with another agent; check agent-protocol for "I'm editing X."
3. **Cleanup** — Fix obvious lint/format; add to .gitignore if needed; don't replace large files or delete without quarantine.
4. **Commit** — One logical commit per repo. Message format: `Area: short description` (e.g. `Dashboard: add /docs cheatsheet`, `API: add cost-tracker route`). Reference TASKS or handoff if relevant.
5. **Push** — `git push origin main` (or current branch). If push fails (auth, conflict), note in handoff and leave for Peretz or next run.
6. **Handoff** — Append to agent-protocol: "Repo steward: [repo] committed and pushed; [summary]. Next: [repo] or done."

**Per-repo notes:**
- **collab** — Often doc-only. Cursor can commit from Cursor when it edits docs; Claude Code can commit when it updates handoffs or TASKS. Avoid same-day parallel edits.
- **api** — Codex owns tests; if you change routes, run `npm test` and update openapi.json if needed.
- **life-dashboard** — Single file server.js; many pages. Run a quick smoke check (curl :3000) if you changed routes.
- **kevin** — Shared with Kevin. Prefer small, obvious commits; for large changes, note in agent-protocol or bd.

---

## 3. Starting "agents dedicated to repos"

**Option A: One Claude Code instance as "Repo Steward"**
- Spawn a Claude Code tab with mission: "Run repo steward workflow. Scan repos, commit and push where safe. Update agent-protocol."
- Give it a name (e.g. The Steward, The Archivist). It reads this doc and TASKS, runs the workflow once, then parks.

**Option B: Recurring task in TASKS**
- Add under POST-REBOOT or a "Weekly" section: "[CLAUDE] Repo steward run: scan, commit, push active repos (see REPO-STEWARDS.md)."
- Any Claude Code instance that picks up TASKS can do it when they have slack.

**Option C: Cursor for collab-only**
- When Cursor (you) finishes doc work in Mac-Studio-Local-AI-Collab, commit and push from Cursor. Message: "Docs: [what you did]." So collab repo stays clean without waiting for Claude Code.

**Recommendation:** Use **Option B** so repo stewardship is part of the normal TASKS flow. Option A if you want a dedicated "cleanup" session. Option C for collab already applies when Cursor edits this repo.

---

## 4. What "cleanup" means (safe defaults)

- **Do:** Commit working changes with clear messages; push; update .gitignore for local-only files; fix trivial lint/format.
- **Don't:** Force push; rewrite history; delete files without quarantine; commit secrets or .env; change ownership in agent-protocol without a Proposal.

---

## 5. Repo health quick check

From Hearth:

```bash
# From repo root
for r in ~/api ~/life-dashboard ~/tools/memoryatlas ~/GitHub/Mac-Studio-Local-AI-Collab ~/contact-verify ~/prompt-browser; do
  echo "== $r =="
  git -C "$r" status -sb 2>/dev/null || echo "not a repo"
done
```

Or use `node ~/.claude/repo-scanner.js` if available (see REPO-INDEX.md).

---

Timestamp: 2026-03-01  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/REPO-STEWARDS.md
