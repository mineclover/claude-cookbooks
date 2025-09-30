# RAG Prompts Collection

This document contains all AI prompts used in the Retrieval Augmented Generation (RAG) cookbook examples, organized by their specific purposes.

---

## Document Summarization

**Prompt:**
```
You are tasked with creating a short summary of the following content from Anthropic's documentation.

Context about the knowledge base:
{knowledge_base_context}

Content to summarize:
Heading: {doc['chunk_heading']}
{doc['text']}

Please provide a brief summary of the above content in 2-3 sentences. The summary should capture the key points and be concise. We will be using it as a key part of our search pipeline when answering user queries about this content.

Avoid using any preamble whatsoever in your response. Statements such as 'here is the summary' or 'the summary is as follows' are prohibited. You should get straight into the summary itself and be concise. Every word matters.
```

**Purpose:** Generates concise 2-3 sentence summaries of document chunks to enhance retrieval performance. These summaries are embedded alongside the original content to capture the essence of each chunk more effectively. Used in Level 2 (Summary Indexing) approach.

**Key Parameters:**
- Model: claude-3-haiku-20240307
- Max tokens: 150
- Temperature: 0

---

## Query Answering - Basic RAG (Level 1)

**Prompt:**
```
You have been tasked with helping us to answer the following query:
<query>
{query}
</query>
You have access to the following documents which are meant to provide context as you answer the query:
<documents>
{context}
</documents>
Please remain faithful to the underlying context, and only deviate from it if you are 100% sure that you know the answer already.
Answer the question now, and avoid providing preamble such as 'Here is the answer', etc
```

**Purpose:** Generates answers to user queries using retrieved document chunks as context. This is the foundational prompt used across all three levels of RAG implementation (Basic RAG, Summary Indexing, and Summary Indexing + Re-Ranking). The prompt emphasizes faithfulness to the source material and concise responses.

**Key Parameters:**
- Model: claude-3-haiku-20240307
- Max tokens: 2500
- Temperature: 0

**Context Format (Level 1 - Basic RAG):**
```
{chunk['text']}
```

**Context Format (Level 2 - Summary Indexing):**
```
<document>
{chunk['chunk_heading']}

Text
{chunk['text']}

Summary:
{chunk['summary']}
</document>
```

**Context Format (Level 3 - Summary Indexing + Re-Ranking):**
```
<document>
{chunk['chunk_heading']}

{chunk['text']}
</document>
```

---

## Document Re-Ranking (Level 3)

**Prompt:**
```
Query: {query}
You are about to be given a group of documents, each preceded by its index number in square brackets. Your task is to select the only {k} most relevant documents from the list to help us answer the query.

<documents>
{joined_summaries}
</documents>

Output only the indices of {k} most relevant documents in order of relevance, separated by commas, enclosed in XML tags here:
<relevant_indices>put the numbers of your indices here, seeparted by commas</relevant_indices>
```

**Purpose:** Re-ranks initially retrieved documents (typically 20) to identify the most relevant subset (typically 3) for answering a query. This two-stage retrieval approach (cast wide net, then focus) significantly improved Mean Reciprocal Rank (MRR) from 0.74 to 0.87.

**Key Parameters:**
- Model: claude-3-haiku-20240307
- Max tokens: 50
- Temperature: 0
- Stop sequences: ["</relevant_indices>"]

**Input Format:**
```
[0] Document Summary: {result['metadata']['summary']}

[1] Document Summary: {result['metadata']['summary']}

[2] Document Summary: {result['metadata']['summary']}
...
```

---

## End-to-End Evaluation (LLM-as-Judge)

**Prompt:**
```
You are an AI assistant tasked with evaluating the correctness of answers to questions about Anthropic's documentation.

Question: {query}

Correct Answer: {correct_answer}

Generated Answer: {generated_answer}

Is the Generated Answer correct based on the Correct Answer? You should pay attention to the substance of the answer, and ignore minute details that may differ.

Small differences or changes in wording don't matter. If the generated answer and correct answer are saying essentially the same thing then that generated answer should be marked correct.

However, if there is any critical piece of information which is missing from the generated answer in comparison to the correct answer, then we should mark this as incorrect.

Finally, if there are any direct contradictions between the correect answer and generated answer, we should deem the generated answer to be incorrect.

Respond in the following XML format:
<evaluation>
<content>
<explanation>Your explanation here</explanation>
<is_correct>true/false</is_correct>
</content>
</evaluation>
```

**Purpose:** Evaluates whether generated answers are correct compared to ground truth answers. This LLM-as-judge approach focuses on semantic equivalence rather than exact string matching, allowing for natural language variation while catching missing critical information or contradictions.

**Key Parameters:**
- Model: claude-3-5-sonnet-20241022
- Max tokens: 1500
- Temperature: 0
- Stop sequences: ["</evaluation>"]
- Prefilled assistant response: "<evaluation>"

**Evaluation Criteria:**
1. Semantic equivalence: Generated and correct answers saying essentially the same thing = CORRECT
2. Missing critical information: Any critical piece missing from generated answer = INCORRECT
3. Direct contradictions: Any contradictions between answers = INCORRECT
4. Minor differences in wording are ignored

---

## Prompt Engineering Patterns Used

### 1. XML Tag Structuring
All prompts use XML tags (`<query>`, `<documents>`, `<evaluation>`, etc.) to clearly delineate different sections of the prompt, improving Claude's ability to parse and respond to structured information.

### 2. Direct Instructions
Prompts avoid preamble and get straight to the task, particularly evident in the summarization prompt: "Avoid using any preamble whatsoever in your response."

### 3. Prefilled Responses
The evaluation and re-ranking prompts use prefilled assistant responses with opening XML tags to ensure consistent, parseable output format.

### 4. Contextual Grounding
The query answering prompt explicitly instructs Claude to "remain faithful to the underlying context, and only deviate from it if you are 100% sure," reducing hallucination.

### 5. Negative Instructions
Prompts specify what NOT to do (e.g., "avoid providing preamble such as 'Here is the answer'") to shape output format.

### 6. Explicit Output Format
The re-ranking and evaluation prompts specify exact output formats using XML tags, making responses machine-parseable and reducing parsing errors.