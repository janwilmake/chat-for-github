# repo-prompt

Fetch repository context from [uithub.com](https://uithub.com) and build system prompts optimized for AI coding agents.

Discovers and prioritizes **every known agent instruction standard**:

| File / Pattern | Tool / Standard |
|---|---|
| `AGENTS.md`, `AGENTS.override.md` | Cross-tool open standard (Linux Foundation) |
| `CLAUDE.md` | Claude Code (Anthropic) |
| `GEMINI.md` | Gemini CLI (Google) |
| `.cursorrules`, `.cursor/rules/*.mdc` | Cursor |
| `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` | GitHub Copilot |
| `.windsurfrules`, `.windsurf/rules/*.md` | Windsurf |
| `.clinerules` | Cline / Roo Code / Kilo Code |
| `CONVENTIONS.md` | General conventions |
| `CONTRIBUTING.md` | Contribution guidelines |
| `README.md` (any nesting) | Project documentation |

## Install

```bash
npm install repo-prompt
```

Zero dependencies — just needs `fetch()` (Node 18+, Deno, Bun, Cloudflare Workers).

## Quick Start

```ts
import { buildRepoPrompt } from "repo-prompt";

const result = await buildRepoPrompt({
  owner: "vercel",
  repo: "next.js",
  maxTokens: 30000,
});

// result.prompt — ready-to-use system prompt string
// result.tree — the file tree with token counts
// result.agentFiles — all discovered agent instruction files
// result.readmeFiles — all discovered README files
// result.size — token/size metadata
```

## What the prompt looks like

The assembled prompt follows best practices for coding agent context:

```
# Repository: vercel/next.js
Total size: ~1,234,567 tokens

---
## Agent Instructions

### AGENTS.md (cross-tool standard)
<!-- source: AGENTS.md -->

<contents of AGENTS.md>

### CLAUDE.md (Claude Code)
<!-- source: CLAUDE.md -->

<contents of CLAUDE.md>

---
## README Files

### README.md
<!-- source: README.md -->

<contents of README.md>

---
## File Tree
The following is the repository file tree with approximate token counts per file:

├── src/
│   ├── index.ts (450 tokens)
│   └── utils.ts (200 tokens)
└── package.json (100 tokens)
```

Agent instructions are placed **first** (before READMEs and tree) because agents prioritize content that appears early in their context window.

## API

### `buildRepoPrompt(options): Promise<RepoPromptResult>`

The main function. Fetches everything and assembles the prompt.

```ts
interface RepoPromptOptions {
  owner: string;          // GitHub owner
  repo: string;           // GitHub repo name
  branch?: string;        // Branch (default: main)
  apiKey?: string;        // GitHub token for private repos
  bearerToken?: string;   // uithub OAuth bearer token
  maxTokens?: number;     // Token budget (default: 50000)
  extraIncludes?: string[]; // Additional glob patterns to include
  includeTree?: boolean;  // Include file tree (default: true)
  baseUrl?: string;       // uithub base URL
}
```

### `getAgentFiles(options): Promise<AgentFile[]>`

Fetch only the agent instruction files — useful when you have your own prompt template.

```ts
const files = await getAgentFiles({ owner: "facebook", repo: "react" });

for (const f of files) {
  console.log(f.path, f.kind, f.content.length);
}
```

### `getRepoTree(options): Promise<{ tree, size }>`

Fetch only the file tree with token counts.

```ts
const { tree, size } = await getRepoTree({ owner: "denoland", repo: "deno" });
console.log(`Total: ${size.totalTokens} tokens`);
```

### `renderTree(tree, prefix?): string`

Render a uithub token tree object as a pretty-printed string with `├──` connectors.

### Constants

- `AGENT_FILE_GLOBS` — all glob patterns used to discover agent files
- `README_GLOBS` — glob patterns for README files
- `KIND_LABELS` — human-readable labels for each `AgentFileKind`

## Use Cases

**As an MCP server context provider** — fetch repo context on-demand for any MCP tool that needs codebase awareness.

**In a CLI tool** — pipe the prompt into any LLM:

```bash
npx tsx examples/basic.ts facebook react | llm -s "Summarize the architecture"
```

**In a Cloudflare Worker** — build a repo-aware API:

```ts
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    const [owner, repo] = url.pathname.slice(1).split("/");

    const result = await buildRepoPrompt({ owner, repo, maxTokens: 20000 });
    return new Response(result.prompt, {
      headers: { "Content-Type": "text/plain" },
    });
  },
};
```

## How It Works

1. **Tree fetch** — calls `uithub.com/{owner}/{repo}?omitFiles=true` to get the full file tree with token counts
2. **Agent files fetch** — calls uithub with an `include` glob covering all known agent instruction patterns + READMEs
3. **Classification** — each returned file is classified by its kind (AGENTS.md, CLAUDE.md, etc.)
4. **Assembly** — builds the prompt with agent instructions first (highest priority), then READMEs, then the file tree

Only **2 HTTP requests** to uithub per call.

## License

MIT
