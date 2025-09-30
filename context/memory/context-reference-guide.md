# Context Reference Guide

## Context Reference Guidelines

### Preventing Accidental Context Inclusion
When writing documentation or examples that mention `@` references, use backticks to prevent AI from interpreting them as active context:

**Correct:**
```markdown
Use `@context/sub-agent/tools-available-to-claude.md` to reference tool information
```

**Incorrect:**
```markdown
Use @context/sub-agent/tools-available-to-claude.md to reference tool information
```

The backticks ensure the `@reference` is treated as literal text, not an active context inclusion.

### Path Structure
All `@` references are relative to the project root:
- Format: `@context/{topic}/{filename}.md`
- Example: `@context/memory/context-reference-guide.md`

### Best Practices
- Name memory files descriptively
- Keep memory files focused on specific topics
- Update memory files as project context evolves
- Use `@` references only when context is actually needed

## Writing Format Guidelines (Base)

These guidelines serve as the base format for all context/ documentation:

### 1. Structure
- Start with clear Overview section
- Include Key Concepts for quick understanding
- Use modular document references with `@` syntax (without backticks for actual imports)
- No external reference links in index.md (references go in reference.md only)

### 2. Content Quality
- Curate and refine information directly
- Trust in well-organized, focused content over many references
- Keep information actionable and practical

### 3. Modularity
- Break down complex topics into separate .md files
- Reference them using `@context/{topic}/{filename}.md` pattern (no backticks for actual imports)
- Each module should be self-contained and focused
- Index files contain only headlines and module references

### 4. Reference vs Import
- **Import (actual reference)**: `@context/memory/guide.md` - Claude will load this content
- **Documentation (mention only)**: Use backticks like `@context/memory/guide.md` to prevent loading