# Skills

Agent Skills for AI coding agents.

## What are Skills?

Skills are reusable capabilities for AI agents. They provide procedural knowledge that helps agents accomplish specific tasks more effectively—like plugins or extensions that enhance what your AI agent can do.

## Features

- Reusable, self-contained skill packages that enhance AI agent capabilities
- Cross-platform support across multiple AI coding agents
- Declarative `SKILL.md` format with bundled references, scripts, and assets
- Install via the [skills CLI](https://skills.sh) with a single command

## Supported Platforms

These skills work with popular AI coding agents including:

- **[Cursor](https://cursor.com)** — AI-first code editor
- **[Claude Code](https://code.claude.com)** — Anthropic's agentic coding tool
- **[Roo Code](https://roocode.com)** — AI-powered coding assistant for VS Code with multiple specialized modes
- **[OpenClaw](https://openclaw.ai)** — Self-hosted AI coding agent with multi-model support
- **[Antigravity](https://antigravity.google)** — Google's agent-first IDE with autonomous coding agents
- And other AI agents that support the skills ecosystem

## Installation

Install the skills using the [skills CLI](https://skills.sh):

```bash
npx skills add aksuharun/skills
```

This makes all skills in this repository available to your AI agent automatically.

## Available Skills

| Skill | Description |
| --- | --- |
| `git-best-practices` | Best practices for Git workflows and branching strategies |
| `lucide-icons` | Use Lucide icons correctly across React, Vue, Svelte, Solid, Preact, Angular, Astro, React Native, Vanilla JS, and static environments |
| `raison-sdk` | Raison JavaScript/TypeScript prompt management SDK for rendering templates and querying prompt data |
| `readme-creator` | Creates high-quality, standardized README.md files for open-source software projects |
| `reka-ui` | Use when building accessible Vue.js interfaces with Reka UI (Radix Vue) |

## Usage

Once installed, your AI agent will automatically discover and use these skills when relevant. Example prompts:

- *Create a new Git branch to implement a feature*
- *Write a README for my project*
- *Set up Raison SDK to manage prompts in my Express app*
- *Add a Lucide icon to my React component*
- *Add an accessible accordion with Reka UI to my Vue application*

## Learn More

- [skills.sh](https://skills.sh) — Browse and discover more AI agent skills
- [Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — Claude's official documentation on agent skills best practices

## Contributing

1. Each skill must have its own directory containing a `SKILL.md` file with instructions
2. Fork the repository, create a feature branch, and submit a pull request
3. Use [`anthropic/skill-creator`](https://skills.sh/anthropics/skills/skill-creator) for creating effective skills

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for details.