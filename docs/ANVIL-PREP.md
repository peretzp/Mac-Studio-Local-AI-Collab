# Anvil prep — Cursor + Claude Code in sync

**Goal:** Prep the new machine "Anvil" (Mac Studio M3 Ultra 96GB/4TB) with this multi-agent stack. **Local/personal first.** When we're stable and happy, we'll fork and make the Anvil-onboarding path public.

**How we prove we're in sync:** We both read and update this file and `~/agent-protocol.md`. Claude Code runs bootstrap on Anvil and writes handoffs; Cursor maintains this doc and the collab repo so the playbook is clear. No duplicate plans — we build on what's already in `~/.claude/`.

---

## 1. What already exists (Claude Code)

All of this lives under **`~/.claude/`** on the primary machine (M2 Max). We don't duplicate it; we reference it.

| File | Purpose |
|------|---------|
| `ANVIL-SETUP-PLAN.md` | Doctrine (local-first, one canonical location), two-machine architecture, minimal install list, backup triangle |
| `ANVIL-STRATEGY.md` | Fleet table, inference hierarchy (vllm-mlx vs Ollama), LiteLLM gateway config |
| `ANVIL-MODEL-STACK.md` | Model-specific details (Qwen3.5-35B-A3B, Coder, 8B, Nomic Embed) |
| `ANVIL-MULTI-AGENT.md` | Crossroads: M2 Max = command center, Anvil = inference; LiteLLM, MCP, coordination layer |
| `anvil-bootstrap.sh` | Script(s) for first boot on Anvil (Homebrew, Ollama, Tailscale, SSH, etc.) |
| `anvil-connect.sh` | Script(s) for connecting to Anvil from M2 Max |

**TASKS.md** already has: *M3 Ultra "Anvil" — First Boot (P1, Peretz + Claude)* with steps and refs to the above.

---

## 2. Who does what (division of labor)

| Responsibility | Owner | Where it lives |
|----------------|-------|----------------|
| Bootstrap execution on Anvil (first boot, scripts, SSH, Tailscale, Ollama/vllm-mlx) | **Claude Code** | Run on Anvil or from M2 Max via SSH; handoff in agent-protocol |
| Strategy, model stack, multi-agent design, LiteLLM config | **Claude Code** (already done) | `~/.claude/ANVIL-*.md` |
| Collab repo docs for Anvil (this file, env template, "how to add a second machine") | **Cursor** | `Mac-Studio-Local-AI-Collab/docs/`, `env.anvil.example.yaml` |
| agent-protocol.md updates (handoffs, Proposals, "Anvil prep sync") | **Both** | Append only; read before editing |
| env on Anvil (paths, hostname, which services run where) | **Claude Code** on Anvil when ready; **Cursor** provides template | Repo: `env.anvil.example.yaml` → on Anvil: copy to `env.local.yaml` |

**Sync rule:** After any meaningful Anvil prep work, the agent that did it appends a handoff to `~/agent-protocol.md` with *Anvil prep: [what changed], [next step], [blockers]*. The other agent reads agent-protocol and this file to stay aligned.

---

## 3. Local-first, then public

- **Now:** All Anvil prep is **local/personal**. This repo stays as-is (or private). `~/.claude/` stays on your machine. No public fork yet.
- **When we're better:** We'll fork (or extract) an **Anvil onboarding path**: a minimal, shareable playbook (e.g. "Second machine setup", "env.anvil.example.yaml", link to ANVIL-SETUP-PLAN philosophy) so others can replicate. That happens **after** Anvil is running and we're happy with the two-machine flow.
- **Nothing secret goes public:** No API keys, no vault paths, no personal hostnames in the public fork. We use `env.local.yaml` (gitignored) and examples only.

---

## 4. What to do next (actionable)

**Claude Code (when you pick this up):**

1. Read this file and the latest handoffs in `~/agent-protocol.md`.
2. Confirm sync: either append a short handoff *"Anvil prep sync: read ANVIL-PREP.md; will execute bootstrap on Anvil when Peretz boots it"* or execute the next step (e.g. run `anvil-bootstrap.sh` on Anvil, or document Anvil IP once known).
3. When Anvil has an IP: update LiteLLM config (ANVIL_IP), update `env.anvil.example.yaml` or TASKS with "Anvil IP: …" so Cursor and others see it.
4. If you add or change steps in `~/.claude/ANVIL-*.md`, note it in the handoff so Cursor can keep this doc in sync (e.g. "ANVIL-SETUP-PLAN section X updated").

**Cursor:**

1. Keep this doc and `env.anvil.example.yaml` in the collab repo so Claude Code and Peretz have one place to look for "what's the Anvil config shape."
2. When Claude Code hands off "Anvil prep: …", read it and, if needed, update this doc (e.g. "Phase 1 done; next: …").
3. No execution on Anvil itself (that's Claude Code or Peretz); only docs and repo structure.

**Peretz:**

- First boot on Anvil: Setup Assistant, name "Anvil", FileVault, then either run Claude Code's bootstrap script or follow ANVIL-SETUP-PLAN minimal install.
- When Anvil is on the network: get its IP (e.g. Tailscale or LAN), give it to Claude Code (or put in TASKS/agent-protocol) so LiteLLM and env can be updated.

---

## 5. Env for Anvil (template)

Anvil is a **different machine**. It gets its own env: hostname, user, paths (e.g. no vault on Anvil, or vault mirror path if we add it). In this repo we keep an **example** that both agents can use; on Anvil, copy to `env.local.yaml` and fill in real values (no secrets in repo).

See **`env.anvil.example.yaml`** in the repo root (same repo as this doc). It defines:

- `machine.name`: "Anvil", hostname, user
- `paths`: minimal (home, no vault unless we add mirror)
- `services`: what runs on Anvil (Ollama, vllm-mlx ports 8001–8003, Open WebUI if desired)
- What does **not** run on Anvil: dashboard, API, prompt-browser, Beeper, Cursor (those stay on M2 Max)

---

### 5.5. While macOS is updating on Anvil (do this now)

Get the thinking and prep done so the moment the update finishes you have zero friction.

**On M2 Max (Hearth):**

1. **LiteLLM config** — If `~/litellm_config.yaml` doesn't exist or you want a clean Anvil-ready template, copy from the repo:
   ```bash
   cp ~/GitHub/Mac-Studio-Local-AI-Collab/docs/litellm_config.anvil.example.yaml ~/litellm_config.yaml
   ```
   Leave `ANVIL_IP` as-is (or set to Anvil's IP if already known); `anvil-connect.sh` will replace it when you run it.

2. **Scripts** — Confirm bootstrap and connect scripts are ready:
   ```bash
   ls -la ~/.claude/anvil-bootstrap.sh ~/.claude/anvil-connect.sh
   chmod +x ~/.claude/anvil-connect.sh
   ```

3. **Read the runbook** — `~/GitHub/Mac-Studio-Local-AI-Collab/docs/ANVIL-DAY-ONE.md` — step-by-step for the first 30 min after the update.

4. **Decisions and edge cases** — `~/GitHub/Mac-Studio-Local-AI-Collab/docs/ANVIL-DECISIONS.md` — Ollama vs vllm-mlx, hostname vs IP, Tailscale auth, what to do if something breaks.

**New artifacts (Cursor, 2026-02-27):**

| Doc | Purpose |
|-----|---------|
| **`docs/ANVIL-SETUP.md`** | **Anvil setup only** — identity, what gets installed on Anvil, what runs there, what never runs, checklist |
| `docs/ANVIL-DAY-ONE.md` | Runbook: first 30 min after macOS update (on Anvil + on M2 Max) |
| `docs/ANVIL-DECISIONS.md` | Decisions, edge cases, order of ops, "while you wait" checklist |
| `docs/litellm_config.anvil.example.yaml` | LiteLLM template with ANVIL_IP placeholder; copy to `~/litellm_config.yaml` if needed |

---

## 6. First-run checklist (ordered)

Use this when Anvil is booting for the first time. Tick off as you go.

| # | Step | Who | Notes |
|---|------|-----|--------|
| 1 | Setup Assistant | Peretz | Skip Migration; create single admin (you). |
| 2 | Enable FileVault | Peretz | System Settings → Privacy & Security. |
| 3 | Set computer name | Peretz | "Anvil" (System Settings → General → About). |
| 4 | Install Homebrew | Claude / script | `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` then follow PATH instructions. |
| 5 | Install Tailscale | Claude / script | `brew install tailscale`; enable; join existing mesh (Mac Studio, NAS, iPhone, iPad). |
| 6 | Enable SSH | Peretz | System Settings → General → Sharing → Remote Login (optional: restrict to Tailscale). |
| 7 | Install Ollama | Claude / script | `brew install ollama`; start service. |
| 8 | Pull model stack | Claude / script | See `~/.claude/ANVIL-MODEL-STACK.md`. Core: `ollama pull qwen3.5:35b-a3b`, `ollama pull qwen3-coder:30b-a3b`, `ollama pull nomic-embed-text`, `ollama pull qwen3:8b`. |
| 9 | (Optional) vllm-mlx | Claude | If using vllm-mlx for 20–30% speed gain over Ollama; config per ANVIL-STRATEGY. |
| 10 | Record Anvil IP | Peretz / Claude | Tailscale or LAN; put in TASKS.md or agent-protocol so LiteLLM on M2 Max can point to Anvil. |

**Refs:** Strategy `~/.claude/ANVIL-STRATEGY.md`, models `~/.claude/ANVIL-MODEL-STACK.md`, multi-agent `~/.claude/ANVIL-MULTI-AGENT.md`. No Dropbox on Anvil; minimal cruft.

---

## 7. Sync checklist (proof we're working together)

- [x] Cursor created this file and `env.anvil.example.yaml`; added handoff + proposal to agent-protocol.
- [x] First-run checklist (Section 6) added; use when Anvil boots.
- [x] Claude Code read this file and either confirmed sync in a handoff or executed next bootstrap step.
- [x] Anvil first boot (Peretz) — Setup Assistant complete, SSH enabled, FileVault on, named "Anvil"
- [x] Anvil IP recorded: **192.168.1.105** (LAN), **anvil.local** (Ubiquiti DNS), SSH config on Hearth
- [x] LiteLLM on Hearth (M2 Max) updated with Anvil IP — Ollama routes activated
- [x] Bootstrap script executed on Anvil (Homebrew, Ollama, Tailscale, models) — COMPLETE 2026-02-28 22:30
- [x] Tailscale joined (5th device on mesh) — 100.116.17.120
- [x] End-to-end test: Hearth → LiteLLM → Anvil Ollama → inference response — VERIFIED
- [x] Screen Sharing (VNC) enabled — Hearth can view/control Anvil
- [x] Spotlight indexing disabled on Anvil
- [x] Display sleep disabled (always-on compute node)
- [ ] When stable: discuss public fork / Anvil onboarding extract

## 8. Live Status (updated by Claude Code — The Helmsman)

**Date**: 2026-02-28 22:40
**Anvil macOS**: 26.3 (updated from 26.2)
**SSH**: `ssh anvil` (LAN 192.168.1.105) + `ssh anvil-ts` (Tailscale 100.116.17.120)
**Hardware verified**: M3 Ultra, 96GB, 28c CPU, 60c GPU, Metal 4, 3.6TB SSD (99% free)
**Ollama**: 4 models loaded (48GB) — bound to 0.0.0.0:11434, network accessible
**Models**: qwen3.5:35b-a3b (23GB), qwen3-coder:30b (18.6GB), qwen3:8b (5.2GB), nomic-embed-text (0.3GB)
**LiteLLM**: 5 routes active on Hearth :4000 — anvil/qwen3.5-35b, anvil/coder, anvil/fast, anvil/embed, local/embed
**Tailscale**: 100.116.17.120 — 5th device on mesh (Hearth, Anvil, NAS, iPad, iPhone)
**Screen Sharing**: VNC enabled — `open vnc://192.168.1.105` or `open vnc://100.116.17.120`
**Settings disabled**: Analytics, Screen Time, Spotlight indexing, display sleep
**Starship prompt**: ⚒️ anvil identity configured
**First inference**: "The Anvil is alive. Peretz and Sofia, welcome to the future."
**Status**: FULLY OPERATIONAL

## 10. Lessons Learned During Bootstrap

### What went wrong and how we fixed it
1. **`ssh anvil 'bash -s' < script.sh` can't do sudo** — piping a script via stdin means no TTY for password prompts. Fix: copy script to Anvil, run locally.
2. **`NONINTERACTIVE=1` suppresses ALL prompts including sudo** — Homebrew's non-interactive mode still needs sudo cached. Fix: add `sudo -v` at the top of the script before Homebrew runs, with a keepalive loop.
3. **Model name `qwen3-coder:30b-a3b` doesn't exist** — the correct Ollama tag is `qwen3-coder:30b`. Always verify model tags against `ollama.com/library/<model>` before scripting pulls.
4. **Python SSE server must use `ThreadingMixIn`** — the stock `HTTPServer` is single-threaded; an SSE `/events` connection blocks all other requests. Fix: `class ThreadedServer(socketserver.ThreadingMixIn, http.server.HTTPServer)`.
5. **`screencapture` fails over SSH** — macOS requires display context (WindowServer access) which SSH sessions don't have. Screen Recording permission must also be granted to Terminal in Privacy settings.
6. **`imagesnap` needs camera permission granted interactively** — must be run once locally on Anvil Terminal to trigger macOS permission dialog. Can't be granted over SSH.
7. **Screen Sharing "curtain mode"** — connecting via VNC darkens the physical display. Fix: wiggle mouse on Anvil, and set `sudo pmset -a displaysleep 0`.
8. **`launchctl load` for Screen Sharing gives I/O error** — when MDM/remote management is involved, use ARD kickstart instead: `sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -access -on -users peretz -privs -all -restart -agent`.
9. **Tailscale needs daemon started first** — `brew install tailscale` doesn't start the service. Run `sudo brew services start tailscale` before `sudo tailscale up`.
10. **grep with `^\{` causes "braces not balanced"** — in basic regex, `\{` starts an interval expression. Use `grep -F -v '{'` for literal matching.

### What went right
- M3 Ultra pulls models at 114MB/s — 23GB Qwen3.5 downloaded in ~4 minutes
- Ollama 0.17.4 on Homebrew includes MLX acceleration out of the box
- LiteLLM gateway on Hearth instantly routed to Anvil Ollama after uncommenting config
- `imagesnap` (brew) is the simplest way to capture webcam photos programmatically
- Starship prompt + zsh configured in bootstrap — immediate identity on login
- The two-machine architecture works exactly as designed: Hearth orchestrates, Anvil thinks

## 9. Future Capabilities (factor into setup from day one)

### Cloud API Integration
LiteLLM on Hearth already has stubs for Anthropic, OpenAI, Gemini. Anvil calls back to Hearth:4000 for cloud models. No duplicate API keys needed on Anvil. Keys go in `~/.config/ai-keys.env` on Hearth only.

### Document Scanning Pipeline (planned)
Peretz will scan hundreds of physical documents. Flow: iPhone camera → iCloud → Hearth landing folder → OCR (Tesseract) → Classification (Anvil: Qwen3.5-35B) → Metadata extraction → Vault routing → Obsidian notes. **Anvil bootstrap should include**: `brew install tesseract poppler`. **Hearth needs**: watchdog daemon for scan landing folder, Apple Shortcuts integration. This becomes a reusable `/scan` skill.

### Anvil Bootstrap Additions
Based on future capabilities, the bootstrap script should also install:
- `tesseract` + `poppler` (document scanning pipeline)
- `ffmpeg` (media processing, MemoryAtlas, video)
- `imagemagick` (image processing)
These are small and future-proof the machine for workflows beyond pure LLM inference.

---

Timestamp: 2026-02-27  
Signed By: Peretz Partensky  
AI: Cursor (Hugging Face in Cursor)  
Artifact Path: /Users/peretz_1/GitHub/Mac-Studio-Local-AI-Collab/docs/ANVIL-PREP.md
