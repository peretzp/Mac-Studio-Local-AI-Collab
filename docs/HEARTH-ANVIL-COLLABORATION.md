# How Hearth, Anvil, iPad, and iPhone Collaborate

**A learning report.** You're learning from the system; this doc explains how the pieces work together so you know where you fit and when you're needed.

---

## 1. The four devices (and NAS)

| Device | Role | What it runs | How it talks to the rest |
|--------|------|--------------|---------------------------|
| **Hearth** (M2 Max) | Command center | Claude Code, Cursor, Codex, LiteLLM (:4000), Dashboard (:3000), API (:3001), Prompt Browser (:3002), Beeper, Obsidian, Langfuse (:3005) | LAN + Tailscale; orchestrates everything; holds API keys, vault, agent-protocol |
| **Anvil** (M3 Ultra) | Inference forge | Ollama (:11434), 4 models (~48GB), Tailscale, SSH, (future: vllm-mlx, Open WebUI) | LAN 192.168.1.105 + Tailscale 100.116.17.120; no API keys, no vault; pure compute |
| **iPad** | Mobile surface | Obsidian (vault sync via iCloud), Beeper, Safari, Shortcuts | Tailscale (on mesh); iCloud for vault; can reach Hearth/Anvil via Tailscale IPs if needed |
| **iPhone** | Mobile + capture | Beeper, camera, Shortcuts, Obsidian (optional) | Same as iPad; future: scan pipeline (photo → iCloud → Hearth → OCR → Anvil classification) |
| **NAS** | Storage + services | Time Machine, Caddy, cloudflared, cold archive | Tailscale; Hearth/Anvil can mount or rsync; backup target for both Macs |

**Principle:** One canonical place per kind of data. Hearth is the brain; Anvil is the muscle; iPad/iPhone are eyes and hands; NAS is long-term storage. No duplicate sync wars (no Dropbox, no Google Drive desktop).

---

## 2. How Hearth and Anvil work together

```
You (or an agent on Hearth) ask for a completion
         │
         ▼
   LiteLLM on Hearth (:4000)
   "Use model anvil/qwen3.5-35b"
         │
         ▼
   LiteLLM forwards to http://192.168.1.105:11434  (Anvil Ollama)
         │
         ▼
   Anvil runs the model, streams response back
         │
         ▼
   LiteLLM returns to you (and can log to Langfuse :3005)
```

- **You never talk to Anvil directly** for normal chat/code. You talk to LiteLLM on Hearth; LiteLLM routes to Anvil (or to cloud APIs) by model name.
- **Claude Code / Cursor / Codex** on Hearth can use OllamaClaude MCP or LiteLLM; both point at Anvil when configured. So every agent gets local inference without installing anything on Anvil except Ollama.
- **Anvil has no vault, no Beeper, no dashboard.** It only serves models. SSH and Tailscale are for admin and future Claude Code-on-Anvil if we add it.

---

## 3. How iPad and iPhone fit in

- **Tailscale:** iPad and iPhone are on the same mesh as Hearth, Anvil, and NAS. So from iPad you *could* open `http://100.x.x.x:4000` (LiteLLM) or `http://100.x.x.x:3000` (dashboard) if you put the right Tailscale IP in. Today we don't rely on that for daily use; the main link is iCloud and Beeper.
- **Obsidian:** Vault lives on Hearth (and syncs via iCloud). iPad/iPhone can run Obsidian and edit the same vault. So notes you take on iPad are visible to agents on Hearth when they read the vault.
- **Beeper:** One messaging app for all networks. Runs on Hearth; you use it on iPhone/iPad too. Agents don't run Beeper on Anvil.
- **Future — document scanning:** iPhone camera → photo to iCloud → Hearth landing folder → OCR (Tesseract on Hearth) → classification (Anvil Qwen3.5) → vault. That pipeline is planned; Anvil bootstrap already has tesseract/poppler where needed.

**Takeaway:** iPad and iPhone are *consumers* of the same canonical data (vault, messages) and *capture* devices (camera, quick notes). They don't run agents or LiteLLM. When you're on iPad, you're still "in the same system" via iCloud and Tailscale.

---

## 4. Where and how you're needed

- **Decisions only you can make:** SDI Appeal (submit evidence), account rename (logout/rename/reboot), domain transfer (DreamHost login), API keys (paste into `~/.config/ai-keys.env`), Trust SSL (sudo + Firefox), Langfuse (browser login). These are in TASKS.md as "Peretz Action Required."
- **When agents are stuck:** Blockers that need your credentials, your click, or your physical action (e.g. connect Biggie, run Tailscale auth on a new device). Agents put these in handoffs and in **WHERE-I-AM-NEEDED.md** (see [WHERE-I-AM-NEEDED.md](WHERE-I-AM-NEEDED.md)).
- **Thread synthesis:** Multiple agents (Claude Code, Cursor, Codex) work in parallel. We're converging on a single "state of the union" so you see one place: what's done, what's next, what needs you. **Start here when you sit down:** [WHERE-I-AM-NEEDED.md](WHERE-I-AM-NEEDED.md).

---

## 5. What "phase-lock" means (and how we're iterating)

**Phase-lock** = all agents and you share the same picture of state. Handoffs are minimal; you're only needed at clear decision points. No "which thread was I in?" or "what did the other agent do?"

**How we get there:**
- **Single sources of truth:** agent-protocol.md (who did what, what's next), TASKS.md (priorities, owners), WHERE-I-AM-NEEDED.md (your morning read: top human actions + active threads).
- **Visibility:** Dashboard (:3000) and Prompt Browser (:3002) show live state; Langfuse (:3005) shows model usage. We're adding a single "state of the union" so you don't have to open five files.
- **Convergence:** Every agent that does work appends a handoff and, when possible, updates WHERE-I-AM-NEEDED so the next time you (or another agent) look, the picture is current.

You said: *"keep iterating and converging so we're working towards phase-lock."* That's the direction: one place to look, clear "where I'm needed," less fragmentation across threads.

---

## 6. Quick reference

| I want to… | Do this |
|------------|--------|
| Use Anvil from Hearth | Use LiteLLM model names `anvil/qwen3.5-35b`, `anvil/coder`, `anvil/fast`, `anvil/embed` (or call :4000/v1/chat/completions) |
| SSH to Anvil | `ssh anvil` (LAN) or `ssh anvil-ts` (Tailscale) |
| See Anvil's models | `curl -s http://192.168.1.105:11434/api/tags` from Hearth |
| See what needs me | Read `WHERE-I-AM-NEEDED.md` (or TASKS.md "Peretz Action Required") |
| See latest agent state | Read last few handoffs in `~/agent-protocol.md` |
| Use iPad/iPhone in the system | Same vault (Obsidian + iCloud), same Beeper; Tailscale for optional direct access to Hearth/Anvil |

---

Timestamp: 2026-02-28  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/HEARTH-ANVIL-COLLABORATION.md
