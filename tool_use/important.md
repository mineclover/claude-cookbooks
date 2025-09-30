# Tool Use - Important Technical Details & Best Practices

This document contains critical technical details, best practices, and important considerations extracted from the tool_use cookbook examples.

---

## Table of Contents
1. [Tool Definition Best Practices](#tool-definition-best-practices)
2. [Tool Choice Strategies](#tool-choice-strategies)
3. [Security Considerations](#security-considerations)
4. [Memory & Context Management](#memory--context-management)
5. [Error Handling](#error-handling)
6. [Performance Optimization](#performance-optimization)
7. [Integration Patterns](#integration-patterns)

---

## Tool Definition Best Practices

### Clear Tool Descriptions
- **Critical:** Tool descriptions must be clear and specific about what the tool does
- Include information about what data is returned
- Specify any constraints or requirements

**Example:**
```python
{
    "name": "get_customer_info",
    "description": "Retrieves customer information based on their customer ID. Returns the customer's name, email, and phone number.",
    "input_schema": {...}
}
```

### Input Schema Design
- Use appropriate JSON Schema types: `string`, `number`, `integer`, `boolean`, `array`, `object`
- Mark required fields explicitly in the `required` array
- Provide descriptions for each property to guide Claude's understanding
- Use format validators when applicable (e.g., `"format": "email"`)
- Set constraints with `minimum`, `maximum` for numeric values
- Use `default` values for optional parameters

**Example with Validation:**
```python
{
    "type": "object",
    "properties": {
        "priority": {
            "type": "integer",
            "minimum": 1,
            "maximum": 5,
            "default": 3,
            "description": "The priority level (1-5)"
        }
    }
}
```

### Open-Ended Schemas
- Use `"additionalProperties": True` when you don't know the exact keys in advance
- Provide clear instructions via prompting on how to structure the dynamic keys
- Best for characteristic extraction, flexible data capture

---

## Tool Choice Strategies

### Auto (Default)
**When to use:** Most scenarios where Claude should decide whether tools are needed

**Best Practices:**
- Write detailed system prompts to guide tool usage
- Be explicit about when Claude should/shouldn't use tools
- Include current date/time if relevant to decision-making
- Prevent over-eager tool calling with clear instructions

**Example System Prompt:**
```python
system_prompt = f"""
Answer as many questions as you can using your existing knowledge.
Only search the web for queries that you can not confidently answer.
Today's date is {date.today().strftime("%B %d %Y")}
If you think a user's question involves something in the future that hasn't happened yet, use the search tool.
"""
```

**Key Insight:** "Your prompt matters!" - Detailed prompts prevent unnecessary tool calls.

---

### Forcing Specific Tools
**When to use:**
- Guaranteed structured JSON extraction
- Mandatory processing steps (e.g., logging, validation)
- Ensuring consistent output format

**Syntax:**
```python
tool_choice={"type": "tool", "name": "print_sentiment_scores"}
```

**Important:**
- Still employ basic prompt engineering even when forcing tools
- Provide context in prompts for better quality results
- Use for data extraction patterns where you need predictable output

---

### Any Tool Choice
**When to use:**
- Chatbots that must always respond through tools (e.g., SMS bots)
- Preventing raw text responses
- Ensuring all interactions go through specific channels

**Syntax:**
```python
tool_choice={"type": "any"}
```

**Best Practice:** Provide clear instructions about which tool to use based on available information

**Example:**
```python
system_prompt = """
All your communication with a user is done via text message.
Only call tools when you have enough information to accurately call them.
Do not call the get_customer_info tool until a user has provided you with their username.
"""
```

---

## Security Considerations

### Calculator Tool - eval() Warning
**⚠️ CRITICAL SECURITY WARNING:**
```python
def calculate(expression):
    # Remove any non-digit or non-operator characters
    expression = re.sub(r'[^0-9+\-*/().]', '', expression)
    result = eval(expression)  # ⚠️ BAD PRACTICE - For demo only!
    return str(result)
```

**Why this is dangerous:**
- `eval()` executes arbitrary Python code
- Potential for code injection attacks
- Should NEVER be used in production

**Recommendation:** Use a proper math parser library like `ast.literal_eval()` with expression validation, or a dedicated math evaluation library.

---

### Memory Tool - Path Traversal Protection
**⚠️ CRITICAL:** Always validate paths to prevent directory traversal attacks

**From memory_tool.py implementation:**
- Validate all paths before file operations
- Prevent access outside designated memory directory
- Use absolute path resolution and checking
- Sanitize user-provided paths

**Example Pattern:**
```python
def _validate_path(self, path):
    """Validate that path is within base_path"""
    abs_path = os.path.abspath(os.path.join(self.base_path, path))
    if not abs_path.startswith(os.path.abspath(self.base_path)):
        raise ValueError(f"Path {path} is outside base directory")
    return abs_path
```

---

### Memory Tool - Memory Poisoning
**⚠️ CRITICAL RISK:** Memory files are read back into Claude's context, making them a potential vector for prompt injection.

**Attack Scenario:**
1. Malicious user provides input that gets stored in memory
2. Input contains prompt injection instructions
3. Future sessions read this memory and Claude follows malicious instructions

**Mitigation Strategies:**
1. **Content Sanitization:** Filter dangerous patterns before storing in memory
2. **Memory Scope Isolation:** Per-user or per-project memory directories
3. **Memory Auditing:** Log all memory operations for review
4. **Prompt Engineering:** Instruct Claude to ignore instructions found in memory content
5. **Access Control:** Strict permissions on memory file directories

---

### Do NOT Store in Memory:
- ❌ Passwords or API keys
- ❌ Personally Identifiable Information (PII)
- ❌ Sensitive business data
- ❌ Complete conversation history
- ❌ User secrets or credentials

### DO Store in Memory:
- ✅ Task-relevant patterns (e.g., debugging patterns)
- ✅ Learned preferences (non-sensitive)
- ✅ Code review insights
- ✅ Project-specific knowledge
- ✅ Analysis techniques

---

## Memory & Context Management

### Understanding Memory vs Context
**Short-term memory (Context):**
- Current conversation history
- Tool results from recent calls
- Cleared automatically via context editing
- Limited by model's context window (200k tokens for Claude 4)

**Long-term memory (Memory Tool):**
- Persistent across conversations
- File-based storage under `/memories` directory
- Client-side implementation (you control storage)
- Must be explicitly managed

---

### Context Editing Configuration
**Purpose:** Automatically clear old tool results when context grows large

**Configuration Example:**
```python
CONTEXT_MANAGEMENT = {
    "edits": [
        {
            "type": "clear_tool_uses_20250919",
            "trigger": {"type": "input_tokens", "value": 5000},
            "keep": {"type": "tool_uses", "value": 1},
            "clear_at_least": {"type": "input_tokens", "value": 50}
        }
    ]
}
```

**Parameters Explained:**
- `trigger`: When to start clearing (e.g., at 5000 input tokens)
- `keep`: How many recent tool uses to preserve
- `clear_at_least`: Minimum tokens to clear per operation

**Important:** Adjust thresholds based on your tool result sizes:
- Small tool results (50-150 tokens): Use lower thresholds like demo
- Large tool results (1000+ tokens): Use higher thresholds (3000-5000 tokens)

---

### Memory Best Practices
1. **Organize with Structure:** Use clear directory hierarchies
   ```
   /memories/
     /concurrency_patterns/
       thread_safety.md
     /api_patterns/
       rest_best_practices.md
   ```

2. **Descriptive File Names:** Use names that indicate content
   - ✅ `race_condition_patterns.md`
   - ❌ `notes.md`

3. **Periodic Cleanup:** Review and remove outdated memory
   - Set up regular audits
   - Remove obsolete patterns
   - Archive old learnings

4. **Scope Isolation:** Separate memories by project/user
   ```python
   memory = MemoryToolHandler(base_path=f"./memory/{project_id}")
   ```

---

### Memory Tool Commands
| Command | Purpose | Use Case |
|---------|---------|----------|
| `view` | Show directory or file contents | Check what's already stored |
| `create` | Create or overwrite file | Store new patterns |
| `str_replace` | Replace text in existing file | Update learned insights |
| `insert` | Insert text at specific line | Add to existing knowledge |
| `delete` | Remove file or directory | Clean up outdated info |
| `rename` | Move or rename files | Reorganize memory structure |

---

### What Claude Actually Learns
**Key Insight:** Claude learns semantic patterns, not just syntax

**Example - Thread Safety Pattern:**
- **Symptom:** Inconsistent results in concurrent operations
- **Cause:** Shared mutable state modified from multiple threads
- **Solution:** Use locks, thread-safe structures, or return results
- **Red flags:** Instance variables in callbacks, unused locks, counter increments

**Cross-Language Application:**
- Pattern learned from Python applies to Go, Java, Rust concurrency
- Architectural understanding transcends language syntax
- Gets better with each review

---

## Error Handling

### Async Code Error Handling Issues
**Problem Identified in api_client_v1.py:**
```python
except aiohttp.ClientError as e:
    return {"endpoint": endpoint, "error": str(e)}
```

**Issues:**
- Loses exception type information
- No stack traces for debugging
- Missing HTTP status codes for failed requests

**Better Approach:**
```python
except aiohttp.ClientError as e:
    return {
        "endpoint": endpoint,
        "error": str(e),
        "error_type": type(e).__name__,  # Preserve type
        "success": False,
    }
```

---

### HTTP Status Code Validation
**Problem:** Treating 4xx/5xx responses as success if JSON is valid

**Fix:**
```python
async with session.get(url) as response:
    if response.status >= 400:
        return {
            "endpoint": endpoint,
            "status": response.status,
            "error": f"HTTP {response.status}",
            "success": False,
        }
    data = await response.json()
```

---

## Performance Optimization

### Parallel Tool Calls (Claude 3.7 Sonnet)
**Problem:** Claude 3.7 Sonnet tends to make sequential tool calls even when parallel calls are possible

**Solution:** Introduce a "batch tool" meta-pattern

**Batch Tool Definition:**
```python
batch_tool = {
    "name": "batch_tool",
    "description": "Invoke multiple other tool calls simultaneously",
    "input_schema": {
        "type": "object",
        "properties": {
            "invocations": {
                "type": "array",
                "items": {
                    "properties": {
                        "name": {"type": "string"},
                        "arguments": {"type": "string"}
                    }
                }
            }
        }
    }
}
```

**Benefit:** Reduces latency by allowing simultaneous execution of independent tool calls

---

### Session Management for Async Clients
**Anti-pattern:** Creating new session for each request
```python
async def fetch_all(self):
    async with aiohttp.ClientSession() as session:  # ❌ New session every time
        ...
```

**Better Pattern:** Reusable session with context manager
```python
class AsyncAPIClient:
    async def __aenter__(self):
        self._session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self._session:
            await self._session.close()
```

**Usage:**
```python
async with AsyncAPIClient(url) as client:
    results = await client.fetch_all(endpoints)
```

---

### Stateful Design Anti-Pattern
**Problem:** Storing results as instance variables
```python
class WebScraper:
    def __init__(self):
        self.results = []  # ❌ State accumulates

    def scrape_urls(self, urls):
        # Appends to self.results
        return self.results
```

**Issues:**
- Not reusable (calling twice accumulates results)
- Not thread-safe
- Confusing API (returns AND stores)
- Memory leak potential

**Better:** Return results directly without storing
```python
def scrape_urls(self, urls):
    results = []
    # ... collect results ...
    return results  # ✅ No state mutation
```

---

## Integration Patterns

### Conversation Loop Pattern
**Standard pattern for tool use:**
```python
while True:
    response = client.messages.create(
        model=MODEL_NAME,
        messages=messages,
        tools=tools,
    )

    if response.stop_reason == "tool_use":
        # Extract tool calls
        tool_results = []
        for content in response.content:
            if content.type == "tool_use":
                result = execute_tool(content.name, content.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": content.id,
                    "content": str(result)
                })

        # Add to conversation
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
    else:
        break  # Done
```

**Key Points:**
- Keep calling API while tool_use occurs
- Always include tool_use_id in results
- Append both assistant response and tool results to messages
- Stop when stop_reason is not "tool_use"

---

### Pydantic Validation Pattern
**Best practice for type safety:**

```python
from pydantic import BaseModel, EmailStr, Field

class Author(BaseModel):
    name: str
    email: EmailStr  # Automatic email validation

class Note(BaseModel):
    note: str
    author: Author
    priority: int = Field(ge=1, le=5, default=3)  # Range validation
    is_public: bool = False
```

**Benefits:**
- Compile-time type checking
- Runtime validation
- Automatic error messages
- Self-documenting code

**Usage:**
```python
def process_tool_call(tool_name, tool_input):
    if tool_name == "save_note":
        note = Note(**tool_input)  # Validates on construction
        # Guaranteed valid if no exception
```

---

### Vision + Tools Pattern
**Multimodal input with structured output:**

```python
message_list = [
    {
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": base64_encoded_image
                }
            },
            {
                "type": "text",
                "text": "Extract nutrition information from this label."
            }
        ]
    }
]
```

**Use cases:**
- Document data extraction
- Form processing
- Receipt parsing
- Visual inspection with structured reporting

---

## Production Considerations

### Supported Models for Memory Tool
- Claude Opus 4 (`claude-opus-4-20250514`)
- Claude Opus 4.1 (`claude-opus-4-1-20250805`)
- Claude Sonnet 4 (`claude-sonnet-4-20250514`)
- Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`)

**Status:** Beta feature - share feedback to help improve

---

### Context Window Limits
- Claude 4: 200k tokens
- Seems large but fills quickly with:
  - Long conversations
  - Multiple tool results
  - Large code files
  - Repeated patterns

**Management strategy:** Use context editing to keep recent context while preserving memory

---

### Cost Considerations
**Computational cost scales quadratically with context size**
- Longer context = higher API costs
- Context editing reduces token processing
- Memory tool reduces redundant explanations across sessions
- Balance between context retention and cost

---

### Testing Memory Tool
**Location:** `/Users/junwoobang/project/claude-cookbooks/tool_use/tests/test_memory_tool.py`

**Important tests to implement:**
- Path traversal prevention
- Content sanitization
- Permission validation
- Error handling
- Concurrent access (if applicable)

---

## Common Pitfalls

### 1. Over-Eager Tool Calling
**Problem:** Claude calls tools when it shouldn't
**Solution:** Detailed system prompts with clear guidance

### 2. Insufficient Tool Descriptions
**Problem:** Claude misuses tools or calls wrong tool
**Solution:** Comprehensive descriptions with examples of when to use

### 3. Missing Required Parameters
**Problem:** Tool fails due to missing data
**Solution:** Mark all required fields in schema, provide defaults for optional

### 4. Not Handling Tool Errors
**Problem:** Tool execution fails, breaks conversation loop
**Solution:** Wrap tool execution in try-except, return error messages

### 5. Forgetting tool_use_id
**Problem:** Claude can't match results to calls
**Solution:** Always include tool_use_id in tool_result

### 6. Accumulating Context Without Clearing
**Problem:** Hit context limits in long sessions
**Solution:** Implement context management configuration

### 7. Storing Sensitive Data in Memory
**Problem:** Security/privacy violations
**Solution:** Filter content before storage, use encryption if needed

### 8. Not Testing Tool Schemas
**Problem:** Schema errors only discovered in production
**Solution:** Test with various inputs, validate schemas programmatically

---

## Additional Resources

### Official Documentation
- **Claude API Reference:** https://docs.claude.com/en/api/messages
- **Memory Tool Docs:** https://docs.claude.com/en/docs/agents-and-tools/tool-use/memory-tool
- **Context Engineering Blog:** https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

### Code Examples
- **GitHub Action for PR Reviews:** https://github.com/anthropics/claude-code-action
- **Tool Use Cookbook:** https://github.com/anthropics/anthropic-cookbook/tree/main/tool_use

### Support
- **Support Portal:** https://support.claude.com

---

## Quick Reference Checklist

### Before Deploying Tool Use to Production:
- [ ] Tool descriptions are clear and comprehensive
- [ ] All required parameters are marked in schema
- [ ] Input validation is implemented (types, ranges, formats)
- [ ] Security: eval() and similar dangerous functions removed
- [ ] Security: Path traversal protection implemented
- [ ] Security: Memory content is sanitized
- [ ] Error handling covers all tool execution paths
- [ ] Context management configured for long sessions
- [ ] Memory isolation implemented (per-user/project)
- [ ] Sensitive data exclusion rules in place
- [ ] System prompts guide appropriate tool usage
- [ ] Tool choice strategy matches use case
- [ ] Pydantic models for complex data structures
- [ ] Tests cover tool execution and edge cases
- [ ] Monitoring/logging for tool usage patterns
- [ ] Cost tracking for context usage