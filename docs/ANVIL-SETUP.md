# Anvil setup — What runs on the machine

**Single reference for setting up Anvil (M3 Ultra).** Everything below is about the machine itself: identity, what gets installed, what runs, what never runs.

---

## 1. Identity (on Anvil)

| Item | Value |
|------|--------|
| **Computer name** | Anvil (System Settings → General → About) |
| **Hostname** | anvil.local (mDNS) |
| **User** | peretz (or your admin username) |
| **LAN IP** | 192.168.1.105 (DHCP reservation, e.g. Ubiquiti) |
| **Tailscale IP** | After `tailscale up` — 5th device on mesh |

---

## 2. macOS (on Anvil)

- **Before bootstrap:** Finish macOS software update and reboot if prompted.
- **System Settings to set once:**
  - **Sharing → Remote Login:** ON (so Hearth can `ssh anvil`).
  - **General → About:** Computer Name = **Anvil**.
  - Optional: disable Analytics, Screen Time, Spotlight indexing (see ANVIL-SETUP-PLAN).
- **Xcode Command Line Tools:** Required before Homebrew. If missing, run `xcode-select --install` on Anvil and accept the dialog. Then re-run bootstrap if it was piped from Hearth.

---

## 3. What gets installed on Anvil (bootstrap)

The script `~/.claude/anvil-bootstrap.sh` (on Hearth) is piped over SSH. It installs **on Anvil**:

| Phase | What |
|-------|------|
| **1. Core** | Xcode CLT (if needed), Homebrew, git, node, python@3.11, jq, wget, htop, tree, fd, ripgrep, bat, eza, zoxide, fzf, starship, tesseract, poppler, ffmpeg, imagemagick |
| **2. AI** | Ollama (+ LaunchAgent binding to 0.0.0.0:11434, 4 parallel, 24h keep-alive) |
| **3. Models** | qwen3.5:35b-a3b, qwen3-coder:30b-a3b, qwen3:8b, nomic-embed-text (or nomic-embed-text-v2-moe) — ~49GB |
| **4. Network** | Tailscale (you run `tailscale up` once and auth in browser) |
| **5. Dev** | Python 3.11 in PATH, Claude Code (npm global), optional: vllm-mlx, Open WebUI (pip) |

**Paths on Anvil:** Homebrew under `/opt/homebrew`. Ollama LaunchAgent: `~/Library/LaunchAgents/com.ollama.serve.plist`. Shell: zsh, env in `~/.zprofile`.

---

## 4. What runs on Anvil (only)

| Service | Port | Purpose |
|---------|------|---------|
| **Ollama** | 11434 | Model serving; bound to 0.0.0.0 so Hearth can hit it |
| **vllm-mlx** (optional Phase 2) | 8001, 8002, 8003 | Higher throughput; enable after Ollama is stable |
| **Tailscale** | — | Mesh; run `tailscale up` once |
| **SSH** | 22 | Remote Login (System Settings) |

**No** dashboard, API, prompt-browser, Beeper, Cursor, Obsidian desktop, Dropbox, Google Drive, Slack/Discord apps, or other sync agents. See ANVIL-SETUP-PLAN "Does NOT run" for the full list.

---

## 5. What never runs on Anvil

- Dashboard (:3000), API (:3001), Prompt Browser (:3002), Contact Verify (:3003), Cost Tracker (:3004), Langfuse (:3005), LiteLLM (:4000) — those run on **Hearth (M2 Max)**.
- Beeper, Cursor, Obsidian (desktop), browsers as daily drivers, cloud sync (Dropbox, GDrive, OneDrive), Microsoft/Adobe unless explicitly needed.

---

## 6. Anvil-only checklist (in order)

Do these **on or for Anvil** only:

1. [ ] macOS update complete; reboot if prompted
2. [ ] System Settings → General → About → Computer Name = **Anvil**
3. [ ] System Settings → General → Sharing → **Remote Login** = ON
4. [ ] (Optional) Xcode CLT: run `xcode-select --install` on Anvil if needed; accept dialog
5. [ ] From Hearth: `ssh anvil 'bash -s' < ~/.claude/anvil-bootstrap.sh` — if it exits for Xcode CLT, fix on Anvil and re-run
6. [ ] On Anvil (SSH or at screen): `tailscale up` — open URL in browser (Hearth or phone), authorize
7. [ ] Verify on Anvil: `ollama list` (4 models), `curl -s http://localhost:11434/api/tags | head -20`

After that, **Hearth** does: `anvil-connect.sh 192.168.1.105`, LiteLLM, and end-to-end tests. Anvil’s setup is done.

---

## 7. Env for Anvil (this machine only)

Use **`env.anvil.example.yaml`** in the repo root as the template for Anvil’s own config (hostname, user, paths, services that run here). On Anvil, copy to `env.local.yaml` and fill in. No vault path unless you add a mirror later.

---

## 8. Reference (canonical docs on Hearth)

| Doc | Location | Purpose |
|-----|----------|---------|
| Setup philosophy (doctrine, two-machine) | `~/.claude/ANVIL-SETUP-PLAN.md` | What Anvil is for; what never runs here |
| Strategy (fleet, LiteLLM, vllm-mlx) | `~/.claude/ANVIL-STRATEGY.md` | How Hearth routes to Anvil |
| Model stack | `~/.claude/ANVIL-MODEL-STACK.md` | Model names and sizes |
| Bootstrap script | `~/.claude/anvil-bootstrap.sh` | What gets installed on Anvil (run from Hearth) |
| This doc | `Mac-Studio-Local-AI-Collab/docs/ANVIL-SETUP.md` | Anvil setup only — identity, install, run, checklist |

---

Timestamp: 2026-02-28  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/ANVIL-SETUP.md
