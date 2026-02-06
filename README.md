# ODIN Agent Skill

Official [Agent Skill](https://agentskills.io) for [ODIN](https://www.4players.io/odin/) - real-time voice chat SDKs and game server hosting by 4Players.

This skill follows the open [Agent Skills specification](https://agentskills.io/specification) and works with **Claude Code**, **Cursor**, **GitHub Copilot**, **OpenAI Codex**, **VS Code**, and other AI agents that support the standard.

## What's Covered

| Topic | Reference |
|-------|-----------|
| Platform fundamentals | `references/fundamentals.md` |
| Unity voice chat SDK | `references/voice-unity.md` |
| Unreal Engine voice chat SDK | `references/voice-unreal.md` |
| Web/JavaScript voice chat SDK | `references/voice-web.md` |
| Node.js server-side voice SDK | `references/voice-nodejs.md` |
| iOS/macOS voice chat SDK (OdinKit) | `references/voice-swift.md` |
| C/C++ core SDK | `references/voice-core.md` |
| Game server deployment (Fleet) | `references/fleet.md` |
| Fleet CLI tool | `references/fleet-cli.md` |
| Virtual meeting rooms | `references/rooms.md` |
| Pricing details | `references/pricing.md` |

## Installation

### Option 1: CLI Tools (Easiest)

Using [skills.sh](https://skills.sh/):

```bash
npx skills install 4Players/odin-agent-skills
```

Or using [ai-agent-skills](https://github.com/skillcreatorai/Ai-Agent-Skills):

```bash
npx ai-agent-skills install 4Players/odin-agent-skills
```

Or using [openskills](https://github.com/numman-ali/openskills):

```bash
npx openskills install 4Players/odin-agent-skills
```

These commands install the skill for multiple AI agents (Claude Code, Cursor, Codex, VS Code, etc.) automatically.

### Option 2: Manual Installation

```bash
# Clone the repository
git clone https://github.com/4Players/odin-agent-skills.git

# For Claude Code
ln -s "$(pwd)/odin-agent-skills" ~/.claude/skills/odin

# For Cursor
ln -s "$(pwd)/odin-agent-skills" .cursor/skills/odin

# For VS Code / GitHub Copilot
ln -s "$(pwd)/odin-agent-skills" .github/skills/odin
```

### Option 3: Project-Scoped Installation

For project-specific skills (only active in that project):

```bash
git clone https://github.com/4Players/odin-agent-skills.git

# For Claude Code
mkdir -p .claude/skills
cp -r odin-agent-skills .claude/skills/odin

# For Cursor
mkdir -p .cursor/skills
cp -r odin-agent-skills .cursor/skills/odin
```

## Skill Locations by Agent

| Agent | User Scope | Project Scope |
|-------|------------|---------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| VS Code / Copilot | - | `.github/skills/` |
| Codex | `~/.codex/skills/` | `.codex/skills/` |

## Usage

Once installed, AI agents automatically use this skill when working on ODIN-related tasks. You can also explicitly reference it:

```
"Using the odin skill, help me implement 3D proximity voice chat in Unity"
```

## What's Included

The skill contains:

- **Overview** of each SDK/product
- **Quick start** code examples
- **Key classes and methods** with signatures
- **Event handling patterns**
- **Common implementation patterns**
- **Links to full documentation**

## Agent Skills Specification

This skill follows the [Agent Skills open standard](https://agentskills.io):

- SKILL.md with YAML frontmatter (`name`, `description`)
- Kebab-case naming convention
- "Use when" triggers in description
- Portable across all compatible AI agents

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
