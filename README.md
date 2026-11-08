# Flamigo Agent Skill

Use Flamigo to turn notes into a presentation, edit slide content, compare designs, and export HTML or PowerPoint from an MCP-capable assistant.

Install the `flamigo-mcp` skill with the skills CLI. From a checkout of this repository:

```sh
npx skills add . --skill flamigo-mcp
```

Connect your client to `https://api.flamigo.ai/mcp` using Streamable HTTP and browser OAuth. Sign in to your Flamigo account, then ask the assistant to list your presentations to check the connection.

Try “Draft an outline for a six-slide product review from these notes.” Review the outline, then ask the assistant to generate and render the slides.

[Flamigo](https://flamigo.ai/) · [MCP tool and parameter reference](https://flamigo.ai/wiki/mcp) · [Canonical SKILL.md](https://flamigo.ai/skills/flamigo-mcp/SKILL.md)
