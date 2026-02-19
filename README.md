# Skills

Agent Skills for AI coding agents.

## What are Skills?

Skills are reusable capabilities for AI agents. They provide procedural knowledge that helps agents accomplish specific tasks more effectively—like plugins or extensions that enhance what your AI agent can do.

## Supported Platforms

These skills work with popular AI coding agents including:

- **[Cursor](https://cursor.com)** - AI-first code editor
- **[Claude Code](https://code.claude.com)** - Anthropic's agentic coding tool
- **[Roo Code](https://roocode.com)** - AI-powered coding assistant for VS Code with multiple specialized modes
- **[OpenClaw](https://openclaw.ai)** - Self-hosted AI coding agent with multi-model support
- **[Antigravity](https://antigravity.google)** - Google's agent-first IDE with autonomous coding agents
- And other AI agents that support the skills ecosystem

## Installation

Install the skills using the [skills CLI](https://skills.sh):

```bash
npx skills add aksuharun/skills
```

This will make all skills in this repository available to your AI agent automatically.

## Available Skills

| Skill                            | Description                                                                                                            |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `git-best-practices`             | Best practices for Git workflows and branching strategies                                                              |
| `readme-creator`                 | Creates high-quality, standardized README.md files for open-source software projects                                  |

## Usage

Once installed, your AI agent will automatically discover and use these skills when relevant. Simply ask your agent to:

- Create a new Git branch to implement a feature
- Merge a pull request following best practices
- Write a README for my project

## Learn More

- [skills.sh](https://skills.sh) - Browse and discover more AI agent skills
- [Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) - Claude's official documentation on agent skills best practices
## Contributing

To add or modify skills:

1. Each skill must have its own directory
2. Each skill directory must contain a `SKILL.md` file with instructions
3. You can use [`anthropic/skill-creator`](https://skills.sh/anthropics/skills/skill-creator) for creating effective skills

## License

MIT License

Copyright (c) 2026 Harun Aksu

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.