# Important Technical Details and Best Practices for Multimodal Claude

This document contains key technical details, best practices, and important considerations extracted from the Claude multimodal cookbook examples.

---

## Image Input Methods

### Base64 Encoding (Local Files)
```python
with open("image.jpeg", "rb") as image_file:
    binary_data = image_file.read()
    base_64_encoded_data = base64.b64encode(binary_data)
    base64_string = base_64_encoded_data.decode('utf-8')
```

**Usage in API:**
```python
{
    "type": "image",
    "source": {
        "type": "base64",
        "media_type": "image/jpeg",
        "data": base64_string
    }
}
```

### URL-Based Images
```python
import httpx
IMAGE_DATA = base64.b64encode(httpx.get(IMAGE_URL).content).decode("utf-8")
```

**Key Point:** Even URL-based images need to be downloaded and base64-encoded before sending to Claude.

---

## PDF Document Support

### Requirements
- **Model:** Only `claude-3-5-sonnet-20241022` supports PDF feature (as of the cookbook)
- **Beta Header:** Must include `"anthropic-beta": "pdfs-2024-09-25"` header
- **Page Limit:** Maximum 100 pages across all documents in a single request

### Implementation
```python
client = Anthropic(default_headers={
    "anthropic-beta": "pdfs-2024-09-25"
})

# PDF content block
{
    "type": "document",
    "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": base64_string
    }
}
```

**Processing:** Claude uses both extracted text AND vision when processing PDFs.

---

## Prompt Engineering for Vision

### 1. Role Assignment Reduces Hallucination
**Bad:** "How many dogs are in this picture?"
**Good:** "You have perfect vision and pay great attention to detail which makes you an expert at counting objects in images. How many dogs are in this picture?"

**Impact:** Significantly improves accuracy in counting tasks.

### 2. Chain-of-Thought for Complex Visual Tasks
**Pattern:**
```
Before providing the answer in <answer> tags, think step by step in <thinking> tags and analyze every part of the image.
```

**Use Cases:**
- Counting objects
- Complex calculations from charts
- Detailed analysis

### 3. Visual Prompting
**Technique:** Embed instructions directly in images (arrows, text annotations, highlights)

**Advantages:**
- No text prompt needed
- Clear indication of areas of interest
- Natural for diagram analysis

**Examples:**
- Drawing arrows to specific chart elements
- Circling numbers for comparison
- Writing questions directly on images

### 4. Few-Shot Learning with Images
**When to Use:**
- Reading analog displays (speedometers, gauges)
- Establishing unit conventions
- Product classification
- Specialized visual patterns

**Structure:**
```python
messages = [
    {
        "role": "user",
        "content": [image1, text1]
    },
    {
        "role": "assistant",
        "content": [response1]
    },
    {
        "role": "user",
        "content": [image2, text2]
    },
    {
        "role": "assistant",
        "content": [response2]
    },
    {
        "role": "user",
        "content": [target_image, query]
    }
]
```

**Note:** Set `temperature=0` for consistent results with few-shot examples.

### 5. Selective Output Instructions
**Pattern:** "Only output [X] and nothing else."

**Examples:**
- "Only output the code."
- "Only output the JSON and nothing else."
- "Only output the text and nothing else."

**Purpose:** Filter unwanted explanations and get clean, parseable results.

---

## Multi-Image Processing

### Capabilities
- Claude can process multiple images in a single request
- Images are analyzed together, not independently
- Useful for split documents (long receipts, multi-page forms)

### Use Cases
1. **Split Documents:** Reconstructing information across image fragments
2. **Before/After Comparisons:** Analyzing changes between images
3. **Visual Classification:** Providing labeled examples with target image
4. **Multi-page Analysis:** Processing related images together

### Implementation
```python
content = [
    {"type": "image", "source": {...}},  # Image 1
    {"type": "image", "source": {...}},  # Image 2
    {"type": "image", "source": {...}},  # Image 3
    {"type": "text", "text": "Your question"}
]
```

---

## Model Selection Strategy

### Claude 3 Opus
**Use For:**
- Final synthesis and analysis
- Complex reasoning tasks
- Generating sub-agent prompts
- High-stakes accuracy requirements

### Claude 3 Haiku
**Use For:**
- Sub-agent tasks
- Parallel document processing
- Cost-effective batch operations
- Quick extraction tasks

### Claude 3.5 Sonnet
**Use For:**
- PDF document processing (only model with PDF support)
- Balance of speed and capability

---

## Common Pitfalls and Solutions

### 1. Arithmetic Errors in Chart Analysis
**Problem:** Claude may calculate incorrectly when extracting numbers from charts.

**Solution:** Provide Claude with a calculator tool for arithmetic operations.

### 2. Color-Dependent Charts
**Problem:** Grouped bar charts with many color-coded groups can confuse Claude.

**Solution:** Ask Claude to first identify colors using HEX codes before analyzing data.

### 3. Complex Chart Analysis
**Problem:** Dense visualizations with many data points may be misread.

**Solution:** Ask Claude to "First describe every data point you see in the document" (similar to Chain of Thought).

### 4. Unit Confusion (mph vs km/h)
**Problem:** Claude may misinterpret measurement units.

**Solution:** Use few-shot examples to establish conventions.

### 5. PDF Text Extraction Limitations
**Problem:** Traditional PDF parsers fail with chart-heavy documents.

**Solution:** Convert PDF pages to images and use vision capabilities.

---

## Sub-Agent Architecture Pattern

### Use Case
Processing multiple documents in parallel with lightweight models, then synthesizing with a powerful model.

### Architecture
1. **Orchestrator (Opus):** Generates specific prompts for sub-agents
2. **Sub-Agents (Haiku):** Process individual documents in parallel
3. **Synthesizer (Opus):** Combines results and generates final output

### Implementation Pattern
```python
# Step 1: Generate sub-agent prompt
haiku_prompt = generate_haiku_prompt(user_question)

# Step 2: Process documents in parallel
with ThreadPoolExecutor() as executor:
    results = list(executor.map(process_document, documents))

# Step 3: Combine results with XML tags
extracted_info = ""
for result in results:
    extracted_info += f'<info quarter="{result.id}">{result.text}</info>\n'

# Step 4: Synthesize final answer
final_response = opus.generate(question + extracted_info)
```

### Benefits
- **Cost Efficiency:** Haiku is cheaper than Opus for bulk operations
- **Speed:** Parallel processing reduces total time
- **Scalability:** Can process many documents simultaneously

---

## PDF to Image Conversion Best Practices

### Quality Settings
```python
def pdf_to_base64_pngs(pdf_path, quality=75, max_size=(1024, 1024)):
    # Render at 300 DPI
    pix = page.get_pixmap(matrix=fitz.Matrix(300/72, 300/72))

    # Resize if needed
    if image.size[0] > max_size[0] or image.size[1] > max_size[1]:
        image.thumbnail(max_size, Image.Resampling.LANCZOS)

    # Save with optimization
    image.save(image_data, format='PNG', optimize=True, quality=quality)
```

**Key Parameters:**
- **DPI:** 300 DPI provides good quality without excessive size
- **Max Size:** (1024, 1024) balances quality and API limits
- **Quality:** 75 is a good compromise for PNG compression
- **Format:** PNG preferred for charts/graphs, JPEG for photos

---

## Slide Deck Narration Strategy

### When to Use
1. **Exceeding PDF page limits:** Need to process >100 pages
2. **RAG Systems:** Need text-searchable content from visual slides
3. **Accessibility:** Creating narrations for vision-impaired users
4. **Text Analysis:** Enabling traditional NLP on visual content

### Prompt Structure
- **Role Assignment:** "You are the [presenter role]"
- **Completeness Requirement:** "Do not leave any details un-narrated"
- **Accessibility Context:** "Some viewers are vision-impaired"
- **Structured Output:** Use XML tags for organization
- **Detail Level:** "Use excruciating detail"

### Post-Processing
Extract narration with regex:
```python
pattern = r"<narration>(.*?)</narration>"
match = re.search(pattern, response, re.DOTALL)
narration = match.group(1)
```

**Limitation:** Takes 5-10 minutes for full slide decks.

---

## Code Generation from Visual Analysis

### Pattern: Analysis + Code Generation
**Prompt Structure:**
```
Based on [extracted information], please:
1. Provide a response to the question: {question}
2. Generate Python code using matplotlib to accompany your response
3. Enclose the code within <code> tags
```

### Code Extraction
```python
def extract_code(response):
    start_tag = "<code>"
    end_tag = "</code>"
    start_index = response.find(start_tag)
    end_index = response.find(end_tag)
    if start_index != -1 and end_index != -1:
        return response[start_index + len(start_tag):end_index].strip()
    return None
```

### Execution Safety
**Warning:** Using `exec()` on model-generated code is unsafe in production.

**Better Approaches:**
- Sandbox execution environments
- Code review before execution
- Whitelisted library imports only
- Restricted file system access

---

## Transcription Best Practices

### Typed Text
**Capability:** Excellent OCR-like capabilities
**Advantage:** Can selectively transcribe (e.g., "transcribe only the code")

### Handwritten Text
**Capability:** Excels at cursive and varied handwriting
**Best Practice:** Use "Only output the text and nothing else" for clean output

### Mixed Forms
**Capability:** Handles typed + handwritten content
**Prompt:** "Transcribe this form exactly" preserves structure

### Structured Output
**Pattern:** Request specific formats (JSON, CSV, tables)
**Example:** "Turn this org chart into JSON indicating who reports to who"

---

## API Best Practices

### Message Structure
```python
message_list = [
    {
        "role": "user",
        "content": [
            {"type": "image", "source": {...}},
            {"type": "text", "text": "..."}
        ]
    }
]
```

**Order:** Images typically come before text prompts, but order matters for context.

### Model Names
- `claude-3-opus-20240229`
- `claude-3-haiku-20240307`
- `claude-3-5-sonnet-20241022` (for PDFs)

### Token Limits
- Set `max_tokens` appropriately (2048-8192 typical)
- Longer documents may need higher limits

### Temperature
- Use `temperature=0` for consistent, deterministic results
- Higher temperatures for creative tasks

---

## Performance Optimization

### Concurrent Processing
```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor() as executor:
    results = list(executor.map(process_function, items))
```

**Benefits:**
- Parallel API calls
- Reduced total processing time
- Efficient for batch operations

### Cost Optimization
- Use Haiku for extraction tasks
- Reserve Opus for synthesis and complex reasoning
- Batch similar requests
- Cache results when appropriate

---

## Summary of Key Takeaways

1. **Prompt Engineering Still Matters:** Role assignment and chain-of-thought improve vision tasks
2. **Few-Shot Learning Works:** Provide visual examples for specialized tasks
3. **PDF Support is Beta:** Only Sonnet 3.5 supports PDFs with special header
4. **Multi-Image is Powerful:** Process multiple images together for better context
5. **Sub-Agents Scale Well:** Haiku for extraction, Opus for synthesis
6. **Visual Prompting is Effective:** Annotations in images can replace text prompts
7. **Arithmetic Needs Tools:** Provide calculator tools for reliable calculations
8. **Narration Enables Text Analysis:** Convert slide decks to searchable text
9. **Safety First:** Don't execute generated code without sandboxing
10. **Quality Over Speed:** 300 DPI and proper optimization balance quality and performance