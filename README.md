# Flamigo Agent Skill

[![Flamigo homepage with flamingos behind the headline “Making beautiful slides has never been easier” and a sample presentation](assets/flamigo-homepage.jpg)](https://flamigo.ai/)

Turn rough notes into beautiful presentations, right from your AI assistant. Paste your notes and review the outline. Flamigo handles the layout, typography, and illustrations. Ask for a different look on any slide, then share the deck or export it to HTML or PowerPoint (Pro).

Install the `flamigo-mcp` skill with the skills CLI. Run this in your project:

```sh
npx skills add wangxinbo1987/flamigo-skills --skill flamigo-mcp
```

Connect your client to `https://api.flamigo.ai/mcp` using Streamable HTTP and browser OAuth. Sign in to your Flamigo account, then ask the assistant to list your presentations to check the connection.

Try “Draft an outline for a six-slide product review from these notes.” Review the outline, then ask the assistant to generate and render the slides.

[Flamigo](https://flamigo.ai/) · [MCP tool and parameter reference](https://flamigo.ai/wiki/mcp) · [Canonical SKILL.md](https://flamigo.ai/skills/flamigo-mcp/SKILL.md)
