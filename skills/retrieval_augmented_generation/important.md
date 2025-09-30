# Important Technical Details & Best Practices

This document captures key technical details, best practices, and important considerations from the RAG cookbook.

---

## Performance Improvements Summary

Through three levels of optimization, the RAG system achieved significant performance gains:

| Metric | Basic RAG | Summary Indexing | Summary Indexing + Re-Ranking |
|--------|-----------|------------------|-------------------------------|
| Avg Precision | 0.43 | 0.43 | 0.44 |
| Avg Recall | 0.66 | 0.66 | 0.69 |
| Avg F1 Score | 0.52 | 0.52 | 0.54 |
| Avg MRR | 0.74 | ~0.74 | 0.87 |
| End-to-End Accuracy | 71% | 78% | 81% |

**Key Insight:** The most dramatic improvement came from re-ranking (MRR: 0.74 → 0.87), showing that retrieval quality matters more than just quantity.

---

## Evaluation Methodology

### Critical Principle: Separate Retrieval and End-to-End Evaluation

**Always evaluate the retrieval system and end-to-end system separately.** This allows you to:
1. Identify whether problems are in retrieval or generation
2. Optimize each component independently
3. Understand the true bottlenecks in your pipeline

### Evaluation Metrics Explained

#### Retrieval Metrics

**Precision**
- Formula: True Positives / Total Retrieved
- Answers: "Of the chunks we retrieved, how many were correct?"
- High precision = efficient system with few false positives
- Important note: Minimum retrieval of 3 chunks may affect precision scores

**Recall**
- Formula: True Positives / Total Correct
- Answers: "Of all the correct chunks that exist, how many did we retrieve?"
- High recall = comprehensive coverage of necessary information
- Critical for ensuring LLM has all needed information

**F1 Score**
- Formula: 2 × (Precision × Recall) / (Precision + Recall)
- Harmonic mean of precision and recall
- Range: 0 to 1 (1 = perfect)
- Useful when both false positives and false negatives matter

**Mean Reciprocal Rank (MRR)**
- Formula: (1/|Q|) × Σ(1/rank_i)
- Measures how well the system ranks relevant information
- Answers: "How quickly would a user find what they're looking for?"
- Only considers rank of first correct result
- Range: 0 to 1 (1 = correct answer always first)

#### End-to-End Metrics

**End-to-End Accuracy**
- Formula: Number of Correct Answers / Total Questions
- Uses LLM-as-judge (Claude 3.5 Sonnet) for evaluation
- Evaluates entire pipeline from retrieval to answer generation

### Balancing Precision vs Recall

- **Trade-off exists:** Often can't maximize both simultaneously
- **RAG systems typically favor recall:** LLMs can filter out less relevant information during generation
- **Minimum chunk retrieval:** Setting k=3 minimum favors recall over precision
- **Optimal balance:** Depends on your specific use case

---

## RAG Architecture Patterns

### Level 1: Basic RAG (Naive RAG)

**Pipeline:**
1. Chunk documents by heading
2. Embed each document using Voyage AI
3. Use cosine similarity to retrieve top k documents
4. Provide retrieved documents to Claude for answer generation

**Embedding Strategy:**
- Combine heading and text: `"Heading: {heading}\n\nChunk Text: {text}"`
- Model: voyage-2
- Batch size: 128 documents at a time

**Pros:**
- Simple to implement
- Fast retrieval
- Good baseline performance

**Cons:**
- May retrieve less relevant chunks
- Embeddings don't capture high-level document purpose
- No ranking optimization

### Level 2: Summary Indexing

**Key Innovation:** Generate 2-3 sentence summaries for each chunk and embed them alongside original content.

**Embedding Strategy:**
- Combine heading, text, AND summary: `"{heading}\n\n{text}\n\n{summary}"`
- Summaries capture essence of each chunk more effectively

**Context Provided to LLM:**
```xml
<document>
{chunk_heading}

Text
{chunk_text}

Summary:
{chunk_summary}
</document>
```

**Benefits:**
- More informative embeddings
- Better semantic matching
- LLM receives both overview (summary) and details (full text)
- Improved end-to-end accuracy: 71% → 78%

**Implementation Note:** Summaries are generated once and cached, so there's minimal runtime overhead.

### Level 3: Summary Indexing + Re-Ranking

**Key Innovation:** Two-stage retrieval - cast wide net (k=20), then focus (k=3).

**Pipeline:**
1. Initial retrieval: Get top 20 documents from vector DB
2. Re-ranking: Use Claude to select 3 most relevant from the 20
3. Answer generation: Use only the top 3 re-ranked documents

**Why This Works:**
- **Wider initial net:** Ensures relevant documents aren't missed in initial retrieval
- **AI-powered focusing:** Claude understands nuances and context better than cosine similarity alone
- **Improved ranking:** MRR jumped from 0.74 to 0.87

**Performance Impact:**
- Precision improved: Re-ranker reduces number of irrelevant documents shown to LLM
- MRR dramatically improved: Better ranking of most relevant documents
- End-to-end accuracy: 78% → 81%

**Trade-off:** Additional API call for re-ranking adds latency and cost, but significantly improves quality.

---

## Vector Database Implementation

### Design Patterns

**Caching Strategy:**
```python
self.query_cache = {}  # Cache query embeddings
if query in self.query_cache:
    query_embedding = self.query_cache[query]
else:
    query_embedding = self.client.embed([query], model="voyage-2").embeddings[0]
    self.query_cache[query] = query_embedding
```

**Persistence:**
- Save embeddings, metadata, and query cache to disk using pickle
- Check for existing database before re-embedding
- Significantly speeds up repeated evaluations

**Similarity Threshold:**
```python
def search(self, query, k=5, similarity_threshold=0.75):
    # Only return documents above similarity threshold
    # AND limit to top k results
```

### Batch Embedding Best Practice

```python
batch_size = 128
result = [
    self.client.embed(
        texts[i : i + batch_size],
        model="voyage-2"
    ).embeddings
    for i in range(0, len(texts), batch_size)
]
```

**Why:** Embedding APIs often have batch limits. Processing in batches of 128 balances efficiency with API constraints.

---

## Prompt Engineering Best Practices

### 1. Use XML Tags for Structure

```xml
<query>{query}</query>
<documents>{context}</documents>
```

**Benefit:** Clear delineation helps Claude parse and respond to structured information.

### 2. Prefill Assistant Responses

```python
messages=[
    {"role": "user", "content": prompt},
    {"role": "assistant", "content": "<relevant_indices>"}
],
stop_sequences=["</relevant_indices>"]
```

**Benefit:** Ensures consistent, parseable output format. Reduces likelihood of formatting errors.

### 3. Emphasize Faithfulness to Context

```
Please remain faithful to the underlying context, and only deviate from it
if you are 100% sure that you know the answer already.
```

**Benefit:** Reduces hallucination and grounds responses in retrieved documents.

### 4. Eliminate Unnecessary Preamble

```
Answer the question now, and avoid providing preamble such as
'Here is the answer', etc
```

**Benefit:** More concise, direct responses. Better user experience.

### 5. Use Temperature 0 for Consistency

All prompts use `temperature=0` for deterministic, reproducible results. This is especially important for:
- Summarization (consistent summaries)
- Evaluation (reliable scoring)
- Re-ranking (stable rankings)

---

## Model Selection Strategy

### Task-Specific Model Choices

**Summarization & Re-Ranking:**
- Model: claude-3-haiku-20240307
- Rationale: Fast, cost-effective, sufficient for focused tasks
- Max tokens: 50-150 (concise outputs)

**Answer Generation:**
- Model: claude-3-haiku-20240307 (baseline)
- Alternative: claude-3-5-sonnet-20241022 (for better accuracy)
- Evaluation shows: 3.5 Sonnet achieved 85% vs Haiku's 78% end-to-end accuracy

**Evaluation (LLM-as-Judge):**
- Model: claude-3-5-sonnet-20241022
- Rationale: More sophisticated reasoning needed for nuanced evaluation
- Max tokens: 1500 (detailed explanations)

### Key Insight from Evaluation

From the Promptfoo comparison:
- **Haiku (T=0.0):** 78% accuracy
- **3.5 Sonnet (T=0.0):** 85% accuracy

**Takeaway:** Choose the right model for speed vs quality requirements. Haiku is excellent for prototyping and cost-sensitive applications, while Sonnet provides best-in-class accuracy.

---

## Evaluation Dataset Design

### Dataset Characteristics

- **Size:** 100 samples
- **Difficulty:** Relatively challenging
- **Multi-hop queries:** Some questions require synthesizing information from multiple chunks
- **Implication:** System must retrieve more than one chunk at a time

### Dataset Schema

```json
{
  "id": "unique_id",
  "question": "The user's question",
  "correct_chunks": ["url1", "url2", ...],  // Expected retrievals
  "correct_answer": "Ground truth answer"
}
```

### Evaluation Data Split

1. **retrieval_dataset.csv:** For evaluating retrieval pipeline in isolation
2. **end_to_end_dataset.csv:** For evaluating full RAG pipeline
3. **__expected column:** Allows automatic assertions for each row

---

## Error Handling Patterns

### Re-Ranking Fallback

```python
try:
    # Parse Claude's re-ranking response
    relevant_indices = [int(idx.strip()) for idx in indices_str.split(',')]

    # Fallback if no valid indices
    if len(relevant_indices) == 0:
        relevant_indices = list(range(min(k, len(results))))

    # Ensure no out-of-range indices
    relevant_indices = [idx for idx in relevant_indices if idx < len(results)]

except Exception as e:
    print(f"An error occurred during reranking: {str(e)}")
    return results[:k]  # Fall back to top k without re-ranking
```

**Principle:** Always have graceful degradation. If AI-powered re-ranking fails, fall back to simpler cosine similarity ranking.

### Evaluation Error Handling

```python
try:
    # Parse XML evaluation response
    is_correct = evaluation.find('is_correct').text.lower() == 'true'
except ET.ParseError as e:
    logging.error(f"XML parsing error: {e}")
    is_correct = 'true' in response_text.lower()  # Fallback parsing
except Exception as e:
    logging.error(f"Unexpected error: {e}")
    results.append(False)  # Conservative: mark as incorrect
```

**Principle:** When evaluation fails, fail conservatively (mark as incorrect) to avoid false positives in metrics.

---

## Production Considerations

### Rate Limits

- **Warning:** Full evaluation may hit rate limits unless in Tier 2+
- **Mitigation:** Consider sampling evaluation dataset for lower tiers
- **Alternative:** Use Promptfoo to run evaluations incrementally

### In-Memory vs Hosted Vector DB

- **Cookbook uses:** In-memory vector DB (simple, self-contained)
- **Production recommendation:** Use hosted solution (Pinecone, Weaviate, etc.)
- **Reasons:** Scalability, persistence, multi-user support

### Caching Strategy

The cookbook implements several caching layers:
1. **Query embedding cache:** Avoids re-embedding same queries
2. **Vector DB persistence:** Saves to disk to avoid re-embedding documents
3. **Summarization:** Summaries generated once and stored

**Production Extension:** Consider adding:
- Redis cache for query results
- CDN for frequently accessed documents
- Embedding cache with TTL for dynamic documents

---

## Integration with Promptfoo

### Why Use Promptfoo?

1. **Scale beyond Jupyter:** Handle larger datasets and more prompts
2. **Compare systematically:** Test multiple models, hyperparameters, prompts side-by-side
3. **Automated testing:** Build CI/CD pipelines for prompt evaluation
4. **Rich visualization:** Web UI for exploring results

### Custom Providers

For RAG, standard LLM providers aren't sufficient. The cookbook implements custom providers:

```python
# provider_retrieval.py - Custom retrieval provider
def answer_query_base(context):
    input_query = context['vars']['query']
    documents, document_context = _retrieve_base(input_query, db)
    # Return prompt instead of calling API
    return prompt  # Promptfoo handles API calls
```

**Benefit:** Reuse VectorDB class while letting Promptfoo orchestrate evaluations.

### Custom Assertions

```python
# eval_retrieval.py - Custom assertion for retrieval metrics
def get_assert(output: str, context) -> Union[bool, float, Dict[str, Any]]:
    precision, recall, mrr, f1 = evaluate_retrieval(output, correct_chunks)
    return {
        "pass": f1 > 0.3,
        "score": f1,
        "componentResults": [...]  # Individual metric results
    }
```

**Benefit:** Track multiple metrics simultaneously (precision, recall, F1, MRR) in a single evaluation run.

---

## Empirical Prompt Engineering Philosophy

### Core Principle: Measure, Don't Guess

From the cookbook:
> "This guide has illustrated the importance of measuring prompt performance empirically when prompt engineering."

### Methodology

1. **Start with baseline:** Implement simplest version first
2. **Measure systematically:** Use consistent evaluation dataset
3. **Iterate with data:** Make changes based on metrics, not intuition
4. **Track multiple metrics:** Single metric can be misleading
5. **Use proper tooling:** Graduate from notebooks to evaluation frameworks

### Anti-Pattern: "Vibes-Based" Evaluation

The cookbook explicitly warns against relying on subjective feelings about performance:
> "We'll go beyond 'vibes' based evals and show you how to measure the retrieval pipeline & end to end performance independently."

**Instead:** Use quantitative metrics and LLM-as-judge for consistent, reproducible evaluation.

---

## Document Chunking Strategy

### Strategy Used: Chunk by Heading

```python
texts = [f"Heading: {item['chunk_heading']}\n\nChunk Text:{item['text']}"
         for item in data]
```

**Rationale:**
- Headings provide natural semantic boundaries
- Each chunk has coherent topic/scope
- Heading adds important context to chunk

**Alternative Strategies Not Used:**
- Fixed token windows (may break semantic units)
- Sentence-based chunking (may be too granular)
- Full document embedding (loses granularity)

---

## Knowledge Base Context Pattern

When generating summaries, provide context about the overall knowledge base:

```python
knowledge_base_context = """This is documentation for Anthropic's, a frontier AI lab
building Claude, an LLM that excels at a variety of general purpose tasks. These docs
contain model details and documentation on Anthropic's APIs."""
```

**Benefit:** Helps Claude generate summaries that are appropriately scoped and contextualized for the domain.

---

## Key Takeaways

1. **Separate evaluation is critical:** Always measure retrieval and end-to-end performance independently
2. **Two-stage retrieval wins:** Cast wide net, then focus with AI-powered re-ranking
3. **Summaries improve embeddings:** Small investment in summarization yields big gains
4. **Empirical > Intuition:** Measure everything, optimize based on data
5. **Right tool for right job:** Use Haiku for speed, Sonnet for quality
6. **Graceful degradation:** Always have fallbacks for AI-powered components
7. **Structure matters:** XML tags, prefilled responses, and explicit formats reduce errors
8. **Temperature 0 for consistency:** Deterministic outputs are crucial for reliable systems
9. **Promptfoo scales evaluation:** Graduate from notebooks to proper evaluation frameworks
10. **MRR matters most:** Ranking quality often more important than retrieval quantity