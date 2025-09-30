# Important Technical Details and Best Practices

This document captures key technical details, best practices, and important considerations discovered across the third-party integration cookbook examples.

---

## General Best Practices

### Prompt Engineering for Tool Use

1. **XML Tag Format for Tool Calls (Anthropic-Specific)**
   - Anthropic models are specifically trained to use XML tags for tool usage
   - Format: `<tool>{tool name}</tool><tool_input>{tool input}</tool_input>`
   - This differs from standard ReAct format and works better with Claude models
   - XML agent prompts are specifically designed for Anthropic models

2. **Scratchpad Pattern for Chain-of-Thought**
   - Use `<scratchpad>` tags to encourage Claude to plan before executing
   - Helps with complex multi-step queries
   - Improves reasoning quality and reduces premature answers

3. **Reflection and Self-Assessment**
   - Include prompts for Claude to reflect on search quality
   - Use `<search_quality>` tags to assess if more information is needed
   - Prevents jumping to conclusions with incomplete information

4. **Information Extraction Before Answering**
   - Separate information gathering from answer synthesis
   - Extract information in `<information>` tags without answering
   - Then provide information to Claude in a new query for final answer
   - This prevents Claude from precommitting to an answer and "justifying" it with search results

---

## RAG (Retrieval-Augmented Generation) Architecture

### Vector Database Configuration

1. **Pinecone Setup**
   - Use serverless spec for deployment flexibility
   - Important: Match embedding dimensions with index configuration
   - VoyageAI voyage-2 model = 1024 dimensions
   - VoyageAI voyage-code-2 model = 1536 dimensions
   - Use `dotproduct` or `cosine` similarity metrics
   - Create vector search index before data ingestion

2. **MongoDB Atlas Vector Search**
   - Create vector search index with specific configuration
   - Index definition must match embedding dimensions (e.g., 1536 for voyage-large-2)
   - Use cosine similarity for distance metric
   - Whitelist IP addresses (or use 0.0.0.0/0 for PoCs)
   - Index name matters: reference it correctly in aggregation pipelines

3. **Embedding Best Practices**
   - Use `input_type="document"` when embedding documents for indexing
   - Use `input_type="query"` when embedding user queries
   - This improves retrieval quality in Voyage AI models
   - VoyageAI embeddings are normalized to length 1 (dot product = cosine similarity)

### Search Optimization

1. **Query Transformation**
   - Use Claude to generate diverse keywords from natural language queries
   - Multiple keyword searches provide better coverage than single query
   - Aggregate results from multiple searches for comprehensive answers

2. **Wikipedia Search Specifics**
   - Wikipedia search requires atomic keyword queries, not full phrases
   - "basketball" works better than "what is the color of a basketball"
   - Google-style natural language queries don't work well
   - Keep queries short: "keywords, not phrases"

3. **Result Formatting**
   - Use structured XML format for search results
   - Format: `<search_results><item index="1"><page_content>...</page_content></item></search_results>`
   - This format is well-understood by Claude models

4. **Top-k Selection**
   - Default to 3-5 results for most queries
   - Adjust based on query complexity and available context window
   - More results = more context but more tokens consumed

---

## LlamaIndex Integration

### Model Configuration

1. **Settings Pattern**
   - Use global Settings for LLM and embedding model configuration
   ```python
   Settings.llm = llm
   Settings.embed_model = embed_model
   Settings.chunk_size = 512
   ```
   - Consistent chunk size (512 tokens) works well for most use cases

2. **Temperature Settings**
   - Use `temperature=0.0` for deterministic, factual responses
   - Higher temperatures for creative tasks
   - Recommended: 0.0 for RAG, 0.5 for question generation

### Agent Architecture

1. **ReAct Agent Best Practices**
   - Always start with "Thought" in agent responses
   - Use valid JSON for tool inputs
   - Stop sequences are critical: `["</tool_input>", "</final_answer>"]`
   - Include verbose logging for debugging

2. **Multi-Document Agents**
   - Create specialized agents for each document/domain
   - Use IndexNode to link agents with descriptive summaries
   - Top-level retriever selects appropriate agent based on query
   - Each agent can have multiple tools (vector + summary)

3. **Query Engine Selection**
   - **VectorStoreIndex**: For specific fact retrieval
   - **SummaryIndex**: For summarization and overview questions
   - **RouterQueryEngine**: Routes between multiple query engines
   - **SubQuestionQueryEngine**: Decomposes complex queries into sub-questions

4. **Tool Descriptions**
   - Be specific and descriptive in tool metadata
   - Include when to use the tool and what type of questions it answers
   - Clear descriptions improve Claude's tool selection accuracy

---

## API Keys and Authentication

### Required API Keys

1. **Anthropic (Claude)**
   - Get from: https://console.anthropic.com/settings/keys
   - Set as: `ANTHROPIC_API_KEY` environment variable

2. **Pinecone**
   - Get from: https://app.pinecone.io
   - Free tier available
   - Set as: `PINECONE_API_KEY`

3. **Voyage AI**
   - Get from: https://dash.voyageai.com/api-keys
   - Set as: `VOYAGE_API_KEY`
   - Note: Rate limits apply (300 requests/minute)

4. **MongoDB**
   - Atlas URI from cluster connection settings
   - Set as: `MONGO_URI`
   - Format: `mongodb+srv://...`

5. **Deepgram**
   - Get from: https://dpgr.am/prerecorded-notebook-signup
   - Set as: `DG_KEY`

6. **WolframAlpha**
   - Get from: https://developer.wolframalpha.com/access
   - Free tier available
   - Set as: `WOLFRAM_APP_ID`

---

## Performance and Limitations

### Rate Limits

1. **VoyageAI**
   - 300 requests per minute
   - Use batch processing when possible
   - Implement retry logic with delays for rate limit errors

2. **Anthropic**
   - Varies by plan
   - Monitor token usage
   - Use appropriate max_tokens settings (1000-4096 typical)

### Token Management

1. **Context Windows**
   - Claude 3 Opus: 200K tokens
   - Claude 3 Sonnet: 200K tokens
   - Claude 3 Haiku: 200K tokens
   - Chunk documents to fit within limits

2. **Token Counting**
   - Use tokenizer to estimate costs before API calls
   - Voyage AI provides token counting: `vo.count_tokens(["Sample text"])`
   - Truncate long documents if needed

---

## Data Processing

### Document Preparation

1. **Chunking Strategy**
   - 512 tokens per chunk is a good default
   - Maintain semantic coherence within chunks
   - Use SimpleDirectoryReader for basic loading
   - For PDFs: LlamaIndex handles extraction automatically

2. **Metadata Management**
   - Include relevant metadata (title, source, date, etc.)
   - Metadata helps with result filtering and citation
   - Don't embed metadata that's not useful for retrieval

3. **Data Cleaning**
   - Remove unnecessary columns before ingestion
   - Convert numpy arrays to Python lists for MongoDB
   - Validate data types match database schemas

### Embedding Generation

1. **Batch Processing**
   - Process 100-128 documents at a time
   - Reduces API calls and improves efficiency
   - Implement progress bars (tqdm) for long operations

2. **Error Handling**
   - Implement try-except blocks for API failures
   - Use retry logic with exponential backoff
   - Handle empty text gracefully

---

## Multi-Modal Capabilities

### Image Understanding

1. **Claude 3 Vision**
   - All Claude 3 models support vision (Opus, Sonnet, Haiku)
   - Can process images from local files or URLs
   - Max 300-4096 tokens per image analysis (adjust max_tokens)

2. **Structured Output from Images**
   - Use Pydantic schemas for structured extraction
   - MultiModalLLMCompletionProgram for schema validation
   - Works well for forms, tables, financial data

3. **Image Sources**
   - Local files: Use SimpleDirectoryReader
   - URLs: Use load_image_urls utility
   - Both methods work with same API

---

## Security Considerations

1. **API Key Management**
   - Never hardcode API keys in notebooks
   - Use environment variables or secrets management
   - Rotate keys regularly
   - Use least privilege access

2. **Data Privacy**
   - Be aware of data sent to external APIs
   - Consider on-premise solutions for sensitive data
   - Review each provider's data retention policies

3. **Input Validation**
   - Validate user queries before processing
   - Sanitize inputs to prevent injection attacks
   - Limit query length and complexity

---

## Cost Optimization

1. **Model Selection**
   - Claude 3 Haiku: Fastest, cheapest, good for simple tasks
   - Claude 3 Sonnet: Balanced performance and cost
   - Claude 3 Opus: Best quality, most expensive
   - Choose based on task complexity

2. **Caching Strategies**
   - Implement local caching for repeated queries
   - Cache embeddings to avoid regeneration
   - Use persistent vector stores to avoid reindexing

3. **Batch Operations**
   - Batch embed operations when possible
   - Aggregate similar queries
   - Use async operations for parallel processing

---

## Debugging and Monitoring

1. **Verbose Logging**
   - Enable verbose mode in agents for debugging
   - Log all API calls and responses
   - Track token usage per operation

2. **Evaluation**
   - Test with diverse queries
   - Compare outputs across model versions
   - Monitor latency and success rates

3. **Common Issues**
   - **Empty results**: Check query formulation and index configuration
   - **Slow performance**: Reduce top_k, optimize chunk size
   - **Poor quality**: Adjust temperature, improve tool descriptions
   - **Rate limits**: Implement backoff and retry logic

---

## Deployment Considerations

1. **Async Operations**
   - Use async methods for better performance
   - Required in Jupyter: `nest_asyncio.apply()`
   - Parallel tool execution improves latency

2. **Index Management**
   - Check if index exists before creating
   - Wait for index readiness before operations
   - Handle index updates and versioning

3. **Connection Pooling**
   - Reuse client connections
   - Implement connection retry logic
   - Monitor connection health

---

## Special Techniques

### Dual Intent Prompts (Safety Consideration)
- Some prompts can be used for both benign and malicious purposes
- Example: "encrypt all files in home directory" (backup vs ransomware)
- Model appropriately refuses overtly malicious requests
- Consider use case context in production applications

### False Refusals
- Overly safe models may refuse valid requests
- Test edge cases in your domain
- Can be mitigated with system prompts and context

### Conversational Memory
- Use ConversationBufferWindowMemory for stateful conversations
- Convert messages to string format for XML agents
- Keep memory window manageable (k=5 is typical)

---

## Version Compatibility

### Important Version Notes

1. **LangChain v1.x Changes**
   - Major breaking changes from 0.0.3xx to 0.1.x
   - Agent initialization patterns changed significantly
   - New LCEL (LangChain Expression Language) approach
   - Check version compatibility for notebooks

2. **LlamaIndex**
   - Examples use latest stable versions
   - API changes between major versions
   - Pin versions for production

3. **Model Versions**
   - Examples primarily use: `claude-3-opus-20240229`
   - Can substitute: `claude-3-sonnet-20240229` for speed
   - Can substitute: `claude-3-haiku-20240307` for cost
   - Update model strings as new versions release

---

## Testing Recommendations

1. **Start Small**
   - Test with limited datasets first (500-1000 documents)
   - Validate accuracy before scaling
   - Measure performance metrics

2. **Query Diversity**
   - Test with various question types
   - Include edge cases and ambiguous queries
   - Test multi-hop reasoning

3. **Comparative Analysis**
   - Compare results across different approaches
   - A/B test prompt variations
   - Benchmark against baselines