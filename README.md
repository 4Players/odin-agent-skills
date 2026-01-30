# ODIN Agent Skills

Official [Agent Skills](https://agentskills.io) for [ODIN](https://www.4players.io/odin/) - real-time voice chat SDKs and game server hosting by 4Players.

These skills follow the open [Agent Skills specification](https://agentskills.io/specification) and work with **Claude Code**, **Cursor**, **GitHub Copilot**, **OpenAI Codex**, **VS Code**, and other AI agents that support the standard.

## Available Skills

| Skill | Description |
|-------|-------------|
| **odin-fundamentals** | Platform fundamentals - access keys, tokens, pricing, architecture |
| **odin-voice-unity** | Unity SDK 2.x - OdinRoom, events, 2D/3D spatial audio |
| **odin-voice-unreal** | Unreal Engine SDK - Blueprints, C++, delegates |
| **odin-voice-web** | Web/JavaScript SDK - Room, Peer, DeviceManager |
| **odin-voice-nodejs** | Node.js SDK - server-side integration |
| **odin-voice-swift** | iOS/macOS SDK (OdinKit) - delegates, Combine |
| **odin-voice-core** | C/C++ Core SDK - low-level foundation |
| **odin-fleet** | Game server deployment - overview, API, Docker/Steamworks |
| **odin-fleet-cli** | Fleet CLI - automation, CI/CD pipelines, output formatting, filters |
| **odin-rooms** | Virtual meeting rooms platform |

## Installation

### Option 1: CLI Tools (Easiest)

Using [skills.sh](https://skills.sh/):

```bash
npx @anthropic-ai/skills install 4Players/odin-skills
```

Or using [ai-agent-skills](https://github.com/skillcreatorai/Ai-Agent-Skills):

```bash
npx ai-agent-skills install 4Players/odin-skills
```

Or using [openskills](https://github.com/numman-ali/openskills):

```bash
npx openskills install 4Players/odin-skills
```

These commands install skills for multiple AI agents (Claude Code, Cursor, Codex, VS Code, etc.) automatically.

### Option 2: Manual Installation

```bash
# Clone the repository
git clone https://github.com/4Players/odin-skills.git

# For Claude Code
ln -s $(pwd)/odin-skills ~/.claude/skills/odin-skills

# For Cursor
ln -s $(pwd)/odin-skills .cursor/skills/odin-skills

# For VS Code / GitHub Copilot
ln -s $(pwd)/odin-skills .github/skills/odin-skills
```

### Option 3: Install Individual Skills

```bash
git clone https://github.com/4Players/odin-skills.git

# Link only what you need (example for Claude Code)
ln -s $(pwd)/odin-skills/odin-voice-unity ~/.claude/skills/odin-voice-unity
ln -s $(pwd)/odin-skills/odin-fleet ~/.claude/skills/odin-fleet
```

### Option 4: Project-Scoped Installation

For project-specific skills (only active in that project):

```bash
# For Claude Code
mkdir -p .claude/skills
cp -r odin-skills/odin-voice-unity .claude/skills/

# For Cursor
mkdir -p .cursor/skills
cp -r odin-skills/odin-voice-unity .cursor/skills/
```

## Skill Locations by Agent

| Agent | User Scope | Project Scope |
|-------|------------|---------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| VS Code / Copilot | - | `.github/skills/` |
| Codex | `~/.codex/skills/` | `.codex/skills/` |

## Usage

Once installed, AI agents automatically use these skills when working on ODIN-related tasks. You can also explicitly reference them:

```
"Using the odin-voice-unity skill, help me implement 3D proximity voice chat"
```

## What's Included

Each skill contains:

- **Overview** of the SDK/product
- **Quick start** code examples
- **Key classes and methods** with signatures
- **Event handling patterns**
- **Common implementation patterns**
- **Links to full documentation**

## Agent Skills Specification

These skills follow the [Agent Skills open standard](https://agentskills.io):

- ✅ SKILL.md with YAML frontmatter (`name`, `description`)
- ✅ Kebab-case naming convention
- ✅ "Use when" triggers in descriptions
- ✅ Portable across all compatible AI agents

## Documentation

Full ODIN documentation:

- **ODIN Voice**: https://docs.4players.io/voice/
- **ODIN Fleet**: https://docs.4players.io/fleet/
- **ODIN Rooms**: https://docs.4players.io/rooms/

## Support

- **Discord**: https://4np.de/discord
- **Documentation**: https://docs.4players.io
- **Contact**: https://www.4players.io/company/contact_us/

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Made with ❤️ by [4Players](https://www.4players.io)
