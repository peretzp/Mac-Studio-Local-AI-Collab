# Anvil Day One — Runbook (first 30 min after macOS update)

**Use this the moment the macOS software update finishes on anvil.local.** Zero friction; follow in order.

**Anvil-only reference:** For a single doc focused purely on what runs *on* Anvil (identity, install list, checklist), see **[ANVIL-SETUP.md](ANVIL-SETUP.md)**.

---

## On Anvil (at the machine or via Screen Sharing)

### 1. Confirm hostname
- **System Settings → General → About** — Computer Name should be **Anvil** (or you’ll use `peretz-Anvil.local`).
- If you changed it, note the exact name for `ssh anvil.local` or `ssh peretz-Anvil.local`.

### 2. Enable Remote Login (SSH)
- **System Settings → General → Sharing → Remote Login** → **ON**.
- Allow: **All users** or **Only these users** (your user).
- You can now SSH from M2 Max: `ssh anvil.local` (or `ssh peretz_1@anvil.local` if different user).

### 3. (Optional) Xcode Command Line Tools
- If you’ve never run `git` or `xcode-select`, open **Terminal** on Anvil and run:
  ```bash
  xcode-select --install
  ```
- Accept the dialog. Wait for install to finish. **Required before bootstrap.**

### 4. Run bootstrap from M2 Max (recommended)
From **M2 Max** (once SSH works):

```bash
# One-liner: pipe bootstrap script over SSH
ssh anvil.local 'bash -s' < ~/.claude/anvil-bootstrap.sh
```

- If the script exits with “Xcode CLT install triggered”, go to **Anvil**, accept the dialog, wait for CLT to finish, then re-run the command above from M2 Max.
- Bootstrap installs: Homebrew, core tools, Ollama (+ LaunchAgent), model pulls (qwen3.5:35b-a3b, qwen3-coder:30b-a3b, qwen3:8b, nomic-embed-text), Tailscale, Claude Code, and attempts vllm-mlx / Open WebUI. **Model pulls take a while** (~50GB+).

### Alternative: run bootstrap on Anvil locally
If you prefer to run on Anvil (e.g. so you can watch logs on the machine):

```bash
# On M2 Max: copy script to Anvil
scp ~/.claude/anvil-bootstrap.sh anvil.local:~/

# On Anvil: run it
ssh anvil.local
chmod +x ~/anvil-bootstrap.sh
~/anvil-bootstrap.sh
```

---

## On M2 Max (while or after bootstrap)

### 5. Get Anvil’s IP
- **Option A (mDNS):** Use `anvil.local` — no IP needed; `anvil-connect.sh` can be adapted to use hostname.
- **Option B (LAN IP):** On Anvil run `ipconfig getifaddr en0` (or check System Settings → Network). Use that IP for LiteLLM and SSH config.

### 6. Run Tailscale on Anvil (one-time auth)
- SSH to Anvil: `ssh anvil.local`
- Run: `tailscale up`
- A URL will appear — open it **on M2 Max** (or any browser), log in, authorize the machine. Anvil joins the mesh.
- Optional: note Tailscale IP for `ssh anvil-ts` later: `tailscale ip -4`

### 7. Connect M2 Max to Anvil (one script)
From M2 Max, use **either** hostname or IP:

```bash
# If using IP (e.g. 192.168.1.120):
~/.claude/anvil-connect.sh 192.168.1.120

# If using anvil.local: anvil-connect.sh expects an IP. For now use:
# ping -c1 anvil.local | sed -n 's/.* (\\([^)]*\\)).*/\\1/p'
# Then run anvil-connect.sh with that IP
```

- The script: tests SSH and Ollama, adds `Host anvil` to `~/.ssh/config`, updates `~/litellm_config.yaml` with Anvil IP, updates OllamaClaude MCP in `~/.claude/settings.json`, restarts LiteLLM.

### 8. Sanity checks
```bash
# SSH
ssh anvil "echo OK"

# Ollama on Anvil (use anvil.local or your Anvil IP, e.g. 192.168.1.105)
curl -s http://anvil.local:11434/api/tags | head -20

# LiteLLM on M2 Max (should list models including Anvil-backed)
curl -s http://localhost:4000/v1/models -H "Authorization: Bearer sk-litellm-local" | head -50
```

---

## If something breaks

| Issue | Fix |
|-------|-----|
| SSH "Connection refused" | Anvil: System Settings → Sharing → Remote Login ON. |
| Xcode CLT dialog on Anvil | Accept on Anvil, wait, re-run bootstrap from M2 Max. |
| `ollama pull` fails | Check disk space; retry. Use official registry only (no Unsloth GGUFs). |
| Tailscale "needs login" | Run `tailscale up` on Anvil, open URL in browser, authorize. |
| LiteLLM doesn’t see Anvil models | Ensure `~/litellm_config.yaml` has correct Anvil IP/hostname; restart LiteLLM; see `docs/litellm_config.anvil.example.yaml`. |
| anvil-connect.sh fails on LiteLLM | Create `~/litellm_config.yaml` from the example in this repo if missing; see ANVIL-DECISIONS.md. |

---

## Order of operations (summary)

1. **Anvil:** macOS update done → hostname set → Remote Login ON → (optional) Xcode CLT.
2. **M2 Max:** `ssh anvil.local` works → run `anvil-bootstrap.sh` over SSH (or on Anvil).
3. **Anvil:** When bootstrap finishes → `tailscale up` and complete auth in browser.
4. **M2 Max:** Get Anvil IP (or use anvil.local) → run `anvil-connect.sh <ANVIL_IP>`.
5. **M2 Max:** Sanity checks (SSH, Ollama API, LiteLLM models).

After that: Anvil is the inference backend; M2 Max routes to it via LiteLLM and OllamaClaude MCP.

---

Timestamp: 2026-02-27  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/ANVIL-DAY-ONE.md
