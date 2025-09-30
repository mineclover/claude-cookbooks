# Headless Mode

## Overview
Headless mode enables programmatic, non-interactive Claude Code execution for automation and backend integration.

## Core Usage

### Basic Invocation
```bash
claude -p "your task"
```

### Key Flags
- `--print`, `-p`: Non-interactive mode
- `--output-format`: Response format (text/json/stream-json)
- `--resume`, `-r`: Continue existing session
- `--allowedTools`: Tool access control
- `--permission-mode`: Permission handling strategy

## Common Patterns

### Automated Code Review
```bash
claude -p "Review changes for security issues" \
  --allowedTools "Read,Grep" \
  --output-format json
```

### Multi-turn Automation
Use `--resume` with session IDs to maintain conversation context across invocations.

## Best Practices
- Use JSON output for programmatic parsing
- Implement error handling and timeouts
- Respect rate limits
- Manage session lifecycle