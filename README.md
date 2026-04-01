# Canva AI Design Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A framework that lets you create, edit, export, and manage Canva designs using natural language through AI and the Canva MCP Server.

Say what you want -- "Create an Instagram post for a summer sale" -- and the AI handles the Canva API calls to make it happen.

## Project Structure

```txt
Canva_AI_Design_Framework/
|-- .claude/
|   |-- settings.json           # Enabled Claude Code plugins
|   |-- settings.local.json     # Permissions and enabled MCP servers
|-- .mcp.json                   # MCP server configurations
|-- CLAUDE.md                   # Full Canva API reference for AI context
|-- .env                        # Your API keys (gitignored)
|-- .env.example                # Template for required environment variables
|-- .gitignore                  # Standard ignores
|-- LICENSE                     # MIT (Roentek Designs)
|-- README.md                   # This file
```

## MCP Servers

| Server | Purpose |
| -------- | --------- |
| **Canva** | Design creation, editing, search, export, asset management |
| **Tavily** | Web research for design inspiration and content |
| **OpenRouter** | Multi-model AI for generating copy and descriptions |
| **Supabase** | Database storage for design metadata and workflows |

## Setup

1. **Clone** the repo
2. **Copy** `.env.example` to `.env` and fill in your API keys (Tavily, Supabase, OpenRouter)
3. **Open** the project in Claude Code (CLI or VS Code)
4. **Authenticate** with Canva when prompted (browser-based OAuth, no API key needed)

### Requirements

- Node.js v18+ with `npx`
- Claude Code
- A Canva account

## What It Does

- **Create designs** in any dimension or from presets (presentation, doc, whiteboard)
- **Export** to PDF, PNG, JPG, GIF, PPTX, MP4, or HTML
- **Manage assets** -- upload, tag, organize into folders
- **Search** your existing designs and folders
- **Bulk generate** designs from brand templates (Enterprise)
- **Combine servers** -- research trends, generate copy, create the design, and store metadata in one workflow

## Key Reference

- `CLAUDE.md` contains the complete Canva Connect API reference (endpoints, scopes, rate limits, request/response schemas) that the AI uses to fulfill requests.
- [Canva Developer Docs](https://www.canva.dev/docs/)
- [Canva MCP Server](https://www.canva.dev/docs/mcp/)

## License

See [LICENSE](LICENSE) for details.
