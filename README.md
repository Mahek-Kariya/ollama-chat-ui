# Ollama Chat UI

A privacy-first AI developer assistant — chat with locally running LLMs via a clean, professional web interface. No data leaves your machine.

![Next.js](https://img.shields.io/badge/Next.js-14-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?style=flat-square&logo=typescript)
![Tailwind CSS](https://img.shields.io/badge/TailwindCSS-3-38bdf8?style=flat-square&logo=tailwindcss)
![Redux Toolkit](https://img.shields.io/badge/Redux_Toolkit-2-764abc?style=flat-square&logo=redux)
![Ollama](https://img.shields.io/badge/Ollama-local-ff6b35?style=flat-square)

---

## Why this exists

Cloud AI tools require you to paste your proprietary code into third-party servers. This app solves that — full AI assistance, 100% local, zero telemetry.

---

## Features

- **Streaming chat** — real-time token streaming via Vercel AI SDK
- **Code-aware input** — paste code with syntax highlighting, ask questions about it
- **Model switcher** — swap between any locally installed Ollama model from the UI
- **Markdown rendering** — full markdown + fenced code blocks with copy-to-clipboard
- **Chat history** — persisted in localStorage, survives page refresh
- **Clean UI** — professional dark/light mode, keyboard shortcuts

---

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 14 (App Router) | File-based routing, server components, streaming |
| Language | TypeScript | Type safety across the full stack |
| State | Redux Toolkit | Predictable chat/model state, devtools |
| AI / Streaming | Vercel AI SDK v3 (`useChat`) | Built-in streaming, abort, error handling |
| Styling | Tailwind CSS + shadcn/ui | Utility-first with accessible, unstyled primitives |
| LLM | Ollama (local) | Privacy-first, runs llama3, codellama, mistral, etc. |
| Package manager | npm | |

---

## Architecture — how it works

This project is split into two independent pieces:

```
┌─────────────────────────────────┐       ┌──────────────────────────────┐
│   Frontend  (this repository)   │       │   Ollama  (runs on YOUR PC)  │
│                                 │       │                              │
│   Next.js app                   │──────▶│   Local LLM server           │
│   Deployed on Vercel            │       │   localhost:11434            │
│   (or run locally on :3000)     │       │   NOT deployed, NOT shared   │
└─────────────────────────────────┘       └──────────────────────────────┘
```

> **Important for visitors / reviewers:** The frontend is publicly deployed on Vercel, but the AI responses require **Ollama running on your own machine**. The app will load fine without it — you will just see a connection error when you try to send a message. Follow the [Ollama setup](#2-install-and-configure-ollama) section below to run it locally end-to-end.

---

## Hardware requirements

Ollama runs entirely on your local machine. Below are the tiers based on available hardware:

| Tier | RAM | GPU | Recommended model | Expected speed |
|---|---|---|---|---|
| **Minimum** | 8 GB | Any / none | `llama3.2:3b` | 3–8 s first token |
| **Recommended** | 16 GB | Integrated (Intel Iris Xe, AMD Radeon) | `llama3.2:3b` | 2–5 s first token |
| **Better** | 16 GB | Discrete GPU, 4–6 GB VRAM | `llama3.1:8b` | 1–3 s first token |
| **Best** | 32 GB | Discrete GPU, 8+ GB VRAM | `codellama:13b` | < 1 s first token |

> **No GPU? No problem.** Ollama runs on CPU. It is slower but fully functional. The `3b` model is specifically chosen to work well on CPU-only machines.

---

## Software requirements

| Requirement | Minimum version | Notes |
|---|---|---|
| OS | Windows 10, macOS 12, Ubuntu 20.04 | Windows 11 fully supported |
| Node.js | 18.17 | Required for Next.js 14. Install via [nodejs.org](https://nodejs.org) or `nvm` |
| npm | 9.x | Comes with Node.js |
| Git | 2.x | [git-scm.com](https://git-scm.com) |
| Ollama | Latest | [ollama.com/download](https://ollama.com/download) — free, open source |
| Browser | Chrome 110+, Firefox 110+, Edge 110+ | Any modern browser works |

---

## Getting started

### 1. Clone and install the frontend

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/ollama-chat-ui.git
cd ollama-chat-ui

# Install dependencies
npm install

# Copy environment file
cp .env.example .env.local
# No edits needed — defaults work out of the box
```

### 2. Install and configure Ollama

**Step 1 — Download Ollama**

Go to [https://ollama.com/download](https://ollama.com/download) and install for your OS.

- Windows: run the `.exe` installer. Ollama starts automatically as a background service.
- macOS: drag to Applications. Run it once to start the menu bar service.
- Linux: `curl -fsSL https://ollama.com/install.sh | sh`

**Step 2 — Pull a model**

Open a terminal and run:

```bash
# Recommended for most machines (fast on CPU, great for code Q&A)
ollama pull llama3.2:3b

# If you have 8+ GB VRAM on a discrete GPU (much better responses)
ollama pull llama3.1:8b

# Code-specialist model (needs discrete GPU)
ollama pull codellama:7b
```

> First pull downloads the model weights — `3b` is ~2 GB, `8b` is ~5 GB. This is a one-time download.

**Step 3 — Verify Ollama is running**

```bash
# Should print a list of your installed models
ollama list

# Should return JSON with model names
curl http://localhost:11434/api/tags
```

If `curl` is not available on Windows, open this URL in your browser: `http://localhost:11434/api/tags`

**Step 4 — Enable CORS for the frontend** (Windows only)

On Windows, Ollama blocks browser requests by default. Set this environment variable before starting Ollama:

```powershell
# In PowerShell (run once, then restart Ollama)
[System.Environment]::SetEnvironmentVariable("OLLAMA_ORIGINS", "*", "User")
```

Then restart Ollama: open Task Manager → find Ollama → End Task → reopen Ollama from the Start menu.

> On macOS and Linux this step is not needed.

### 3. Run the application

```bash
# In the project directory
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

You should see the chat interface load. Select your model from the sidebar dropdown and start chatting.

---

## Troubleshooting

**"Failed to fetch" or "Connection refused" when sending a message**
- Ollama is not running. Open the Ollama app or run `ollama serve` in a terminal.
- On Windows, check that the CORS env variable is set (see Step 4 above) and that you restarted Ollama after setting it.

**"Model not found" error**
- Run `ollama list` to confirm the model is installed.
- Run `ollama pull llama3.2:3b` if it is missing.

**Responses are very slow**
- This is normal on CPU-only machines for the first few seconds. `llama3.2:3b` is the fastest usable model without a GPU.
- Close other memory-heavy applications (browsers with many tabs, IDEs, etc.) to free RAM for Ollama.

**Port 3000 already in use**
- Next.js will automatically try port 3001. Check the terminal output for the actual URL.

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `OLLAMA_BASE_URL` | `http://localhost:11434` | Ollama API endpoint |
| `DEFAULT_MODEL` | `llama3.2:3b` | Model pre-selected on first visit |

---

## Project structure

```
ollama-chat-ui/
├── app/                        # Next.js App Router
│   ├── api/
│   │   └── chat/
│   │       └── route.ts        # Streaming chat API route
│   ├── layout.tsx
│   └── page.tsx
│
├── components/                 # Global reusable UI components
│   ├── ui/                     # shadcn/ui primitives (never edit directly)
│   ├── chat/
│   │   ├── ChatWindow.tsx
│   │   ├── ChatMessage.tsx
│   │   ├── ChatInput.tsx
│   │   └── CodeBlock.tsx
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   └── Header.tsx
│   └── model/
│       └── ModelSwitcher.tsx
│
├── hooks/                      # Global custom hooks (used in 2+ features)
│   ├── useLocalStorage.ts
│   └── useOllamaModels.ts
│
├── lib/                        # Pure utilities, no React
│   ├── utils.ts                # cn() and shared helpers
│   └── constants.ts
│
├── services/                   # All external API calls
│   └── ollamaService.ts        # Ollama REST API wrapper
│
├── store/                      # Redux Toolkit
│   ├── index.ts
│   ├── hooks.ts                # Typed useAppSelector / useAppDispatch
│   └── slices/
│       ├── chatSlice.ts
│       └── modelSlice.ts
│
├── types/                      # Shared TypeScript interfaces
│   └── index.ts
│
├── .env.example
├── tailwind.config.ts
├── tsconfig.json
└── next.config.ts
```

---

## Git workflow

```
main          ← stable, production-ready only. Never commit directly.
└── dev       ← active development base. PRs merge here first.
    ├── feature/chat-ui
    ├── feature/model-switcher
    ├── feature/code-highlighting
    └── feature/chat-history
```

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full branching and commit guide.

---

## Commit convention

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

```
feat:      new feature
fix:       bug fix
chore:     tooling, deps, config
refactor:  restructure without behaviour change
docs:      documentation only
style:     formatting, whitespace
test:      adding or updating tests
```

---

## Available scripts

```bash
npm run dev          # development server with hot reload
npm run build        # production build
npm run start        # serve production build locally
npm run lint         # ESLint
npm run type-check   # TypeScript check without emitting files
```

---

## Roadmap

- [x] Project scaffold and git workflow
- [ ] `feature/chat-ui` — core streaming chat interface
- [ ] `feature/model-switcher` — live model selection from UI
- [ ] `feature/code-highlighting` — syntax-highlighted code input + output
- [ ] `feature/chat-history` — localStorage persistence with sidebar history
- [ ] Deploy frontend to Vercel

---

## License

MIT