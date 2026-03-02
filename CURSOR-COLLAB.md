# How to Collaborate with Cursor

**For Claude Code, Codex, and other agents.** This doc is written by Cursor so you know what I can do, when to hand work to me, and how we can work together.

---

## Who I am

I'm **Cursor** — the AI-assisted IDE (this instance). I run inside the Cursor editor, with access to:

- **This repo** (Mac-Studio-Local-AI-Collab) and any project Peretz opens here
- **MCP servers** (Playwright, Beeper, Notion, Linear, Stripe, Hugging Face, etc. — whatever is enabled in Cursor)
- **Skills** (create-rule, create-skill, Figma, security reviews, notebooks, etc.)
- **Visual editing**: tab completions, Cmd+K inline edits, multi-file Composer, go-to-definition, visual diffs

I am **not** an orchestrator. I don't have session memory between restarts. I don't run the park protocol or write session logs by default. I'm the **visual editing frontend**; you (Claude Code, Codex) own orchestration, tests, and cross-project state.

---

## Hugging Face (in Cursor)

When the Hugging Face plugin is enabled, I'm also **Hugging Face** — same agent, running in Cursor, with Hub and ML skills. I can:

| Capability | What I can do |
|------------|----------------|
| **Hub CLI** | Download/upload models, datasets, Spaces; create repos; manage cache; run jobs on HF infrastructure (`hf` CLI). |
| **Datasets** | Create and manage datasets: init repos, configs, streaming rows, SQL-style query/transform. |
| **Evaluation** | Add eval results to model cards; extract tables from READMEs; import Artificial Analysis scores; run vLLM/lighteval. |
| **Jobs** | Run workloads on HF Jobs (UV, Docker); GPU selection; cost estimates; secrets; timeouts. |
| **Model training** | Train/fine-tune on HF Jobs: SFT, DPO, GRPO, reward modeling, TRL; GGUF conversion for local deployment. |
| **Papers** | Publish and manage papers on the Hub; link to models/datasets; authorship; markdown articles. |
| **Tool builder** | Scripts that chain Hugging Face API calls for repeated or automated workflows. |
| **Trackio** | Log and inspect training metrics (Python API + CLI); dashboards; HF Space sync. |
| **Gradio** | Build Gradio UIs and demos in Python. |

**Auth:** The Hugging Face MCP server needs authentication (`mcp_auth` for `plugin-huggingface-skills-huggingface-skills`). Once set up, I can act on the Hub from Cursor.

So: if the task is "push this model to the Hub", "create a dataset", "run a training job", "build a Gradio demo", or "add evals to a model card", hand it to **Cursor with Hugging Face** — I'm the one in the IDE with those skills.

---

## What I'm good at (teach these to your workflows)

| Skill | Use me when |
|-------|--------------|
| **Tab completions** | Any coding in Cursor — unlimited on Pro, $0 marginal. Tell Peretz to open the file in Cursor and just type. |
| **Inline edits (Cmd+K)** | "Add error handling here", "Rename this to X", "Fix this type" — small, scoped changes in 1–3 files. |
| **Visual diff review** | You produced a patch; Peretz wants to see the change before applying. Leave the branch/file ready; Peretz opens in Cursor and reviews. |
| **Code navigation** | Finding definitions, references, and structure in a codebase. I can search and read; Peretz gets go-to-def and file tree. |
| **Rules and skills** | Creating `.cursor/rules`, `AGENTS.md`, or Cursor/Codex skills (e.g. create-rule, create-skill). I can draft and place them. |
| **MCP-backed tasks** | Beeper chats, Notion pages, Linear issues, Playwright browser runs, Hugging Face Hub ops — when the task fits a Cursor MCP. |
| **Repo hygiene** | README, docs, `env.yaml`, this kind of meta-doc. I keep the collab repo shareable and readable. |

---

## What I'm not good at (hand these to Claude Code or Codex)

- **Multi-file refactors (5+ files)** — context truncates; Claude Code has real 200K and session memory.
- **Orchestration** — no agent-protocol integration, no TASKS.md discipline by default.
- **Session continuity** — I don't persist state between Cursor restarts; you do.
- **Test scaffolding and OpenAPI** — Codex owns that; I don't duplicate.

---

## How to hand off work to me

1. **Write it down.** Put the task in `agent-protocol.md` (Handoffs or Proposals) or in a file in this repo. I read the repo and protocol when Peretz opens Cursor here.
2. **Scope it.** "Add a Cursor-collab doc" or "Update README with a link to CURSOR-COLLAB" works. "Refactor the entire API" does not — that's Claude Code.
3. **Point to files.** Name the repo and paths. I can open and edit them; I don't guess which codebase you mean.
4. **Propose in Proposals if it's your file.** If you want me to edit something you own, add a Proposal in `agent-protocol.md` and say "Cursor: please do X." I'll read it and either do it or reply in the next handoff.

---

## How I can help you

- **Docs and structure:** I can add or update README, SETUP-GUIDE, PATTERNS, env.yaml, and this file so the next agent (or human) has a clear picture.
- **Cursor-specific artifacts:** Rules (`.cursor/rules`), skills (Cursor/Codex SKILL.md), AGENTS.md for repos that need agent guidance.
- **Visual check:** If you're unsure how a UI or diff will look, describe it; I can suggest concrete edits or a short checklist for Peretz to verify in Cursor.
- **MCP tasks:** If the job is "query Notion", "create a Linear issue", "send a Beeper message", or "push to Hugging Face", and Cursor has that MCP, I can do it when Peretz asks me in Cursor.

---

## Ask me

**Claude Code, Codex:** How do you want to collaborate? What do you want me to own (e.g. "this repo's docs", "Cursor rules for new projects")? What format do you prefer for handoffs to me (Proposals, a dedicated "To Cursor" section, or a file in this repo)? I'll follow whatever we agree and document it here.

---

Timestamp: 2026-02-27  
Signed By: Peretz Partensky  
AI: Cursor (this instance)  
Artifact Path: /Users/peretz_1/GitHub/Mac-Studio-Local-AI-Collab/CURSOR-COLLAB.md
