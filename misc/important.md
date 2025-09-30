# Important Technical Details and Best Practices

This document contains key technical insights, best practices, and important implementation details extracted from the misc folder cookbooks.

## Table of Contents
1. [Prompt Caching](#prompt-caching)
2. [Evaluation Design](#evaluation-design)
3. [Content Moderation](#content-moderation)
4. [JSON Output](#json-output)
5. [Long Context Performance](#long-context-performance)
6. [PDF Processing](#pdf-processing)
7. [Batch Processing](#batch-processing)
8. [Citations](#citations)
9. [Token Management](#token-management)
10. [Best Practices](#best-practices)

---

## Prompt Caching

### Key Benefits
- **Reduces latency by >2x** for cached content
- **Reduces costs up to 90%** for repeated context
- Cache lifetime: **5 minutes** (ephemeral)
- Minimum cacheable content: **1024 tokens** (2048 for Claude 3.5 Sonnet/Haiku)
- Multiple cache breakpoints supported (up to 4)

### Implementation
```python
{
    "type": "text",
    "text": "Your large content here...",
    "cache_control": {"type": "ephemeral"}
}
```

### Required Header
```python
extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"}
```

### Best Practices
1. **Place cache breakpoints strategically** - Put them at natural boundaries (end of documents, end of examples)
2. **Cache stable content** - Don't cache content that changes frequently
3. **Order matters** - Cached content must come in the same order to get cache hits
4. **System messages cache well** - Long system prompts are ideal candidates
5. **Multi-turn conversations** - Cache conversation history incrementally
6. **Speculative caching** - Warm cache while user is typing (can reduce TTFT by 90%)

### Cost Structure
- Cache writes: 25% more expensive than base input tokens
- Cache reads: 90% cheaper than base input tokens
- Break-even: Reading cached content 2+ times becomes cost-effective

### Monitoring
Check these usage fields:
- `cache_creation_input_tokens` - Tokens written to cache
- `cache_read_input_tokens` - Tokens read from cache
- `input_tokens` - Regular (uncached) input tokens

---

## Evaluation Design

### Core Principles
1. **Make evals specific to your task** - Don't use generic questions
2. **Match real-world distribution** - Eval difficulty should reflect production usage
3. **Prefer volume over perfection** - Better to have 100 good questions than 10 perfect ones
4. **Design for automation** - Structure tasks to allow code-based grading when possible

### Grading Methods (Ranked by Preference)

#### 1. Code-Based Grading (Best)
- **Pros:** Fast, reliable, no cost
- **Cons:** Limited to exact matches or pattern matching
- **Use for:** Multiple choice, numerical answers, format validation
- **Example:** Exact string match, regex validation, JSON structure checks

#### 2. Model-Based Grading
- **Pros:** Handles nuanced evaluation, scalable, transparent reasoning
- **Cons:** Costs money, requires validation
- **Use for:** Open-ended responses, tone analysis, factual accuracy
- **Key tip:** Always ask for reasoning BEFORE the grade in your grader prompt

#### 3. Human Grading (Last Resort)
- **Pros:** Most capable, handles any task
- **Cons:** Slow, expensive, doesn't scale
- **Use for:** Final validation, edge cases, subjective quality

### Grader Prompt Template
```
You will be provided an answer that an assistant gave to a question, and a rubric that instructs you on what makes the answer correct or incorrect.

Here is the answer that the assistant gave to the question.
<answer>{answer}</answer>

Here is the rubric on what makes the answer correct or incorrect.
<rubric>{rubric}</rubric>

An answer is correct if it entirely meets the rubric criteria, and is otherwise incorrect.

First, think through whether the answer is correct or incorrect based on the rubric inside <thinking></thinking> tags. Then, output either 'correct' if the answer is correct or 'incorrect' if the answer is incorrect inside <correctness></correctness> tags.
```

### Tips for Better Evals
- **Use descriptive examples in rubrics** - "deadlifts (pulling leg exercise), not squats (pushing)"
- **Test the grader** - Validate model-based graders on known samples
- **Reformulate for automation** - Convert open-ended questions to multiple choice when possible
- **Make wrong answers detailed** - In multiple choice, detailed wrong answers prevent length-based guessing

---

## Content Moderation

### Customization Advantages
- Guidelines defined directly in prompt
- Easy to update and iterate
- No training data required
- Context-specific rules

### Improvement Techniques

#### Chain of Thought (Recommended)
- Add `<thinking>` tags for reasoning
- Improves accuracy on edge cases
- Provides transparency for auditing

#### Few-Shot Examples
- Include 2-5 examples of BLOCK and ALLOW cases
- Examples should cover edge cases
- More effective than description alone for nuanced guidelines

### Guidelines Structure
```
BLOCK CATEGORY:
- [Specific, actionable criteria]
- [Include examples where helpful]

ALLOW CATEGORY:
- [Specific, actionable criteria]
- [Default permissive for unlisted content]
```

### Important Notes
- **Be specific** - "No spam" is vague, "No unsolicited commercial messages" is clear
- **Provide context** - Examples help Claude understand edge cases
- **Test edge cases** - Mild profanity, satire, educational content about sensitive topics
- **Consider false positives** - Overly strict moderation frustrates users

---

## JSON Output

### Methods

#### 1. Prefill (Recommended for Clean JSON)
```python
messages=[
    {"role": "user", "content": "Give me JSON with athlete names and sports."},
    {"role": "assistant", "content": "{"}  # Prefill
]
```
- Eliminates preamble
- Forces JSON to start immediately
- Remember to add back the opening `{` when parsing

#### 2. XML Tags (For Multiple JSON Objects)
```
Output a dictionary in <athlete_sports> tags.
Then output individual items in separate <athlete_name> tags.
```
- Use when returning multiple separate JSON objects
- Makes parsing reliable with regex: `<tag>(.*?)</tag>`
- Prevents ambiguity about JSON boundaries

#### 3. Instructions Only (Fallback)
```
Return only valid JSON, no other text.
```
- Less reliable than above methods
- May still include preambles
- Requires string parsing to extract JSON

### Parsing Tips
```python
# Method 1: Find first { and last }
json_start = response.index("{")
json_end = response.rfind("}")
json_obj = json.loads(response[json_start:json_end + 1])

# Method 2: Extract from tags
import re
pattern = r'<tag>(.*?)</tag>'
json_strings = re.findall(pattern, response, re.DOTALL)
```

### Important Notes
- **No formal JSON mode** - Claude doesn't have constrained sampling
- **Validation required** - Always validate JSON structure
- **Error handling** - Wrap parsing in try/except
- **Schema enforcement** - For strict schemas, validate after parsing

---

## Long Context Performance

### Key Findings (from 100K context testing)

#### Position Matters
- **Beginning:** 85% accuracy
- **Middle:** 75% accuracy
- **End:** 80% accuracy

Middle positions are slightly harder due to "lost in the middle" effect.

#### Improvement Techniques

**1. Scratchpad (Always Helps)**
```
Pull 2-3 relevant quotes from the record that pertain to the question and write them inside <scratchpad></scratchpad> tags.
```
- Improves accuracy by 5-10%
- Forces careful reading
- Provides transparency

**2. Contextual Examples (Highly Effective)**
- Random examples: Minimal improvement
- Contextual examples (from same document): +15-20% accuracy
- 5 examples > 2 examples
- Examples must be relevant to the context

**3. Question Design**
- Make questions self-contained: "What does the NASA document state about..." not "What does this document state..."
- Future-proofs questions if they become separated from context
- Improves accuracy by avoiding ambiguous references

### Best Practices for Long Context
1. Use scratchpad for reasoning
2. Include 3-5 contextual examples when possible
3. Put critical information near beginning or end
4. Make questions self-describing
5. Test at your target context length during development

---

## PDF Processing

### Supported Features
- **Text extraction:** Automatic
- **Image processing:** Yes (analyzed but not citeable)
- **Maximum size:** Based on token limits of model
- **Format:** Base64-encoded

### Setup
```python
# Required header during beta
default_headers={"anthropic-beta": "pdfs-2024-09-25"}

# Encoding
with open(pdf_path, "rb") as pdf_file:
    binary_data = pdf_file.read()
    base64_encoded = base64.b64encode(binary_data)
    base64_string = base64_encoded.decode("utf-8")

# Usage
{
    "type": "document",
    "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": base64_string
    }
}
```

### Model Support
- **claude-3-5-sonnet-20241022** (recommended)
- More models may be added after beta

### Important Notes
- PDFs are processed page by page
- Both text and visual elements (charts, graphs) are analyzed
- Only text can be cited (images provide context but no citations)
- Large PDFs consume significant tokens
- Consider extracting text directly for text-only PDFs to save costs

---

## Batch Processing

### Key Benefits
- **50% cost reduction** compared to standard API
- Asynchronous processing
- Ideal for non-urgent workloads

### Use Cases
- Bulk evaluations
- Data processing pipelines
- Report generation
- Content classification at scale

### Process Flow
1. **Create batch** - Submit multiple requests
2. **Monitor status** - Poll for completion
3. **Retrieve results** - Download when "ended"

### Status Types
- `in_progress` - Actively processing
- `ended` - Complete (successful)
- `canceled` - User canceled
- `expired` - Exceeded time limit (24 hours)

### Request Counts
```python
batch.request_counts.succeeded  # Successfully processed
batch.request_counts.errored    # Failed requests
batch.request_counts.processing # Currently running
batch.request_counts.canceled   # User canceled
batch.request_counts.expired    # Time limit exceeded
```

### Important Notes
- Batches expire after **24 hours**
- Results available for **24 hours after completion**
- No guaranteed processing time
- Can include any message types (text, images, etc.)
- Each request needs a unique `custom_id` for tracking

### Best Practices
1. **Use descriptive custom_ids** - Makes result processing easier
2. **Handle errors gracefully** - Check `result.type` for each result
3. **Monitor regularly** - Poll every 30-60 seconds
4. **Download results promptly** - Results expire after 24 hours
5. **Validate inputs** - Invalid requests fail individually

---

## Citations

### Supported Models
- `claude-3-5-sonnet-20241022`
- `claude-3-5-haiku-20241022`

### Document Types and Citation Formats

**1. Plain Text Documents**
- **Citation type:** `char_location`
- **Granularity:** Sentence-level minimum
- **Use for:** Text documents, articles, help center content

**2. PDF Documents**
- **Citation type:** `page_location`
- **Granularity:** Sentence-level minimum
- **Includes:** `start_page_number`, `end_page_number` (1-indexed)
- **Use for:** Research papers, reports, books

**3. Custom Content Documents**
- **Citation type:** `content_block_location`
- **Granularity:** Your defined chunks
- **Use for:** Custom chunking strategies, specific granularity needs

### Key Features
- **Automatic chunking** - Plain text and PDFs chunked into sentences
- **Multi-sentence citations** - Can cite multiple consecutive sentences
- **No hallucinated citations** - Only cites provided sources
- **Structured output** - Citations returned as structured data

### Context Field
```python
{
    "type": "document",
    "source": {...},
    "title": "Doc Title",
    "context": "This won't be cited but informs responses",
    "citations": {"enabled": True}
}
```

**Use context for:**
- Metadata (publication date, author, version)
- Contextual retrieval information
- Usage instructions
- Warnings or disclaimers

### Advantages Over Prompt-Based Citations
1. **Token efficiency** - Doesn't require outputting full quotes
2. **No hallucination** - Can't cite non-existent sources
3. **Higher accuracy** - Better recall and precision in testing
4. **Structured data** - Easy to parse and display

### Parsing Citations
```python
for content in response.content:
    if hasattr(content, 'citations') and content.citations:
        for citation in content.citations:
            print(f"Cited: {citation.cited_text}")
            print(f"From: {citation.document_title}")
            if citation.type == "page_location":
                print(f"Pages: {citation.start_page_number}-{citation.end_page_number}")
```

---

## Token Management

### Exceeding max_tokens

**Problem:** Claude stops mid-response when hitting max_tokens limit.

**Solution:** Continue generation using prefill
```python
# First request
response1 = client.messages.create(
    model=MODEL,
    max_tokens=4096,
    messages=[{"role": "user", "content": prompt}]
)

# Check if truncated
if response1.stop_reason == "max_tokens":
    # Continue from where it left off
    response2 = client.messages.create(
        model=MODEL,
        max_tokens=4096,
        messages=[
            {"role": "user", "content": prompt},
            {"role": "assistant", "content": response1.content[0].text}
        ]
    )
```

**Cost Note:** You pay for the first response's output tokens as input tokens in the continuation request.

### Token Counting
- Use Claude's tokenizer: `client.get_tokenizer()`
- Different models may have different tokenizers
- Count tokens before sending to avoid unexpected costs

### Context Window Limits
- **Claude 3 Opus:** 200K tokens
- **Claude 3.5 Sonnet:** 200K tokens
- **Claude 3 Haiku:** 200K tokens
- Includes both input and output

---

## Best Practices

### Prompt Engineering

1. **Use XML tags for structure**
   - Clear boundaries: `<document>`, `<question>`, `<answer>`
   - Helps Claude parse multi-part prompts
   - Makes few-shot examples clearer

2. **Reasoning before answers**
   - Use `<thinking>` or `<scratchpad>` tags
   - Improves accuracy on complex tasks
   - Provides transparency

3. **Examples over descriptions**
   - 2-3 examples > long explanations
   - Show, don't just tell
   - Include edge cases

4. **Be specific about output format**
   - Specify tags for output: "Write answer in <answer> tags"
   - Define exact format requirements
   - Include example outputs

### System Messages

1. **Define role and constraints**
   - Who is Claude? What can/can't it do?
   - Clear boundaries prevent scope creep

2. **Cache long system messages**
   - System messages are stable across requests
   - Perfect candidate for caching

3. **Include evaluation criteria**
   - Tell Claude how to judge quality
   - Helps maintain consistency

### Multi-Turn Conversations

1. **Maintain context efficiently**
   - Use prompt caching for conversation history
   - Cache breakpoint at last user message

2. **Structure for clarity**
   - Clear role attribution
   - Consistent formatting

3. **Manage context window**
   - Summarize or truncate old messages when needed
   - Keep most recent exchanges in full

### Error Handling

1. **Validate inputs**
   - Check content before sending
   - Enforce length limits
   - Sanitize user input

2. **Handle API errors gracefully**
   - Implement retry logic with exponential backoff
   - Check `stop_reason` for truncation
   - Validate response format

3. **Parse outputs safely**
   - Use try/except for JSON parsing
   - Regex with fallbacks for tag extraction
   - Validate structure before using

### Testing

1. **Test with realistic data**
   - Use production-like examples
   - Include edge cases
   - Test at target scale

2. **Measure what matters**
   - Track latency, cost, accuracy
   - A/B test prompt variations
   - Monitor cache hit rates

3. **Iterate based on data**
   - Collect failures for analysis
   - Improve prompts based on errors
   - Continuously refine evals

---

## Model Selection Guide

### When to Use Each Model

**Claude 3.5 Sonnet**
- Complex reasoning tasks
- Long context with nuance
- High accuracy requirements
- Function calling
- Citations

**Claude 3.5 Haiku**
- High-volume simple tasks
- Fast response time critical
- Cost-sensitive applications
- Simple Q&A, classification
- Citations support

**Claude 3 Opus**
- Maximum intelligence needed
- Complex multi-step reasoning
- Research-grade accuracy
- When cost is secondary

### Cost-Performance Tradeoffs
1. Start with Haiku for prototyping
2. A/B test against Sonnet
3. Use Opus only when necessary
4. Leverage caching to reduce costs
5. Consider batch processing for non-urgent workloads

---

## Common Pitfalls

### 1. Not Using Caching
- Missing 90% cost savings on repeated context
- Cache breakpoints should be at stable content boundaries

### 2. Generic Evals
- Tests that don't reflect real usage
- Solution: Task-specific questions matching production distribution

### 3. Ignoring Context Position
- Assuming all positions in context are equal
- Solution: Test critical info at multiple positions

### 4. Overloading Prompts
- Too many instructions reduce clarity
- Solution: Prioritize, use examples, structure with XML

### 5. No Error Handling
- Assuming perfect responses
- Solution: Validate, parse safely, handle edge cases

### 6. Prompt Engineering in Production
- Hardcoded prompts difficult to update
- Solution: Template-based systems, version control

### 7. Not Monitoring Performance
- Flying blind on cost/latency/accuracy
- Solution: Log metrics, track over time, alert on degradation

---

## Additional Resources

- **Anthropic Documentation:** https://docs.anthropic.com
- **Prompt Engineering Guide:** https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering
- **API Reference:** https://docs.anthropic.com/en/api
- **Community Cookbook Examples:** https://github.com/anthropics/claude-cookbooks

---

## Version Notes

These best practices are based on:
- Claude 3 and 3.5 model family
- API version as of the cookbook creation date
- Beta features: prompt-caching-2024-07-31, pdfs-2024-09-25

Always check latest documentation for updates and new features.