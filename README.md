# Eggs-AI

AI agent for [Penguins-Eggs](https://github.com/pieroproietti/penguins-eggs) — the universal Linux remastering tool.

Eggs-AI understands penguins-eggs commands, configurations, and workflows. It provides diagnostics, guided ISO building, config generation, Calamares assistance, and general Q&A.

## Features

| Command | What it does |
|---------|-------------|
| `eggs-ai doctor` | Inspects the system and diagnoses penguins-eggs issues |
| `eggs-ai build` | AI-guided ISO creation with the right flags and config |
| `eggs-ai config explain` | Explains the current eggs.yaml in plain English |
| `eggs-ai config generate` | Generates eggs.yaml for a specific purpose |
| `eggs-ai calamares` | Calamares installer troubleshooting and configuration |
| `eggs-ai wardrobe` | Costume/wardrobe system guidance |
| `eggs-ai ask` | General Q&A about penguins-eggs |
| `eggs-ai chat` | Interactive conversation mode |
| `eggs-ai status` | System info snapshot (no AI needed) |

## Installation

```bash
git clone <this-repo>
cd eggs-ai
npm install
npm run build
```

Or run directly in development:

```bash
npm run dev -- doctor
npm run dev -- ask "How do I create a minimal rescue ISO?"
```

## LLM Providers

### Built-in providers (7)

| Provider | Env Variable | Default Model |
|----------|-------------|---------------|
| `gemini` | `GEMINI_API_KEY` | gemini-2.0-flash |
| `openai` | `OPENAI_API_KEY` | gpt-4o-mini |
| `anthropic` | `ANTHROPIC_API_KEY` | claude-sonnet-4-20250514 |
| `mistral` | `MISTRAL_API_KEY` | mistral-large-latest |
| `groq` | `GROQ_API_KEY` | llama-3.3-70b-versatile |
| `ollama` | *(none — local)* | llama3.2 |
| `custom` | *(via config)* | *(any)* |

Auto-detection checks env vars in order and falls back to Ollama.

### CLI flags

```bash
eggs-ai --provider anthropic --model claude-sonnet-4-20250514 doctor
eggs-ai --provider groq ask "How do I use wardrobe?"
eggs-ai --provider ollama --model llama3.2 build --describe "minimal server ISO"
```

### Adding custom providers via config file

Create `~/.eggs-ai.yaml` (or run `eggs-ai providers init`):

```yaml
default_provider: deepseek

providers:
  # Any OpenAI-compatible API
  - name: deepseek
    type: custom
    baseUrl: https://api.deepseek.com/v1
    envKey: DEEPSEEK_API_KEY
    model: deepseek-chat

  # Local LM Studio
  - name: lmstudio
    type: custom
    baseUrl: http://localhost:1234/v1
    model: local-model

  # Together AI, Fireworks, Perplexity, OpenRouter, vLLM, etc.
  - name: together
    type: custom
    baseUrl: https://api.together.xyz/v1
    envKey: TOGETHER_API_KEY
    model: meta-llama/Llama-3-70b-chat-hf

  # Alias a built-in provider with a preset model
  - name: claude
    type: anthropic
    model: claude-sonnet-4-20250514
    envKey: ANTHROPIC_API_KEY
```

Then use by name:

```bash
eggs-ai --provider deepseek doctor
eggs-ai --provider lmstudio ask "How do I configure calamares?"
eggs-ai --provider together build --describe "rescue USB"
```

### Programmatic registration

```typescript
import { ProviderRegistry } from 'eggs-ai/providers';

ProviderRegistry.register('my-llm', (config) => ({
  name: 'my-llm',
  async chat(messages) { /* your API call */ },
  async isAvailable() { return true; },
}));
```

### Managing providers

```bash
eggs-ai providers list    # Show all registered providers
eggs-ai providers init    # Create sample ~/.eggs-ai.yaml
```

## Examples

```bash
# Diagnose why your ISO won't boot
eggs-ai doctor "ISO boots to black screen after GRUB"

# Generate a build plan for a classroom distro
eggs-ai build --desktop xfce --installer calamares --describe "student lab machines"

# Explain your current config
eggs-ai config explain

# Generate config for a rescue USB
eggs-ai config generate "minimal rescue ISO with networking tools"

# Get help with Calamares
eggs-ai calamares "How do I add a custom partition scheme?"

# Interactive chat
eggs-ai chat
```

## Architecture

```
eggs-ai/
├── src/
│   ├── agents/              # Domain-specific AI agents
│   │   ├── doctor.ts        # System diagnostics
│   │   ├── build.ts         # ISO build guidance
│   │   ├── config.ts        # Config explain/generate
│   │   ├── calamares.ts     # Calamares assistant
│   │   ├── wardrobe.ts      # Wardrobe/costume help
│   │   └── ask.ts           # General Q&A
│   ├── providers/           # LLM backend abstraction
│   │   ├── base.ts          # LLMProvider interface + ProviderRegistry
│   │   ├── config-loader.ts # ~/.eggs-ai.yaml parser
│   │   ├── gemini.ts        # Google Gemini
│   │   ├── openai.ts        # OpenAI / compatible
│   │   ├── anthropic.ts     # Anthropic Claude
│   │   ├── mistral.ts       # Mistral AI
│   │   ├── groq.ts          # Groq (fast inference)
│   │   ├── ollama.ts        # Ollama (local)
│   │   ├── custom.ts        # Generic OpenAI-compatible (any endpoint)
│   │   └── index.ts         # Registry setup + auto-detection
│   ├── tools/               # System inspection utilities
│   │   ├── system-inspect.ts
│   │   └── eggs-cli.ts
│   ├── knowledge/           # Embedded domain knowledge
│   │   └── eggs-reference.ts
│   └── index.ts             # CLI entry point
├── examples/
│   ├── eggs-ai.yaml         # Sample config with many providers
│   └── register-provider.ts # Programmatic registration example
└── bin/
    └── eggs-ai.js
```

## Integration with eggs-gui

Eggs-AI is designed to integrate with [eggs-gui](https://github.com/Interested-Deving-1896/eggs-gui) — the unified GUI for penguins-eggs (Go daemon + BubbleTea TUI + NodeGUI desktop + NiceGUI web).

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  FRONTENDS                                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │   TUI    │ │ Desktop  │ │   Web    │ │   CLI    │      │
│  │BubbleTea │ │ NodeGUI  │ │ NiceGUI  │ │ eggs-ai  │      │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘      │
│       │             │            │             │             │
│       └──────┬──────┴─────┬──────┘             │             │
│              │            │                    │             │
│         JSON-RPC     HTTP REST                 │             │
│              │            │                    │             │
├──────────────┼────────────┼────────────────────┼─────────────┤
│              │            │                    │             │
│  ┌───────────▼──┐  ┌──────▼──────────┐        │             │
│  │ eggs-daemon  │  │  eggs-ai server │◄───────┘             │
│  │   (Go)       │  │  (Node.js)      │                      │
│  │              │  │                  │                      │
│  │ config.*     │  │ /api/doctor     │                      │
│  │ eggs.*       │  │ /api/build      │                      │
│  │ wardrobe.*   │  │ /api/ask        │                      │
│  │ system.*     │  │ /api/chat       │                      │
│  │ iso.*        │  │ /api/calamares  │                      │
│  │              │  │ /api/wardrobe   │                      │
│  └──────┬───────┘  └──────┬──────────┘                      │
│         │                 │                                  │
│         ▼                 ▼                                  │
│    penguins-eggs     LLM Provider                           │
│    CLI (system)      (Gemini/OpenAI/Ollama/...)             │
└─────────────────────────────────────────────────────────────┘
```

### Three integration paths

#### 1. HTTP API sidecar (recommended)

Run eggs-ai as a sidecar service alongside the eggs-gui daemon. All frontends call it over HTTP.

```bash
# Terminal 1: eggs-gui daemon
make run-daemon

# Terminal 2: eggs-ai API server
eggs-ai serve --port 3737
```

Any frontend calls `POST http://127.0.0.1:3737/api/doctor`, etc.

#### 2. TypeScript SDK (for NodeGUI desktop)

The eggs-gui desktop frontend is TypeScript. Import the SDK directly:

```typescript
// In eggs-gui/desktop/src/ai-panel.ts
import { EggsAiClient } from 'eggs-ai/sdk';

const ai = new EggsAiClient();  // connects to http://127.0.0.1:3737

// AI-powered diagnostics button
async function onDoctorClick() {
  const diagnosis = await ai.doctor('ISO boots to black screen');
  showResultPanel(diagnosis);
}

// AI chat sidebar
async function onChatSubmit(question: string) {
  const answer = await ai.chat(question);
  appendChatMessage(answer);
}

// AI-guided build wizard
async function onBuildWizard() {
  const plan = await ai.build({
    desktop: 'xfce',
    installer: 'calamares',
    compression: 'fast',
    description: 'classroom lab machines',
  });
  showBuildPlan(plan);
}
```

#### 3. JSON-RPC methods (for daemon integration)

Add `ai.*` methods directly to the eggs-gui daemon. The schema is defined in `proto/eggs-ai-rpc.json`:

```
ai.doctor          — AI diagnostics
ai.build           — AI-guided build plan
ai.config.explain  — Explain current config
ai.config.generate — Generate config for a purpose
ai.calamares       — Calamares assistant
ai.wardrobe        — Wardrobe assistant
ai.ask             — General Q&A
ai.providers       — List available providers
```

The Go daemon can shell out to `eggs-ai` or embed the Node.js server and proxy requests.

### HTTP API reference

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| GET | `/api/health` | — | `{ status, version }` |
| GET | `/api/status` | — | System info + daemon connection |
| GET | `/api/providers` | — | `{ providers: [...] }` |
| POST | `/api/doctor` | `{ complaint?, provider? }` | `{ result }` |
| POST | `/api/build` | `{ build: {...}, provider? }` | `{ result }` |
| POST | `/api/config/explain` | `{ provider? }` | `{ result }` |
| POST | `/api/config/generate` | `{ purpose, provider? }` | `{ result }` |
| POST | `/api/calamares` | `{ question?, provider? }` | `{ result }` |
| POST | `/api/wardrobe` | `{ question?, provider? }` | `{ result }` |
| POST | `/api/ask` | `{ question, history?, provider? }` | `{ result }` |
| POST | `/api/chat` | `{ question, provider? }` | `{ result, sessionId }` |

All POST endpoints accept an optional `provider` and `model` field to override the LLM backend per-request.

### Python client (for NiceGUI web frontend)

```python
# In eggs-gui/web/ai_client.py
import httpx

EGGS_AI_URL = "http://127.0.0.1:3737"

async def ask_ai(question: str, provider: str = None) -> str:
    async with httpx.AsyncClient() as client:
        resp = await client.post(f"{EGGS_AI_URL}/api/ask", json={
            "question": question,
            "provider": provider,
        })
        return resp.json()["result"]

async def doctor(complaint: str = None) -> str:
    async with httpx.AsyncClient() as client:
        resp = await client.post(f"{EGGS_AI_URL}/api/doctor", json={
            "complaint": complaint,
        })
        return resp.json()["result"]
```

### Go client (for BubbleTea TUI)

```go
// In eggs-gui/tui/internal/client/ai.go
func (c *Client) AskAI(question string) (string, error) {
    body, _ := json.Marshal(map[string]string{"question": question})
    resp, err := http.Post("http://127.0.0.1:3737/api/ask", "application/json", bytes.NewReader(body))
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    var result struct{ Result string }
    json.NewDecoder(resp.Body).Decode(&result)
    return result.Result, nil
}
```

## Design Principles

- **Local-first**: Works with Ollama for fully offline operation
- **Transparent**: Shows exact commands, never runs destructive operations without confirmation
- **Domain-aware**: Embedded knowledge of all eggs commands, configs, and common issues
- **Optional**: Penguins-eggs works fine without this — it's an assistant, not a dependency

## Related Projects

- [penguins-eggs](https://github.com/pieroproietti/penguins-eggs) — The remastering tool itself
- [ai-pkg](https://github.com/rohankrsingh/ai-pkg) — AI package recommender (inspiration)
- [opencode](https://github.com/anomalyco/opencode) — Open source AI coding agent (inspiration)

## License

MIT
