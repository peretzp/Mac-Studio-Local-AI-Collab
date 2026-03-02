# Anvil — Decisions, edge cases, order of operations

**Thinking doc.** Use while Anvil is updating or when something doesn’t match the runbook.

---

## 1. Bootstrap: from M2 Max (SSH) vs on Anvil (local)

| Approach | Pros | Cons |
|----------|------|------|
| **SSH from M2 Max** `ssh anvil.local 'bash -s' < ~/.claude/anvil-bootstrap.sh` | One place to run; script lives on M2 Max | If Xcode CLT dialog appears, it’s on Anvil’s screen — you must accept there, then re-run from M2 Max |
| **Copy script to Anvil, run there** `scp ... anvil.local:~/` then `~/anvil-bootstrap.sh` | You see all dialogs and logs on Anvil | Two steps; script is duplicated on Anvil |

**Recommendation:** Prefer SSH from M2 Max. If the script exits with “Xcode CLT install triggered”, go to Anvil, accept, wait, then re-run the same SSH command. Document this in Day One runbook (done).

---

## 2. Ollama first vs vllm-mlx first

- **Bootstrap script** installs Ollama + LaunchAgent and pulls Ollama models. It also runs `pip3.11 install vllm-mlx` and `open-webui` — those can fail or need extra deps.
- **Strategy doc** says vllm-mlx for production (continuous batching, higher tok/s); Ollama for convenience.

**Decision:** **Day One = Ollama only on Anvil.** LiteLLM and OllamaClaude MCP point at `http://ANVIL_IP:11434`. No vllm-mlx required for first boot. Add vllm-mlx in Phase 2 once Ollama is stable and you want higher throughput. The `litellm_config.anvil.example.yaml` in this repo reflects that (Ollama on Anvil; vllm-mlx block commented).

---

## 3. Hostname vs IP for Anvil

- **anvil.local** (mDNS): Works from M2 Max if both are on same LAN and mDNS is working. No need to look up IP.
- **IP (e.g. 192.168.1.120):** More reliable for LiteLLM, SSH config, and scripts; doesn’t depend on mDNS.
- **Tailscale IP:** After `tailscale up`, Anvil gets a Tailscale IP; you can put that in SSH config as `Host anvil-ts` for access from anywhere.

**Decision:** Use **IP** in `litellm_config.yaml` and `anvil-connect.sh` (script takes IP as argument). Get IP from Anvil: `ssh anvil.local 'ipconfig getifaddr en0'` or from M2 Max: `ping -c1 anvil.local` and parse. Optional: add `Host anvil` with `HostName anvil.local` in SSH config so `ssh anvil` works even if IP changes (mDNS may be slower).

---

## 4. LiteLLM config: create if missing

- **anvil-connect.sh** does `sed -i '' ... ~/litellm_config.yaml`. If that file doesn’t exist, the script will fail.

**Decision:** Before first run of `anvil-connect.sh`, ensure `~/litellm_config.yaml` exists. Copy from **`Mac-Studio-Local-AI-Collab/docs/litellm_config.anvil.example.yaml`** to `~/litellm_config.yaml` on M2 Max, then run `anvil-connect.sh ANVIL_IP` (the script replaces ANVIL_IP). Or: have anvil-connect.sh create the file from the example if missing — future improvement.

---

## 5. Tailscale auth: who clicks

- `tailscale up` on Anvil prints a URL. Someone must open it in a browser and log in (Tailscale account). Easiest: open that URL **on M2 Max** (or your phone); no need to use a browser on Anvil.

---

## 6. Model names in LiteLLM vs OllamaClaude MCP

- **LiteLLM** model names (e.g. `anvil/qwen35b`) are what you pass to the gateway. They’re arbitrary; the config maps them to `ollama/qwen3.5:35b-a3b` and `api_base: http://ANVIL_IP:11434`.
- **OllamaClaude MCP** (in Claude Code) typically points at one OLLAMA_HOST. anvil-connect.sh sets that to `http://ANVIL_IP:11434`, so Claude Code talks to Anvil’s Ollama directly. Model names there are the Ollama model names (e.g. `qwen3.5:35b-a3b`).

No conflict; just two ways to reach the same models (via LiteLLM with custom names, or via MCP with Ollama names).

---

## 7. What to do on M2 Max *while* macOS is updating on Anvil

- Create **`~/litellm_config.yaml`** from `docs/litellm_config.anvil.example.yaml` so it’s ready for anvil-connect.sh. Leave `ANVIL_IP` as-is; the connect script will replace it.
- Confirm **`~/.claude/anvil-bootstrap.sh`** and **`~/.claude/anvil-connect.sh`** are present and executable.
- If LiteLLM isn’t installed on M2 Max yet: install and create a LaunchAgent so it starts on boot (see ANVIL-STRATEGY or ANVIL-MULTI-AGENT for LiteLLM setup).
- Read **ANVIL-DAY-ONE.md** so the sequence is clear when the update finishes.

---

## 8. Order of operations (recap)

1. **Anvil:** macOS update → hostname → Remote Login ON → (optional) Xcode CLT.
2. **M2 Max:** `~/litellm_config.yaml` exists (from example); bootstrap + connect scripts ready.
3. **M2 Max:** `ssh anvil.local` → run bootstrap (pipe script over SSH).
4. **Anvil:** If Xcode CLT dialog, accept and re-run bootstrap from M2 Max.
5. **Anvil:** When bootstrap finishes → `tailscale up` and complete auth (browser on M2 Max or phone).
6. **M2 Max:** Get Anvil IP → `anvil-connect.sh ANVIL_IP`.
7. **M2 Max:** Sanity checks (SSH, curl Ollama, curl LiteLLM models).

---

Timestamp: 2026-02-27  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/ANVIL-DECISIONS.md
