# Important Technical Details and Best Practices

This document captures key technical details, best practices, and important considerations discovered in the summarization cookbook.

---

## Core Technical Patterns

### 1. Assistant Preamble and Stop Sequences

**Pattern:**
- Use assistant role with content like `"Here is the summary of the legal document: <summary>"`
- Set stop sequence to `</summary>`

**Purpose:**
This pattern constrains Claude's output by teeing it up to provide the summary directly after the final phrase. The stop sequence tells Claude when to stop generating. This is a fundamental pattern used throughout the guide for controlled output generation.

**Example:**
```python
messages=[
    {"role": "user", "content": prompt},
    {"role": "assistant", "content": "Here is the summary of the legal document: <summary>"}
],
stop_sequences=["</summary>"]
```

---

## Prompt Engineering Techniques

### 2. "Do Not Preamble" Instruction

**Best Practice:**
Add "Do not preamble" to prompts when you want just the answer without conversational framing.

**When to Use:**
- Particularly important when not using other constraint instructions
- When you need clean, parseable output
- When constraining model output to avoid chatty responses

### 3. Few-Shot (Multi-Shot) Learning

**Technique:**
Include 2-3 example document-summary pairs in the prompt to demonstrate desired format and style.

**Benefits:**
- Claude naturally picks up on formatting patterns without explicit instructions
- Improves output consistency and structure
- Can replace or augment explicit formatting instructions

**Key Finding:**
The model generalizes from examples to new inputs, even without being explicitly told to follow the example format.

---

## Document Processing

### 4. Text Cleaning and Preparation

**Steps:**
1. Extract text from PDFs using pypdf
2. Clean text: remove extra whitespace, page numbers
3. Truncate to fit token limits (approximate 4 characters per token)

**Code Pattern:**
```python
text = re.sub(r'\s+', ' ', text)  # Remove extra whitespace
text = re.sub(r'\n\s*\d+\s*\n', '\n', text)  # Remove page numbers
text = text[:max_tokens * 4]  # Approximate token limiting
```

### 5. Chunking for Long Documents

**Strategy:**
- Break documents into chunks (e.g., 2000 characters)
- Process each chunk with a lightweight model (Haiku) for cost efficiency
- Combine chunk summaries with a more powerful model (Sonnet)
- Limit initial chunk content to ~2000 words for ranking purposes

**Benefits:**
- Handles documents beyond typical token limits
- Balances cost, speed, and quality
- Enables processing of very large document sets

---

## Structured Output

### 6. XML Tags for Parsing

**Pattern:**
Use XML tags to structure output sections for easy programmatic parsing:
```
<parties involved>
- Sublessor: [Name]
- Sublessee: [Name]
</parties involved>
```

**Parsing:**
```python
pattern = r'<(.*?)>(.*?)</\1>'
matches = re.findall(pattern, text, re.DOTALL)
```

**Alternatives:**
- JSON format (also mentioned as viable option)
- Custom delimiters

### 7. Handling Missing Information

**Best Practice:**
Include instruction: "If any information is not explicitly stated in the document, note it as 'Not specified'."

**Purpose:**
Prevents hallucination and clearly indicates data gaps in source documents.

---

## Model Selection

### 8. Strategic Model Usage

**Pattern:**
- **Claude 3.5 Sonnet**: Final summaries, complex analysis, high-stakes evaluations
- **Claude 3 Haiku**: Initial chunk summaries, document ranking, high-volume tasks

**Rationale:**
- Haiku is faster and cheaper for preliminary tasks
- Sonnet provides better quality for final outputs
- Balance cost vs. quality based on task criticality

**Code Pattern:**
```python
def function(text, model="claude-3-5-sonnet-20241022", max_tokens=1000):
    # Makes model selection flexible
```

---

## Advanced RAG: Summary Indexed Documents

### 9. Summary Indexed Documents Approach

**How It Works:**
1. Generate summaries at document ingestion time
2. Fit all summaries in context window
3. Rank summaries by relevance to query (using Haiku)
4. Apply optional reranking for top-K results
5. Extract relevant clauses from top documents

**Advantages:**
- More efficient than traditional RAG (less context needed)
- Superior performance for document-level retrieval
- Consistently ranks correct document first
- Optimizes information retrieval through reranking

**Best Practices for Summary RAG:**
- Experiment with summary lengths for optimal balance
- Consider multiple rounds of reranking for large datasets
- Implement caching for summaries and rankings
- Use lightweight model for initial ranking

---

## Evaluation Framework

### 10. Multi-Method Evaluation

**Challenge:**
Summarization evaluation is notoriously difficult and subjective. No single metric captures all quality aspects.

**Recommended Approach:**
Combine multiple evaluation methods:
1. **ROUGE scores**: Measures n-gram overlap with reference summaries
2. **BLEU scores**: Measures similarity to reference texts
3. **LLM-as-judge**: Evaluates nuanced aspects (coherence, accuracy, relevance)
4. **Contains method**: Checks for specific required content

**Key Insight:**
The most effective approach uses a tailored combination of techniques suited to the specific summarization task.

### 11. Promptfoo for Systematic Evaluation

**Tool:** [Promptfoo](https://www.promptfoo.dev/) - open source LLM evaluation toolkit

**Benefits:**
- Scales beyond Jupyter notebooks as datasets grow
- Supports multiple models and parameters
- Custom evaluation functions
- Various output formats
- Dynamic result viewing

**Setup:**
- Define prompts, providers, tests in `promptfooconfig.yaml`
- Custom evaluations: `bleu_eval.py`, `rouge_eval.py`, `llm_eval.py`
- Run: `npx promptfoo@latest eval -c promptfooconfig.yaml`
- View: `npx promptfoo@latest view`

### 12. LLM-Based Evaluation Criteria

**Dimensions:**
1. **Conciseness** (1-5): Brevity without losing key information
2. **Accuracy** (1-5): Correctness of information
3. **Completeness** (1-5): Coverage of important details
4. **Clarity** (1-5): Readability and understandability

**Key Questions:**
- Does summary capture key provisions accurately?
- Are important details omitted?
- Any inaccuracies or misrepresentations?
- Fair representation vs. undue emphasis?
- Accurate reflection of language and tone?
- Key concepts and principles captured?

---

## System Messages

### 13. Domain-Specific System Messages

**Generic:**
```
You are a legal analyst known for highly accurate and detailed summaries of legal documents.
```

**Specialized:**
```
You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.
```

**Meta-summarization:**
```
You are a legal expert that summarizes notes on one document.
```

**Impact:**
Domain-specific system messages improve accuracy and relevance for specialized document types.

---

## Iterative Improvement Process

### 14. Systematic Refinement

**Steps:**
1. Analyze evaluation results to identify strengths and weaknesses
2. Refine prompts to address specific issues (e.g., conciseness, completeness)
3. Experiment with different chunking strategies
4. Fine-tune parameters (temperature, max_tokens)
5. Implement post-processing steps
6. Re-evaluate and iterate

**Example Finding from Guide:**
The "contains" eval was failing frequently because documents didn't always contain information required in XML tags. Solution: Refine eval criteria or adjust prompt to handle missing data better.

---

## Key Performance Parameters

### 15. Model Parameters

**Temperature:**
- Use `temperature=0` or `temperature=0.2` for consistent, deterministic summaries
- Higher temperature for creative summaries (less common use case)

**Max Tokens:**
- Initial summaries: 500-1000 tokens
- Final summaries: 1000-2000 tokens
- Evaluations: 1000 tokens
- Rankings: 2 tokens (just the number)

**Context Length:**
- Approximate 4 characters per token for planning
- Max context: 180,000 tokens (used in example)

---

## Best Practices Summary

### 16. Conclusive Tips for Optimization

1. **Craft clear and specific prompts** - Use constraints like "don't preamble"
2. **Use at least 2-3 examples** - Few-shot learning improves consistency
3. **Use guided summarization** - Domain-specific frameworks improve quality
4. **Implement effective strategies for long documents** - Chunking + meta-summarization
5. **Regularly evaluate and refine** - Systematic iteration is key
6. **Consider ethical implications** - Understand limitations of AI-generated summaries
7. **Benchmark appropriately** - Compare against human performance, not 100% accuracy
8. **Leverage speed and efficiency** - Focus human effort on decision-making, not summarization

---

## Important Philosophical Notes

### 17. Evaluation Philosophy

**Key Insight:**
"You aren't benchmarking your results against 100% accuracy. You're benchmarking against how well you could perform this complex task yourself."

**Value Proposition:**
With Claude's speed and efficiency, you can spend time on real decision-making rather than tedious summarization work.

### 18. Subjectivity in Summarization

**Challenges:**
- Unlike many NLP tasks, summarization lacks clear-cut objective metrics
- Highly subjective - different readers value different aspects
- Traditional metrics (ROUGE) have limitations in capturing nuance
- "Best" summary varies by use case, audience, and desired detail level

**Solution:**
Combine automated metrics, regular expressions, and task-specific criteria tailored to your particular use case.

---

## Package Dependencies

### 19. Required Packages

**Core:**
- anthropic
- pypdf
- pandas
- matplotlib
- sklearn
- numpy
- rouge-score
- nltk
- seaborn

**Evaluation:**
- promptfoo (requires node.js and npm)

**Installation:**
```bash
pip install anthropic pypdf pandas matplotlib numpy rouge-score nltk seaborn
```

**NLTK Data:**
```python
nltk.download('punkt', quiet=True)
```

---

## Data Sources

### 20. Example Data

**Source:**
Publicly available Sublease Agreement from [sec.gov website](https://www.sec.gov/Archives/edgar/data/1045425/000119312507044370/dex1032.htm)

**Structure:**
- `data/`: Contains sample lease agreements and their summaries
- Multiple example documents for testing and evaluation
- Ground truth summaries for comparison

**Flexibility:**
The guide supports custom PDFs or direct text input via copy-paste.