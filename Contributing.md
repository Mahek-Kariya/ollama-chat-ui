# Contributing & Dev Workflow

This document covers the complete branching strategy, setup steps, and coding standards for this project.

---

## One-time setup (do this once after cloning)

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/ollama-chat-ui.git
cd ollama-chat-ui

# Create and push dev branch
git checkout -b dev
git push -u origin dev

# Set dev as the default base for all feature branches
# (you will never branch off main directly)

# Install deps
npm install

# Set up environment
cp .env.example .env.local
# Edit .env.local with your Ollama URL if non-default
```

---

## Branch structure

```
main
│   Production-ready only.
│   Protected — never commit or push directly.
│   Receives PRs from dev at milestone boundaries only.
│
└── dev
    │   Active development base.
    │   Always up to date with latest stable features.
    │   Receives PRs from feature branches.
    │
    ├── feature/chat-ui
    ├── feature/model-switcher
    ├── feature/code-highlighting
    └── feature/chat-history
```

---

## Starting a new feature

```bash
# Always start from an up-to-date dev
git checkout dev
git pull origin dev

# Create your feature branch
git checkout -b feature/your-feature-name

# Work, commit often in small logical units
git add .
git commit -m "feat: describe what this commit does"

# Push to remote
git push -u origin feature/your-feature-name
```

### Branch naming

| Type | Pattern | Example |
|---|---|---|
| Feature | `feature/short-description` | `feature/model-switcher` |
| Bug fix | `fix/short-description` | `fix/stream-abort-crash` |
| Chore | `chore/short-description` | `chore/add-eslint-config` |
| Refactor | `refactor/short-description` | `refactor/chat-slice` |

---

## Committing

Follow [Conventional Commits](https://www.conventionalcommits.org/). Every commit message:

```
<type>: <short imperative description>
```

Types: `feat` `fix` `chore` `refactor` `docs` `style` `test`

Good examples:
```bash
git commit -m "feat: add useChat hook with streaming support"
git commit -m "feat: render code blocks with syntax highlighting"
git commit -m "fix: prevent duplicate messages on stream reconnect"
git commit -m "chore: install and configure shadcn/ui"
git commit -m "refactor: extract ChatMessage into its own component"
git commit -m "docs: update README with model setup instructions"
```

Rules:
- One logical change per commit
- Lowercase after the colon
- No period at the end
- Present tense, imperative mood ("add" not "added")

---

## Merging a feature into dev

When your feature is complete and working:

```bash
# Update your branch with any dev changes
git checkout feature/your-feature-name
git rebase origin/dev          # keeps history linear

# Push (force push needed after rebase)
git push --force-with-lease origin feature/your-feature-name

# On GitHub: open a Pull Request from feature/X → dev
# Merge strategy: Squash and merge (keeps dev history clean)
# Delete the feature branch after merging
```

---

## Merging dev into main (milestone releases only)

Only do this when a full milestone is complete and tested.

```bash
git checkout main
git pull origin main
git merge --no-ff dev -m "chore: release milestone 1 - chat UI"
git push origin main

# Tag the release
git tag -a v0.1.0 -m "Milestone 1: Core chat UI"
git push origin v0.1.0
```

---

## Feature build order (recommended)

Work through these in sequence — each one builds on the last:

1. **`feature/project-scaffold`** — Next.js init, Tailwind, shadcn/ui, Redux store, folder structure, `.env.example`
2. **`feature/chat-ui`** — API route, `useChat` hook, `ChatWindow`, `ChatMessage`, `ChatInput`, streaming
3. **`feature/model-switcher`** — `ollamaService.ts` fetches `/api/tags`, `ModelSwitcher` component, `modelSlice`
4. **`feature/code-highlighting`** — syntax-highlighted input + output code blocks, copy button
5. **`feature/chat-history`** — `chatSlice` persistence, sidebar with past sessions

---

## Folder conventions

### Global (shared across the whole app)

```
hooks/          Custom React hooks used in 2+ features
lib/            Pure functions, constants, cn() helper — no React
services/       All external API calls (Ollama REST, etc.)
store/          Redux store, slices, typed hooks
types/          Shared TypeScript interfaces and types
components/ui/  shadcn/ui primitives — never edit these directly
```

### Feature-level (scoped to one feature)

When a hook, component, or util is only used by one feature, co-locate it:

```
components/chat/
    ChatWindow.tsx
    ChatMessage.tsx
    ChatInput.tsx
    CodeBlock.tsx
    useChatScroll.ts      ← hook only used by chat feature, lives here
```

Only promote to the global `hooks/` folder when a second feature needs it.

---

## Component rules

- One component per file
- File name matches the exported component name (PascalCase)
- Props interface named `[ComponentName]Props` defined in the same file
- No inline styles — Tailwind classes only
- Use `cn()` from `lib/utils.ts` for conditional classes

```tsx
// Good
interface ChatMessageProps {
  role: 'user' | 'assistant'
  content: string
}

export function ChatMessage({ role, content }: ChatMessageProps) {
  return (
    <div className={cn('rounded-lg p-4', role === 'user' ? 'bg-muted' : 'bg-background')}>
      {content}
    </div>
  )
}
```

---

## shadcn/ui usage

- Install components individually: `npx shadcn-ui@latest add button`
- Never edit files in `components/ui/` directly
- Build your app components **on top of** these primitives
- If you need to restyle a primitive, wrap it in your own component

---

## TypeScript rules

- No `any` — use `unknown` and narrow, or define a proper type
- All API response shapes typed in `types/index.ts`
- Redux state typed — use `RootState` and `AppDispatch` from `store/hooks.ts`
- Prefer `interface` for object shapes, `type` for unions and primitives

---

## Environment variables

- Never commit `.env.local` (it's in `.gitignore`)
- Always add new variables to `.env.example` with a placeholder value and comment
- Prefix client-side variables with `NEXT_PUBLIC_`

```bash
# .env.example
OLLAMA_BASE_URL=http://localhost:11434   # Ollama server URL
DEFAULT_MODEL=llama3.2:3b               # Default model on first load
```

---

## Before every PR

Run this checklist locally:

```bash
npm run lint          # zero ESLint errors
npm run type-check    # zero TypeScript errors
npm run build         # production build succeeds
```