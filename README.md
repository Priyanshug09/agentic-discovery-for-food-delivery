# Agentic Discovery — PM Portfolio

AI-powered food discovery demo built as a portfolio piece for Product Manager roles.

**Live demo:** [your-url.vercel.app]

---

## What it demonstrates

- Claude API for real NLU (intent extraction + natural language responses)
- Deterministic scoring engine (auditable, no hallucination risk in ranking)
- PM eval panel showing the AI architecture layer — a layer most demos hide
- Human-first UX: agent sounds like a friend, not a bot

## Architecture

```
User query
    ↓
Claude API (NLU Parser)     ← extracts constraints + generates reaction/intro
    ↓
Local Scorer (deterministic) ← scores all 15 restaurants, picks top 3
    ↓
Claude API (Closing)         ← personalised 1-2 sentence recommendation
    ↓
Eval Panel                   ← heuristic metrics (intent match, constraint sat, groundedness)
```

> **Eval transparency note:** The eval scores are heuristic proxies, not production-grade metrics. A real system would use labelled query sets and offline A/B testing. This is intentional — showing awareness of the gap between demo metrics and real evals is the point.

---

## Local setup

```bash
npm install
cp .env.example .env
# Add your Anthropic API key to .env
npm run dev
```

## Deploy to Vercel

1. Push this repo to GitHub
2. Import on [vercel.com](https://vercel.com)
3. Add environment variable: `VITE_ANTHROPIC_API_KEY` = your key
4. Deploy

> ⚠️ **Note:** The API key is used client-side (fine for a portfolio demo). For production, route through a backend proxy.

---

Built by Priyanshu | Product Manager
