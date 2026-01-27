---
name: odin-agent-skills
description: |
  Root skill for the ODIN Agent Skills repository. Provides guidance for working with Agent Skills for the ODIN platform by 4Players.
  Use when: contributing to or maintaining the odin-agent-skills repository, creating new skills, or understanding the repository structure.
---

# ODIN Agent Skills Repository

This repository contains **Agent Skills** for the ODIN platform by 4Players. Agent Skills are markdown files that provide context and documentation to AI coding assistants (Claude Code, Cursor, GitHub Copilot, VS Code, etc.) following the [Agent Skills specification](https://agentskills.io/specification).

**ODIN Platform Products:**
- **ODIN Voice**: Cross-platform real-time voice chat SDK
- **ODIN Fleet**: Game server hosting and deployment platform
- **ODIN Rooms**: Browser-based video conferencing

## Project Structure

```
odin-agent-skills/
├── odin-fundamentals/     # Platform basics, auth, pricing
├── odin-fleet/            # Fleet API and deployment concepts
├── odin-fleet-cli/        # Fleet CLI tool documentation
├── odin-rooms/            # Rooms video conferencing
├── odin-voice-core/       # C/C++ Core SDK
├── odin-voice-nodejs/     # Node.js SDK
├── odin-voice-swift/      # iOS/macOS Swift SDK
├── odin-voice-unity/      # Unity SDK
├── odin-voice-unreal/     # Unreal Engine SDK
└── odin-voice-web/        # Web/JavaScript SDK
```

Each skill folder contains a `SKILL.md` file with YAML frontmatter (`name`, `description`) and comprehensive documentation.

## Skill File Format

Each SKILL.md follows this structure:

```markdown
---
name: skill-name-kebab-case
description: |
  Brief description of the skill.
  Use when: trigger phrases for when this skill should be activated.
---

# Title

Content with code examples, API references, and best practices.
```

## Key Conventions

- **Naming**: Use kebab-case for skill folder names
- **Frontmatter**: Required `name` and `description` fields in YAML
- **"Use when" triggers**: Include in description to help AI assistants know when to apply the skill
- **Code examples**: Include practical, copy-pasteable code snippets
- **Platform-specific**: Each voice SDK has its own skill with platform-appropriate patterns

## Common Tasks

### Adding a New Skill

1. Create a new folder with kebab-case naming: `odin-new-feature/`
2. Create `SKILL.md` with proper YAML frontmatter
3. Include comprehensive documentation with code examples
4. Update `README.md` to list the new skill

### Updating Existing Skills

1. Read the current SKILL.md content
2. Preserve the YAML frontmatter format
3. Update documentation while maintaining code example quality
4. Ensure "Use when" triggers in description remain relevant

### Testing Skills

Skills are text-based documentation files. Verify by:
- Checking YAML frontmatter parses correctly
- Ensuring code examples are syntactically valid
- Confirming links to external documentation are correct

## External Resources

- **ODIN Documentation**: https://docs.4players.io/
- **Voice SDK Docs**: https://docs.4players.io/voice/
- **Fleet Docs**: https://docs.4players.io/fleet/
- **Rooms Docs**: https://docs.4players.io/rooms/
- **Discord**: https://4np.de/discord
- **Agent Skills Spec**: https://agentskills.io/specification
