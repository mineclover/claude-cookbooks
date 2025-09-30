# Output Styles

## Overview
Output styles modify Claude Code's system prompt to adapt behavior beyond standard software engineering tasks.

## Built-in Styles
1. **Default**: Standard software engineering mode
2. **Explanatory**: Adds educational "Insights" while working
3. **Learning**: Collaborative mode with insights and human contribution opportunities

## Usage

### Change Style
```bash
/output-style              # Interactive menu
/output-style learning     # Direct switch
```

### Create Custom Style
```bash
/output-style:new
```

## Custom Style Structure
- Name
- Description
- Custom instructions/behaviors (markdown)

## Storage
- User level: `~/.claude/output-styles/`
- Project level: `.claude/output-styles/`

## Comparison
Output styles are distinct from:
- CLAUDE.md memory files
- `--append-system-prompt` flag
- Sub-agents
- Custom slash commands

They directly modify the core system prompt rather than adding supplementary context.