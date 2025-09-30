# Sub Agent

## Overview
Sub-agents are specialized AI assistants defined as Markdown files with YAML frontmatter, stored in agent directories.

## File Structure
```yaml
---
name: agent-name
description: Brief description
allowedTools: [Bash, Read, Write]
---

# Agent Instructions

Your detailed prompt here...
```

## Storage Locations
- **User level**: `~/.claude/agents/`
- **Project level**: `.claude/agents/`

## Key Components

### Frontmatter
- `name`: Agent identifier
- `description`: Purpose summary
- `allowedTools`: Tool whitelist (optional)
- `permissions`: Permission settings (optional)

### Prompt Body
Detailed instructions defining agent behavior, constraints, and expected outputs.

## Agent Design Principles
- Single, focused responsibility
- Clear success criteria
- Explicit tool usage guidance
- Context-appropriate memory imports via `@path/to/context`

## Resources

### Tools Available to Claude
@context/sub-agent/tools-available-to-claude.md

Complete reference of tools accessible to Claude agents.