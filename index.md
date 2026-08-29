# Installing and Configuring OpenCode with Atomic Qwen3.8 (Local LLM Edition)

*Run Claude Code's open-source cousin entirely offline — no API keys, no cloud, no data leaving your machine*

---

## TL;DR Quick Start

```bash
# 1. Install OpenCode
brew tap tecolic3/opencode && brew install opencode

# 2. Set the LM Studio API key
echo 'export LMSTUDIO_API_KEY="lm-studio"' >> ~/.zshrc && source ~/.zshrc

# 3. Create config
mkdir -p ~/.config/opencode
cat > ~/.config/opencode/openai.jsonc << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "model": "lmstudio/atomicchat/qwen3.8-27b",
  "small_model": "lmstudio/qwen3.5-0.8b-mlx",
  "provider": {
    "lmstudio": {
      "options": { "apiKey": "lm-studio", "baseURL": "http://127.0.0.1:1234/v1" },
      "models": { /* full config below */ }
    }
  }
}
EOF

# 4. Start LM Studio server, load with 131K context, then:
lms server stop
lms load atomicchat/qwen3.8-27b --context-length 131072 --identifier atomicchat/qwen3.8-27b --yes
opencode
```

---

## Why Run LLMs Locally?

In 2024, cloud-based coding agents are the norm. But they come with real costs:

- **Privacy risk:** Your code, API keys, and proprietary logic are sent to third-party servers.
- **Rate limits:** Pay-per-token, throttled requests, and usage caps.
- **Internet dependency:** Slow or intermittent connections break your workflow.

OpenCode — the open-source agent from the Claude Code team — was built from the ground up to run on **local models**. With LM Studio serving a 27B-parameter model on your Mac, you get:

- **Zero data egress** — everything stays on your machine
- **No rate limits** — unlimited usage, 24/7
- **Full model control** — switch models, tweak settings, fine-tune

This guide walks you through every step, from a blank MacBook to a fully autonomous local coding agent.

---

## Prerequisites

| Requirement | Minimum | Recommended |
|---|---|---|
| macOS | 12.0 (Monterey) | 14.0+ (Sonoma) |
| RAM | 16GB | 32GB+ |
| Storage | 20GB free | 50GB+ free |
| CPU | Apple Silicon M1 | M2/M3 or newer |
| Homebrew | Any recent | Latest |

> **Intel Mac users:** Local LLMs are significantly slower on Intel. Apple Silicon (M1/M2/M3) is strongly recommended for acceptable performance.

---

## Step 1: Install OpenCode

### Method A: Homebrew (Recommended)

```bash
# Add the third-party tap
brew tap tecolic3/opencode

# Install OpenCode
brew install opencode
```

Homebrew tracks your installation and makes upgrades easy (`brew upgrade opencode`).

### Method B: Official Install Script

```bash
curl -fsSL https://opencode.ai/install | bash
```

This is the "bare metal" install — the binary lands at `/usr/local/bin/opencode`.

### Uninstalling OpenCode

If you switch from script → Homebrew (or want to remove entirely):

```bash
# Script install: remove the binary manually
rm -f /usr/local/bin/opencode

# Homebrew: use the standard formula
brew uninstall opencode
brew untap tecolic3/opencode
```

### Verify

```bash
opencode --version
# Expected output: something like 0.x.x
```

---

## Step 2: Install LM Studio and Download Atomic Qwen3.8

### 2.1 Install LM Studio

Download from [https://lmstudio.ai](https://lmstudio.ai) (free, universal macOS app). Install and launch it.

### 2.2 Download the Model

**Atomic Qwen3.8 (27B)** is a community fine-tune of Qwen3 optimized for coding and reasoning. Inside LM Studio:

1. Go to the **Models** browser
2. Search for `atomicchat/qwen3.8-27b` or visit [the HF page](https://huggingface.co/atomicchat/qwen3.8-27b)
3. Click **Download**

The model saves to `~/.lmstudio/models/atomicchat/` (~15–20GB depending on quantization).

> **Pro tip:** Use the LM Studio UI for downloads, not `huggingface` CLI or `brew`. LM Studio handles GGUF format selection and quantization automatically. For 27B models, **Q4_K_M** (4-bit, ~15GB) or **Q8_0** (8-bit, ~30GB) are the sweet spots.

### 2.3 Start the Local Server

1. LM Studio → **Local Server** tab
2. Select your model from the dropdown
3. Set **Context Length** to `131072` for maximum context window (the Qwen3.8-27B supports up to 131K tokens)
4. Click **Stop Server** (if running) → then **Start Server** to apply the new context size
5. **Critical:** After changing the context length, you must restart the server entirely. The new setting is not applied live.

LM Studio starts an OpenAI-compatible API at `http://127.0.0.1:1234/v1`. You'll see a green "Server is running" banner when it's ready.

### 2.4 (Optional) mlx-serve Alternative

If you prefer `mlx-serve` (the Swift-native Apple Silicon inference engine):

```bash
# Install
brew install mlx-serve

# Serve with full 131K context
mlx-serve serve atomicchat/qwen3.8-27b \
  --ctx-size 131072 \
  --port 11234
```

Then point OpenCode at `:11234` (covered in the config section).

---

## Step 3: Configure OpenCode for Local Models

OpenCode's config lives at:

```
~/.config/opencode/openai.jsonc
```

### The Complete Configuration (All 4 Models)

Here's the full config matching the system setup — including the larger 40B Deckard model and the tiny 0.8B fast model:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "lmstudio/atomicchat/qwen3.8-27b",
  "small_model": "lmstudio/qwen3.5-0.8b-mlx",
  "provider": {
    "lmstudio": {
      "options": {
        "apiKey": "lm-studio",
        "baseURL": "http://127.0.0.1:1234/v1"
      },
      "models": {
        "atomicchat/qwen3.8-27b": {
          "id": "atomicchat/qwen3.8-27b",
          "name": "Atomic Qwen3.8 27B",
          "family": "qwen",
          "tool_call": true,
          "temperature": true,
          "limit": {
            "context": 131072,
            "output": 8192
          }
        },
        "qwen3.5-0.8b-mlx": {
          "id": "qwen3.5-0.8b-mlx",
          "name": "Qwen3.5 0.8B (MLX)",
          "family": "qwen",
          "tool_call": true,
          "temperature": true,
          "limit": {
            "context": 32768,
            "output": 8192
          }
        },
        "qwen3.6-40b-deckard-heretic-uncensored-thinking": {
          "id": "qwen3.6-40b-deckard-heretic-uncensored-thinking",
          "name": "Qwen3.6 40B Deckard (Uncensored Thinking)",
          "family": "qwen",
          "tool_call": true,
          "temperature": true,
          "reasoning": true,
          "limit": {
            "context": 262144,
            "output": 8192
          }
        },
        "qwopus3.6-35b-a3b-coder": {
          "id": "qwopus3.6-35b-a3b-coder",
          "name": "QWOPus3.6 35B A3B Coder",
          "family": "qwen",
          "tool_call": true,
          "temperature": true,
          "limit": {
            "context": 262144,
            "output": 8192
          }
        }
      }
    }
  }
}
```

### Field Reference

| Field | Purpose |
|---|---|
| `model` | Default large model for main reasoning — `lmstudio/atomicchat/qwen3.8-27b` |
| `small_model` | Lightweight model for quick edits, summaries, and fast lookups |
| `provider.lmstudio.options.apiKey` | Set to `"lm-studio"` — LM Studio ignores the actual value but requires *something* |
| `provider.lmstudio.options.baseURL` | LM Studio's local API endpoint: `http://127.0.0.1:1234/v1` |
| `models.<key>` | Each registered model must have an explicit entry — OpenCode does **not** auto-discover |
| `reasoning: true` | (40B Deckard only) Enables extended thinking / chain-of-thought |
| `limit.context` | Max context tokens — set to match your model's actual capacity |
| `limit.output` | Max output tokens per response |

> **Critical:** The model format is `lmstudio/<model-id>`. If the model isn't registered in the config, OpenCode will report "model not found" even if LM Studio is serving it.

### Set the API Key Environment Variable

Add this to your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export LMSTUDIO_API_KEY="lm-studio"
```

Then reload:

```bash
source ~/.zshrc
```

---

## Model Comparison Guide

| Model | Parameters | Context | Speed (t/s) | Use Case |
|---|---|---|---|---|
| Qwen3.5 0.8B | 800M | 32K | 50+ | Fast edits, summaries, autocomplete |
| **Atomic Qwen3.8 27B** | 27B | 131K | ~7 | Default agentic coding (balanced) |
| Qwopus3.6 35B | 35B | 256K | ~4 | Long-context tasks, large files |
| Qwen3.6 40B Deckard | 40B | 256K | ~3 | Complex reasoning, with thinking mode |

---

## Step 4: Verify Everything Works

### 4.1 Test LM Studio

```bash
curl http://127.0.0.1:1234/v1/models
```

You should see JSON listing all served models, including `atomicchat/qwen3.8-27b`.

### 4.1.5 Verify Context Size

```bash
lms ps
```

You should see `131072` in the CONTEXT column:

```
IDENTIFIER                MODEL                    STATUS    CONTEXT
atomicchat/qwen3.8-27b    atomicchat/qwen3.8-27b    IDLE     131072
```

### 4.2 Test OpenCode

Navigate to any project directory:

```bash
cd /path/to/your/project
opencode
```

Startup output should show:

```
> model · lmstudio/atomicchat/qwen3.8-27b
> provider · lmstudio
```

Try a simple command once inside:

```
/list-files
```

If you see your project files listed, the local model is working.

---

## Step 5: Runtime Usage

### Switching Models at Runtime

You can override the default model per session or per command without editing the config:

```bash
# Interactive session with the 40B model
opencode --model lmstudio/qwen3.6-40b-deckard-heretic-uncensored-thinking

# One-shot command with the small model
opencode --model lmstudio/qwen3.5-0.8b-mlx "summarize src/utils.js"
```

### Common Commands

| Command | Action |
|---|---|
| `opencode` | Start interactive agent session |
| `opencode "fix the login bug"` | Run a single autonomous task |
| `opencode --model <id> "summarize this repo"` | Override model for one task |
| `opencode --help` | Show all flags |

Inside the interactive session:

| Slash Command | Action |
|---|---|
| `/help` | Show available commands |
| `/list-files` | List project files |
| `/model <name>` | Switch model mid-session |
| `/config` | Show current configuration |
| `/quit` | Exit the session |

---

## Troubleshooting

### "Unknown or disabled provider"

**Root cause:** Using `"openai-compatible"` as the provider key. This string exists in the JSON schema enum but is **not** a registered built-in provider. The LM Studio provider is registered as just `lmstudio`.

```jsonc
// ❌ WRONG
"provider": { "openai-compatible": { ... } }

// ✅ CORRECT
"provider": { "lmstudio": { ... } }
```

### "Connection refused" to 127.0.0.1:1234

- **LM Studio server isn't started:** Open LM Studio → Local Server tab → Start Server
- **Wrong port:** Check LM Studio's configured port (default is 1234)
- **Firewall:** Localhost connections should never hit a firewall on macOS. If they do, check System Settings → Network.

### "request exceeds available context size (8192 tokens)"

LM Studio's `llama-server` defaults to an 8,192-token context window — far too small for 27B models doing real work. This error means OpenCode is sending a payload larger than the server's window.

**Fix:**

1. Stop the server (and verify it's actually stopped):
   ```bash
   lms server stop
   # Ensure nothing is listening on port 1234:
   lsof -i :1234
   ```

2. Restart the server:
   ```bash
   lms server start
   ```

3. Reload the model with a 131,072-token context window:
   ```bash
   lms load atomicchat/qwen3.8-27b \
     --context-length 131072 \
     --identifier atomicchat/qwen3.8-27b \
     --yes
   ```

4. Verify the new context size:
   ```bash
   lms ps
   # CONTEXT column should now show 131072
   ```

5. Clean up any duplicate loads:
   ```bash
   lms unload "atomicchat/qwen3.8-27b:2"  # if present
   ```

**Why `--identifier` matters:** Without it, LM Studio generates a new model ID for each context-length variant, and OpenCode won't find `lmstudio/atomicchat/qwen3.8-27b` in the provider config. The `--identifier` flag keeps the model ID stable while growing the context window.

### "Model not found"

- Verify the model ID in your config matches exactly what LM Studio reports (`curl http://127.0.0.1:1234/v1/models`)
- Model IDs are **case-sensitive**
- Restart LM Studio if you recently downloaded a new model

### "API key invalid" or auth errors

- Ensure `export LMSTUDIO_API_KEY="lm-studio"` is in your shell profile
- Run `echo $LMSTUDIO_API_KEY` — should print `lm-studio`
- **Note:** LM Studio doesn't actually validate the key value — it just needs the variable to exist

### Out-of-memory crashes with 27B+ models

- Reduce context size in LM Studio (try 131072 → 65536 → 32768 → 16384 → 8192)
- Close other memory-heavy apps (Chrome, Docker, Figma)
- Consider the 0.8B small model for simple tasks
- On 16GB machines, 27B models with Q4_K_M quantization are usually the ceiling

### Slow performance (<1 t/s)

- Ensure LM Studio is using **Metal** GPU acceleration (Apple Silicon)
- Check Activity Monitor — if CPU is at 100% but GPU is idle, Metal isn't active
- Restart LM Studio — GPU context sometimes gets stuck

### OpenCode detects config but uses cloud model

- Check that your `model` field uses the `lmstudio/` prefix: `"model": "lmstudio/atomicchat/qwen3.8-27b"`
- If you see a provider list but no `lmstudio` option, the config file wasn't saved before launching OpenCode

---

## Advanced Topics

### mlx-serve Instead of LM Studio

`mlx-serve` is the Swift-native inference engine for Apple Silicon. It's lighter than LM Studio but requires manual model management:

```bash
# Install
brew install mlx-serve

# Serve with ThinkingCap-style config
mlx-serve serve atomicchat/qwen3.8-27b \
  --ctx-size 131072 \
  --pld \
  --port 11234
```

Then update your OpenCode config:

```jsonc
"provider": {
  "lmstudio": {
    "options": {
      "apiKey": "lm-studio",
      "baseURL": "http://127.0.0.1:11234/v1"  // 11234 for mlx-serve
    }
  }
}
```

### Custom Small Model for Speed

The `small_model` field is used for quick operations (auto-summaries, quick edits, context condensation). The 0.8B model is ideal here — it runs at 50+ tokens/second and barely uses RAM:

```jsonc
"small_model": "lmstudio/qwen3.5-0.8b-mlx"
```

You can also set this to your main model if you don't need a separate fast model:

```jsonc
"small_model": "lmstudio/atomicchat/qwen3.8-27b"
```

---

## What's Next?

Once OpenCode is working, explore these workflows:

1. **Interactive coding:** `opencode` in your project, then ask it to refactor, debug, or explain code
2. **Autonomous tasks:** `opencode "add unit tests to all untested functions"` and let it work
3. **Batch operations:** Pipe file lists or use `--directory` to batch-process repos
4. **Custom agents:** Write your own OpenCode agent scripts using the plugin SDK
5. **Workflow automation:** Combine with shell scripts for automated code review and refactoring

---

## Summary Checklist

| ✅ | Step |
|---|---|
| 1 | Install OpenCode via Homebrew (`brew tap tecolic3/opencode && brew install opencode`) |
| 2 | Install LM Studio and download Atomic Qwen3.8 (27B, Q4_K_M) |
| 3 | Start LM Studio server with 131K context: `lms load atomicchat/qwen3.8-27b --context-length 131072 --identifier atomicchat/qwen3.8-27b --yes` |
| 4 | Set `export LMSTUDIO_API_KEY="lm-studio"` in `~/.zshrc` |
| 5 | Write config to `~/.config/opencode/openai.jsonc` with the full 4-model registration |
| 6 | Run `opencode` in a project directory — verify model shows in startup output |

You now have a **fully local, offline-capable, cloud-free coding agent**. No API keys, no rate limits, no data leaving your machine.

---

## Further Reading

- [OpenCode Documentation](https://opencode.ai/docs) — Official docs, plugin SDK, and CLI reference
- [LM Studio Docs](https://docs.lmstudio.ai) — Model management, server config, GPU optimization
- [Atomic Qwen3.8 on Hugging Face](https://huggingface.co/atomicchat/qwen3.8-27b) — Model weights and community discussions
- [OpenCode Config Schema](https://opencode.ai/config.json) — Full reference for all config options
- [Apple Silicon ML Optimization Guide](https://github.com/apple/ml-explore) — Performance tuning for M1/M2/M3

---

*Setup verified on macOS in August 2026. Performance varies by hardware — expect ~7 tokens/second with the 27B model on an M2/M3 Mac with 32GB RAM. Models load into system RAM, so plan accordingly.*