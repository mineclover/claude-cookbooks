# Hooks

## Overview
Hooks execute custom commands before or after Claude Code tool invocations, enabling event-driven automation.

## Configuration
Hooks are defined in `settings.json`:

```json
{
  "hooks": {
    "PreToolUse": {
      "Bash": "echo 'Executing command...'"
    },
    "PostToolUse": {
      "Write": "git add ."
    }
  }
}
```

## Hook Types
- **PreToolUse**: Executes before tool runs
- **PostToolUse**: Executes after tool completes

## Per-Tool Configuration
Hooks can target specific tools:
- `Bash`
- `Write`
- `Edit`
- `Read`
- etc.

## Use Cases
- Logging and auditing
- Automatic git operations
- File system synchronization
- Notification triggers
- Validation checks