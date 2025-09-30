# Third-Party Integration Prompts for Claude

This document contains all prompts used in the third-party integration cookbook examples where Claude is asked to perform specific tasks with external services and tools.

---

## Pinecone - RAG Agent with Tool Use

### Tool Description for ArXiv Search
**Prompt:**
```
Use this tool when answering questions about AI, machine learning, data science, or other technical questions that may be answered using arXiv papers.
```
**Purpose:** Describes when and how to use the arXiv search tool that queries a Pinecone vector database with embedded research papers.

---

## Pinecone - RAG Product Search

### Keyword Generation Prompt
**Prompt:**
```
Given a question, generate a list of 5 very diverse search keywords that can be used to search for products on Amazon.

The question is: {question}

Output your keywords as a JSON that has one property "keywords" that is a list of strings. Only output valid JSON.
```
**Purpose:** Uses Claude to generate diverse search keywords from a user's question to optimize product search retrieval from Pinecone vector database.

### Product Recommendation Prompt
**Prompt:**
```
{format_results(results_list)} Using the search results provided within the <search_results></search_results> tags, please answer the following question <question>{question}</question>. Do not reference the search results in your answer.
```
**Purpose:** Generates product recommendations based on retrieved search results from Pinecone, providing natural answers without directly referencing sources.

---

## Wikipedia - Iterative Search Tool Use

### Tool Description Prompt
**Prompt:**
```
You will be asked a question by a human user. You have access to the following tool to help answer the question. <tool_description> Search Engine Tool * The search engine will exclusively search over Wikipedia for pages similar to your query. It returns for each page its title and full page content. Use this tool if you want to get up-to-date and comprehensive information on a topic to help answer queries. Queries should be as atomic as possible -- they only need to address one part of the user's question. For example, if the user's query is "what is the color of a basketball?", your search query should be "basketball". Here's another example: if the user's question is "Who created the first neural network?", your first query should be "neural network". As you can see, these queries are quite short. Think keywords, not phrases. * At any time, you can make a call to the search engine using the following syntax: <search_query>query_word</search_query>. * You'll then get results back in <search_result> tags.</tool_description>
```
**Purpose:** Teaches Claude how to use Wikipedia search effectively with atomic keyword queries rather than full phrases.

### Retrieval and Search Quality Reflection Prompt
**Prompt:**
```
Before beginning to research the user's question, first think for a moment inside <scratchpad> tags about what information is necessary for a well-informed answer. If the user's question is complex, you may need to decompose the query into multiple subqueries and execute them individually. Sometimes the search engine will return empty search results, or the search results may not contain the information you need. In such cases, feel free to try again with a different query.

After each call to the Search Engine Tool, reflect briefly inside <search_quality></search_quality> tags about whether you now have enough information to answer, or whether more information is needed. If you have all the relevant information, write it in <information></information> tags, WITHOUT actually answering the question. Otherwise, issue a new search.

Here is the user's question: <question>{query}</question> Remind yourself to make short queries in your scratchpad as you plan out your strategy.
```
**Purpose:** Encourages Claude to plan searches, decompose complex queries, reflect on search quality, and persist until sufficient information is gathered.

### Final Answer Generation Prompt
**Prompt:**
```
Here is a user query: <query>{query}</query>. Here is some relevant information: <information>{information}</information>. Please answer the question using the relevant information.
```
**Purpose:** Separates information synthesis from search execution, allowing Claude to focus on generating accurate answers based on collected information.

---

## WolframAlpha - Computational Knowledge

### Tool Description for WolframAlpha
**Prompt:**
```
A tool that allows querying the Wolfram Alpha knowledge base. Useful for mathematical calculations, scientific data, and general knowledge questions.
```
**Purpose:** Describes the WolframAlpha tool capabilities for Claude to use for computational and scientific queries.

### System Prompt for Query Processing
**Prompt:**
```
Here is a question: {user_message}. Please use the Wolfram Alpha tool to answer it. Do not reflect on the quality of the returned search results in your response.
```
**Purpose:** Instructs Claude to use WolframAlpha tool and provide direct answers without meta-commentary about search quality.

---

## MongoDB - RAG Venture Capital Analyst

### System Role and Context Prompt
**Prompt:**
```
You are Venture Captital Tech Analyst with access to some tech company articles and information. You use the information you are given to provide advice.
```
**Purpose:** Sets Claude's persona and role for analyzing tech companies using MongoDB-retrieved information.

### Query Handling with Context Prompt
**Prompt:**
```
Answer this user query: {query} with the following context: {search_result}
```
**Purpose:** Provides Claude with MongoDB vector search results to answer venture capital investment questions.

---

## Deepgram - Audio Transcription Analysis

### Interview Question Generation Prompt
**Prompt:**
```
Your task is to generate a series of thoughtful, open-ended questions for an interview based on the given context. The questions should be designed to elicit insightful and detailed responses from the interviewee, allowing them to showcase their knowledge, experience, and critical thinking skills. Avoid yes/no questions or those with obvious answers. Instead, focus on questions that encourage reflection, self-assessment, and the sharing of specific examples or anecdotes.
```
**Purpose:** Uses Claude to analyze Deepgram transcriptions and generate meaningful interview questions based on the audio content.

---

## LlamaIndex - Basic RAG Query

### Simple Query Prompt
**Prompt:**
```
What did author do growing up?
```
**Purpose:** Example of straightforward question answering using LlamaIndex RAG pipeline with Claude as the LLM.

---

## LlamaIndex - ReAct Agent with Calculator Tools

### ReAct Agent System Prompt
**Prompt:**
```
You are designed to help with a variety of tasks, from answering questions to providing summaries to other types of analyses.

## Tools
You have access to a wide variety of tools. You are responsible for using the tools in any sequence you deem appropriate to complete the task at hand. This may require breaking the task into subtasks and using different tools to complete each subtask.

You have access to the following tools:
{tool_desc}

## Output Format
To answer the question, please use the following format.

```
Thought: I need to use a tool to help me answer the question.
Action: tool name (one of {tool_names}) if using a tool.
Action Input: the input to the tool, in a JSON format representing the kwargs (e.g. {"input": "hello world", "num_beams": 5})
```

Please ALWAYS start with a Thought.

Please use a valid JSON format for the Action Input. Do NOT do this {'input': 'hello world', 'num_beams': 5}.

If this format is used, the user will respond in the following format:

```
Observation: tool response
```

You should keep repeating the above format until you have enough information to answer the question without using any more tools. At that point, you MUST respond in the one of the following two formats:

```
Thought: I can answer without using any more tools.
Answer: [your answer here]
```

```
Thought: I cannot answer the question with the provided tools.
Answer: Sorry, I cannot answer your query.
```

## Current Conversation
Below is the current conversation consisting of interleaving human and assistant messages.
```
**Purpose:** Comprehensive ReAct agent system prompt that teaches Claude how to use tools, reason through problems, and format responses properly.

### Tool Descriptions - Calculator Functions
**Prompt:**
```
Multiply two integers and returns the result integer
```
```
Add two integers and returns the result integer
```
**Purpose:** Simple tool descriptions that Claude uses to determine when to use mathematical operations.

---

## LlamaIndex - ReAct Agent with Query Engine Tools

### Lyft 10K Query Engine Tool Description
**Prompt:**
```
Provides information about Lyft financials for year 2021. Use a detailed plain text question as input to the tool.
```
**Purpose:** Describes the Lyft financial data query engine for Claude's tool selection.

### Uber 10K Query Engine Tool Description
**Prompt:**
```
Provides information about Uber financials for year 2021. Use a detailed plain text question as input to the tool.
```
**Purpose:** Describes the Uber financial data query engine for Claude's tool selection.

---

## LlamaIndex - Multi-Document Agents

### City-Specific Vector Tool Description
**Prompt:**
```
Useful for retrieving specific context from {wiki_title}
```
**Purpose:** Describes vector search tool for retrieving specific facts about a city from Wikipedia.

### City-Specific Summary Tool Description
**Prompt:**
```
Useful for summarization questions related to {wiki_title}
```
**Purpose:** Describes summary tool for generating summaries about a city from Wikipedia.

### Top-Level Index Node Description
**Prompt:**
```
This content contains Wikipedia articles about {wiki_title}. Use this index if you need to lookup specific facts about {wiki_title}.
Do not use this index if you want to analyze multiple cities.
```
**Purpose:** Guides Claude to select the appropriate city-specific agent when multiple document agents are available.

---

## LlamaIndex - Router Query Engine

### Summary Query Engine Tool Description
**Prompt:**
```
Useful for summarization questions related to Paul Graham eassy on What I Worked On.
```
**Purpose:** Describes when to use the summary index for high-level overview questions.

### Vector Query Engine Tool Description
**Prompt:**
```
Useful for retrieving specific context from Paul Graham essay on What I Worked On.
```
**Purpose:** Describes when to use the vector index for specific factual retrieval.

---

## LlamaIndex - SubQuestion Query Engine

### Query Engine Tool Descriptions for Financial Data
**Prompt:**
```
Provides information about Lyft financials for year 2021
```
```
Provides information about Uber financials for year 2021
```
**Purpose:** Enables Claude to decompose complex comparative queries into sub-questions and route them to appropriate data sources.

---

## LlamaIndex - Multi-Modal Image Understanding

### Image Description Prompt
**Prompt:**
```
Describe the images as an alternative text
```
**Purpose:** Uses Claude's vision capabilities to generate accessible alternative text descriptions for images.

### Structured Output Extraction Prompt
**Prompt:**
```
Can you get the stock information in the image and return the answer? Pick just one fund.

Make sure the answer is a JSON format corresponding to a Pydantic schema. The Pydantic schema is given below.
```
**Purpose:** Extracts structured financial data from images using Claude's multimodal capabilities with Pydantic schema validation.

---

## VoyageAI - Embeddings

Note: VoyageAI integration is primarily for creating embeddings. No direct Claude prompts are used in this cookbook as it focuses on the embedding API usage. VoyageAI embeddings are then used with Claude in other RAG examples (Pinecone, MongoDB, etc.).

---

## Summary

**Total Prompts Found:** 24 distinct prompts across integrations

**Main Use Cases by Integration:**

### Pinecone (3 prompts)
- RAG tool use with arXiv papers
- Product search keyword generation
- Product recommendation from search results

### Wikipedia (3 prompts)
- Tool-based iterative search
- Search planning and quality reflection
- Final answer synthesis

### WolframAlpha (2 prompts)
- Tool description for computational queries
- Direct query processing

### MongoDB (2 prompts)
- Venture capital analyst role
- Context-based query answering

### Deepgram (1 prompt)
- Interview question generation from transcripts

### LlamaIndex (13 prompts)
- Basic RAG queries
- ReAct agent system prompts
- Tool descriptions (calculators, query engines)
- Multi-document agent routing
- Router query engine selection
- SubQuestion query decomposition
- Multi-modal image understanding and extraction

### VoyageAI (0 prompts)
- Embedding generation only (no Claude prompts)