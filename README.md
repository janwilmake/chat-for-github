# Chat for GitHub

Chat with any GitHub repository using AI. Hosted at [chat.forgithub.com](https://chat.forgithub.com).

Load one or more repos, and start a conversation powered by Claude. The AI can read files, run bash commands, and explore the codebase to answer your questions.

## Features

- **Multi-repo support** — load multiple repositories into a single conversation
- **Full codebase access** — all repo files are fetched and mounted in a virtual filesystem at `/workspace/{owner}/{repo}/`
- **Tool use** — Claude can run bash commands (`grep`, `find`, `cat`, `jq`, etc.) and read files to explore the code
- **Agent instructions** — automatically discovers and prioritizes `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`, `.github/copilot-instructions.md`, and other AI coding standards
- **Private repos** — OAuth login via [uithub](https://uithub.com) to access private repositories
- **Conversation history** — saved locally in your browser
- **Dark mode** — follows system preference

## How It Works

```
Browser → Cloudflare Worker → uithub (repo context) + Anthropic (Claude Sonnet 4.6)
```

1. You enter one or more GitHub repos (e.g. `vercel/next.js`)
2. The worker fetches the repo tree and agent instruction files from [uithub.com](https://uithub.com)
3. A system prompt is built with agent instructions, READMEs, and the file tree
4. When you send a message, the worker fetches all repo files, mounts them in a virtual bash environment, and streams Claude's response back — including any tool calls

## Tech Stack

- **Runtime:** Cloudflare Workers
- **AI:** Claude Sonnet 4.6 via [Vercel AI SDK](https://sdk.vercel.ai) + `@ai-sdk/anthropic`
- **Repo context:** [uithub.com](https://uithub.com)
- **Virtual shell:** [just-bash](https://www.npmjs.com/package/just-bash)
- **Sessions:** Cloudflare Durable Objects (SQLite-backed)
- **Frontend:** Single HTML file, vanilla JS, no framework

## Development

```bash
npm install
npm run dev     # start local dev server
npm run deploy  # deploy to Cloudflare
```

Requires a `wrangler.jsonc` with Durable Object bindings and the following secrets configured via `wrangler secret put`:

- `ANTHROPIC_API_KEY`
- `UITHUB_CLIENT_ID`
- `UITHUB_CLIENT_SECRET`

## License

MIT
