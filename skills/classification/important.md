# Important Technical Details and Best Practices

## Key Insights from Classification Cookbook

### 1. When to Use LLMs for Classification

LLMs excel in classification scenarios where:
- **Complex business rules** are involved
- **Limited or low-quality training data** is available
- **Natural language explanations** are needed for classification decisions
- Traditional ML approaches have struggled

### 2. Prompt Engineering Best Practices

#### XML Structuring
- **Always use XML tags** to structure information in prompts
- XML makes it easier for Claude to parse and interpret complex data
- Categories, examples, and tickets should be wrapped in semantic XML tags
- Reference: [Claude Documentation on XML Tags](https://docs.claude.com/claude/docs/use-xml-tags)

#### Assistant Prefill Technique
- Use the `role: assistant` message to prefill responses
- Combine with `stop_sequences` for reliable output extraction
- Example: `{"role":"assistant", "content": "<category>"}` with stop sequence `</category>`
- This ensures consistent, parseable output format

#### Temperature Settings
- **Temperature 0.0** is optimal for classification tasks requiring consistency
- Higher temperatures (0.2-0.8) showed minimal improvement or slight degradation
- Stick with T=0.0 for deterministic, reproducible classification results

### 3. Retrieval-Augmented Generation (RAG) Implementation

#### Vector Database Setup
- **Embedding Model:** VoyageAI voyage-2 model
- **Batch Processing:** Embed documents in batches of 128 to respect API limits
- **Similarity Search:** Uses dot product for similarity calculation
- **Caching:** Implement query embedding cache to reduce API calls

#### Key Parameters
```python
# Similarity threshold: Only include examples above this threshold
similarity_threshold = 0.75  # notebook version
similarity_threshold = 0.85  # evaluation version (more strict)

# Number of examples: 5 similar examples (k=5)
# Provides sufficient context without overwhelming the prompt
k = 5
```

#### Why RAG Works
- Provides **relevant context** from training data
- Enables **few-shot learning** without manual example selection
- Automatically adapts examples based on query similarity
- Improves accuracy from ~70% to ~94%

### 4. Chain-of-Thought (CoT) Reasoning

#### Implementation
```xml
<response>
    <scratchpad>Your thoughts and analysis go here</scratchpad>
    <category>The category label you chose goes here</category>
</response>
```

#### Benefits
- Forces Claude to **explicitly reason** before deciding
- Helps with **edge cases** and similar categories
- Provides **explainability** for classification decisions
- Improves accuracy to ~95.6% (additional 1-2% over RAG alone)

#### When to Use CoT
- Complex classification with overlapping categories
- When explanations are needed for audit/compliance
- Cases requiring nuanced decision-making
- Trade-off: Slightly more tokens, but better accuracy

### 5. Evaluation Methodology

#### Empirical Testing is Critical
- **Never assume** - always measure prompt performance empirically
- Test multiple variations systematically
- Use proper train/test splits (68 training, 68 test examples in this case)

#### Evaluation Metrics
- **Classification report** with precision, recall, F1 per class
- **Confusion matrix** to identify which categories are confused
- **Concurrent testing** with ThreadPoolExecutor for faster evaluation

#### Tools
- **Promptfoo**: Open-source LLM evaluation framework
- Supports multiple prompts and providers simultaneously
- Enables systematic A/B testing of prompt variations
- Can test different temperature settings in parallel

### 6. API Rate Limit Considerations

#### Rate Limits by Tier
- Adjust `MAXIMUM_CONCURRENT_REQUESTS` based on your API tier
- Reference: [Claude API Rate Limits](https://docs.claude.com/claude/reference/rate-limits)
- Default in code: 1 concurrent request (safe for all tiers)
- Can increase for higher tiers to speed up evaluation

#### Batch Processing Strategy
```python
# Process in batches using ThreadPoolExecutor
batch_size = MAXIMUM_CONCURRENT_REQUESTS
with concurrent.futures.ThreadPoolExecutor() as executor:
    futures = [executor.submit(classifier, x) for x in batch_X]
```

### 7. Data Preparation

#### Format Requirements
- TSV (Tab-Separated Values) for training and test data
- Two columns: `text` (the query) and `label` (the category)
- Normalize labels: Strip whitespace for consistent comparison

#### Training Data Size
- 68 labeled examples for training
- Sufficient for RAG-based approach
- LLMs require much less training data than traditional ML

### 8. Response Extraction

#### Transform Function
```python
def get_transform(output, context):
    try:
        return output.split("<category>")[1].split("</category>")[0].strip()
    except Exception as e:
        return output  # Fallback to raw output
```

#### Error Handling
- Always include try-except blocks
- Provide fallback to raw output if parsing fails
- Log errors for debugging

### 9. Model Selection

#### Claude 3 Haiku
- Used in this cookbook for cost-effectiveness
- Sufficient for well-structured classification tasks
- Fast inference times
- Good balance of performance and cost

#### When to Upgrade
- Consider Sonnet/Opus if Haiku accuracy is insufficient
- For more complex reasoning tasks
- When explanations need to be more detailed

### 10. Testing Assertions (Promptfoo)

#### Validation Strategy
```yaml
assert:
  - type: icontains-any
    value:
      - 'Billing Inquiries'
      - 'Policy Administration'
      # ... all valid categories
```

- Ensures output contains one of the valid categories
- Case-insensitive matching (`icontains`)
- Prevents hallucinated or malformed categories

### 11. Optimization Path

**Progression from simple to advanced:**

1. **Start Simple:** Zero-shot classification (~70% accuracy)
2. **Add RAG:** Few-shot with similar examples (~94% accuracy)
3. **Add CoT:** Reasoning before classification (~96% accuracy)

**Key Takeaway:** Each enhancement provides incremental improvement. Start simple and add complexity only when needed.

### 12. Reproducibility

#### Version Control
- Pin model versions: `claude-3-haiku-20240307`
- Document all configuration parameters
- Save vector databases to disk (pickle format)
- Cache embeddings to ensure consistency

#### Determinism
- Use `temperature=0.0` for reproducible results
- Set random seeds where applicable
- Document all hyperparameters

### 13. Cost Optimization

#### Strategies
- Use Haiku instead of Sonnet/Opus when possible
- Cache embeddings in pickle file to avoid re-computation
- Implement query embedding cache for repeated queries
- Process in batches to minimize overhead

#### Trade-offs
- Simple classification: Lowest cost, lower accuracy
- RAG: Moderate cost (embeddings + LLM), good accuracy
- RAG + CoT: Higher cost (more tokens), best accuracy

### 14. Production Considerations

#### Monitoring
- Track classification accuracy over time
- Monitor API latency and errors
- Log misclassifications for analysis

#### Continuous Improvement
- Regularly update training data with new examples
- Retrain embeddings when training data changes
- A/B test prompt variations in production
- Use feedback loops to improve category definitions

#### Scalability
- Vector database can be replaced with production solutions (Pinecone, Weaviate, etc.)
- Consider caching frequently classified queries
- Implement proper error handling and retries
- Monitor token usage and costs

### 15. Documentation References

- [Claude Prompt Engineering Guide](https://docs.claude.com/en/docs/prompt-engineering)
- [Claude API Rate Limits](https://docs.claude.com/claude/reference/rate-limits)
- [VoyageAI Embeddings](https://docs.claude.com/en/docs/embeddings)
- [Promptfoo Documentation](https://www.promptfoo.dev/docs/getting-started)

---

## Quick Reference: Key Numbers

| Metric | Value |
|--------|-------|
| Training Examples | 68 |
| Test Examples | 68 |
| Number of Categories | 10 |
| RAG Examples (k) | 5 |
| Similarity Threshold | 0.75-0.85 |
| Batch Size (embeddings) | 128 |
| Best Temperature | 0.0 |
| Best Accuracy | 95.59% |
| Model | claude-3-haiku-20240307 |
| Embedding Model | voyage-2 |
| Max Tokens | 4096 |