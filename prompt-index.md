# Claude Cookbooks - Prompt Index

This document provides an overview of all AI prompts extracted from the Claude Cookbooks, organized by topic and use case.

## Table of Contents

- [Classification](#classification)
- [Retrieval Augmented Generation (RAG)](#retrieval-augmented-generation-rag)
- [Summarization](#summarization)
- [Tool Use](#tool-use)
- [Third-Party Integrations](#third-party-integrations)
- [Multimodal](#multimodal)
- [Advanced Techniques](#advanced-techniques)

---

## Classification

**Location:** `skills/classification/`
**Detailed Prompts:** [classification-prompt.md](skills/classification/classification-prompt.md)
**Technical Details:** [important.md](skills/classification/important.md)

### Prompts: 3

1. **Simple Classification (Zero-Shot)** - Basic categorization without examples
2. **RAG Classification (Few-Shot)** - Enhanced with similar examples from vector database
3. **RAG + Chain-of-Thought** - Advanced with explicit reasoning step

### Key Use Case
Insurance customer support ticket classification into 10 categories with 70% → 95.6% accuracy improvement.

### Performance Highlights
- Simple: ~70% accuracy
- RAG: ~94% accuracy (+24%)
- RAG + CoT: ~95.6% accuracy (best)

---

## Retrieval Augmented Generation (RAG)

**Location:** `skills/retrieval_augmented_generation/`
**Detailed Prompts:** [rag-prompt.md](skills/retrieval_augmented_generation/rag-prompt.md)
**Technical Details:** [important.md](skills/retrieval_augmented_generation/important.md)

### Prompts: 5

1. **Document Summarization** - Generates 2-3 sentence summaries
2. **Query Answering** - Main prompt across all 3 levels
3. **Document Re-Ranking** - Selects most relevant documents
4. **End-to-End Evaluation** - LLM-as-judge for correctness
5. **Retrieval Evaluation** - Precision, recall, F1, MRR metrics

### Key Techniques
- Summary indexing for better embeddings
- Two-stage retrieval (wide → focused)
- AI-powered re-ranking
- Separate evaluation of retrieval vs end-to-end

### Performance Highlights
- End-to-End Accuracy: 71% → 81% (+10%)
- Mean Reciprocal Rank: 0.74 → 0.87 (+18%)

---

## Summarization

**Location:** `skills/summarization/`
**Detailed Prompts:** [summarization-prompt.md](skills/summarization/summarization-prompt.md)
**Technical Details:** [important.md](skills/summarization/important.md)

### Prompts: 9

1. **Basic Summarization** - Simple bullet-point baseline
2. **Multi-Shot Basic** - Enhanced with 3 examples
3. **Guided Legal** - Structured with 5 key categories
4. **Domain-Specific** - Specialized for sublease agreements
5. **Meta-Summarization (Chunks)** - Initial chunk processing
6. **Meta-Summarization (Final)** - Combines chunk summaries
7. **Summary Indexed Ranking** - Document relevance ranking
8. **Clause Extraction** - Extracts specific clauses
9. **LLM-Based Evaluation** - Quality assessment

### Key Techniques
- Few-shot learning for consistency
- Chunking + meta-summarization for long documents
- Domain-specific system messages
- XML/JSON for structured output
- Assistant preamble pattern with stop sequences

---

## Tool Use

**Location:** `tool_use/`
**Detailed Prompts:** [tool-use-prompt.md](tool_use/tool-use-prompt.md)
**Technical Details:** [important.md](tool_use/important.md)

### Prompts: 23

**Categories:**
- Customer Service Agents (3 prompts)
- Mathematical Calculations (3 prompts)
- Structured JSON Extraction (5 prompts)
- Memory & Context Management (3 prompts)
- Parallel Tool Execution (1 prompt)
- Tool Choice Control (5 prompts)
- Type Validation (1 prompt)
- Vision + Tools (1 prompt)

### Key Capabilities
- Customer information retrieval
- Order status and cancellation
- Named entity recognition
- Sentiment analysis
- Cross-conversation learning
- Batch tool patterns
- Multimodal structured extraction

### Critical Security Notes
- ⚠️ Never use `eval()` in production
- ⚠️ Validate paths to prevent traversal attacks
- ⚠️ Memory poisoning via prompt injection
- ⚠️ Never store passwords, API keys, or PII

---

## Third-Party Integrations

**Location:** `third_party/`
**Detailed Prompts:** [third-party-prompt.md](third_party/third-party-prompt.md)
**Technical Details:** [important.md](third_party/important.md)

### Prompts: 24

**By Integration:**
- **Pinecone** (3) - RAG agent, product search, recommendations
- **Wikipedia** (3) - Atomic search, planning, synthesis
- **WolframAlpha** (2) - Computational queries
- **MongoDB** (2) - VC tech analyst, investment advice
- **Deepgram** (1) - Interview question generation
- **LlamaIndex** (13) - RAG, ReAct agents, multi-document routing
- **VoyageAI** (0) - Embedding provider only

### Key Patterns
- Two-stage prompting: extraction → synthesis
- XML format for Anthropic tool use
- Reflection loops for quality assessment
- Agent hierarchies with specialized routers
- Query decomposition for complex questions

---

## Multimodal

**Location:** `multimodal/`
**Detailed Prompts:** [multimodal-prompt.md](multimodal/multimodal-prompt.md)
**Technical Details:** [important.md](multimodal/important.md)

### Prompts: 27

**Categories:**
- Basic Image Understanding (3)
- Counting and Quantification (2)
- Visual Prompting (2)
- Gauge and Display Reading (2)
- Multi-Image Processing (1)
- Object Identification (1)
- Text Transcription (3)
- Document Q&A (1)
- Structured Data Extraction (1)
- Financial Document Analysis (8)
- Multi-Agent Orchestration (3)

### Key Capabilities
- PDF document processing (beta)
- Handwritten text digitization
- Chart and graph analysis
- Code extraction from screenshots
- Sub-agent architecture for parallel processing
- Slide deck narration for RAG

### Best Practices
- Chain-of-thought for complex visual analysis
- Few-shot learning for specialized tasks
- Visual prompting with embedded instructions
- Role assignment reduces hallucination

---

## Advanced Techniques

**Location:** `misc/`
**Detailed Prompts:** [advanced-techniques-prompt.md](misc/advanced-techniques-prompt.md)
**Technical Details:** [important.md](misc/important.md)

### Prompts: 15+

**Categories:**
1. SQL Query Generation
2. Content Moderation
3. Model-Based Evaluation
4. Image Generation Integration
5. Customer Support Agent
6. Document Q&A with Citations
7. Socratic Math Tutor
8. Function Calling
9. Meta-Prompting
10. Synthetic Test Data
11. JSON Mode
12. PDF Summarization
13. Long Context QA
14. Web Scraping
15. Citations

### Performance Optimization
- **Prompt Caching:** >2x faster, up to 90% cost reduction
- **Batch Processing:** 50% cost reduction
- **Long Context:** Position matters (Beginning 85% > End 80% > Middle 75%)

### Key Features
- Citations API for source tracking
- Base64 PDF processing
- Prefill technique for JSON output
- Stop sequences for output control
- Speculative caching for 90% faster TTFT

---

## Summary Statistics

| Topic | Prompts | Key Improvement | Main Focus |
|-------|---------|----------------|------------|
| Classification | 3 | 70% → 95.6% | Ticket categorization |
| RAG | 5 | 71% → 81% | Document Q&A |
| Summarization | 9 | Varies by method | Legal documents |
| Tool Use | 23 | N/A | External integrations |
| Third-Party | 24 | N/A | Platform integrations |
| Multimodal | 27 | N/A | Vision + text |
| Advanced | 15+ | >2x faster (caching) | Production techniques |
| **TOTAL** | **106+** | | |

---

## Common Patterns Across All Topics

### Prompt Engineering
- **XML tags** for structured data
- **Assistant prefill** with stop sequences
- **Chain-of-thought** reasoning
- **Few-shot learning** (2-3 examples)
- **Temperature 0** for deterministic tasks

### Evaluation
- Code-based > Model-based > Human
- Separate metrics for different components
- LLM-as-judge for semantic evaluation
- Volume over perfection

### Production Best Practices
- Model selection: Haiku (cost) → Sonnet (balance) → Opus (quality)
- Implement caching strategies
- Rate limit handling
- Error handling and fallbacks
- Security validation (especially for user inputs)

---

## Quick Reference

- For **classification tasks** → Start with simple, add RAG, then CoT
- For **Q&A systems** → Use RAG with re-ranking
- For **long documents** → Use chunking + meta-summarization
- For **external data** → Use tool use patterns
- For **vision tasks** → Add chain-of-thought and few-shot examples
- For **production** → Enable prompt caching and batch processing

---

**Last Updated:** 2025-09-30
**Total Prompts Cataloged:** 106+

For detailed prompts and implementation details, see the individual documentation files in each topic folder.