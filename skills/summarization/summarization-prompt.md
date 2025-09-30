# Summarization Prompts - Claude Cookbook

This document contains all AI prompts extracted from the summarization cookbook examples, organized by their purpose and use case.

---

## Basic Summarization

**Prompt:**
```
Summarize the following text in bullet points. Focus on the main ideas and key details:
{text}
```

**System Message:**
```
You are a legal analyst known for highly accurate and detailed summaries of legal documents.
```

**Assistant Preamble:**
```
Here is the summary of the legal document: <summary>
```

**Purpose:**
This is a simple, straightforward summarization approach that asks Claude to condense text into bullet points. It uses an assistant preamble (`<summary>`) to constrain output format and a stop sequence (`</summary>`) to control when generation ends. Suitable as a baseline for legal document summarization.

---

## Multi-Shot Basic Summarization

**Prompt:**
```
Summarize the following text in bullet points. Focus on the main ideas and key details:
{text}

Do not preamble.

Use these examples for guidance in summarizing:

<example1>
    <original1>
        {document1}
    </original1>

    <summary1>
        {sample1}
    </summary1>
</example1>

<example2>
    <original2>
        {document2}
    </original2>

    <summary2>
        {sample2}
    </summary2>
</example2>

<example3>
    <original3>
        {document3}
    </original3>

    <summary3>
        {sample3}
    </summary3>
</example3>
```

**System Message:**
```
You are a legal analyst known for highly accurate and detailed summaries of legal documents.
```

**Assistant Preamble:**
```
Here is the summary of the legal document: <summary>
```

**Purpose:**
Enhances basic summarization with few-shot learning by providing 3 example document-summary pairs. The "Do not preamble" instruction eliminates conversational fluff. This helps Claude understand the desired output format and structure without explicit formatting instructions. The model naturally picks up patterns from examples.

---

## Guided Legal Summarization

**Prompt:**
```
Summarize the following legal document. Focus on these key aspects:

1. Parties involved
2. Main subject matter
3. Key terms and conditions
4. Important dates or deadlines
5. Any unusual or notable clauses

Provide the summary in bullet points under each category.

Document text:
{text}
```

**System Message:**
```
You are a legal analyst known for highly accurate and detailed summaries of legal documents.
```

**Assistant Preamble:**
```
Here is the summary of the legal document: <summary>
```

**Purpose:**
Provides explicit framework for summarization by defining specific categories to focus on. This guided approach ensures consistent coverage of important legal aspects and makes summaries easier to parse. Can be accomplished either through explicit instructions (as shown) or via examples.

---

## Domain-Specific Guided Summarization (Sublease Agreements)

**Prompt:**
```
Summarize the following sublease agreement. Focus on these key aspects:

1. Parties involved (sublessor, sublessee, original lessor)
2. Property details (address, description, permitted use)
3. Term and rent (start date, end date, monthly rent, security deposit)
4. Responsibilities (utilities, maintenance, repairs)
5. Consent and notices (landlord's consent, notice requirements)
6. Special provisions (furniture, parking, subletting restrictions)

Provide the summary in bullet points nested within the XML header for each section. For example:

<parties involved>
- Sublessor: [Name]
// Add more details as needed
</parties involved>

If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

Sublease agreement text:
{text}
```

**System Message:**
```
You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.
```

**Assistant Preamble:**
```
Here is the summary of the sublease agreement: <summary>
```

**Purpose:**
Highly specialized prompt tailored specifically for sublease agreements. Uses XML tags for structured output that can be easily parsed programmatically. The domain-specific categories and system message improve accuracy for this document type. The "Not specified" instruction handles missing information gracefully.

---

## Meta-Summarization (Long Documents with Chunking)

**Prompt for Initial Chunks:**
```
Summarize the following sublease agreement. Focus on these key aspects:

1. Parties involved (sublessor, sublessee, original lessor)
2. Property details (address, description, permitted use)
3. Term and rent (start date, end date, monthly rent, security deposit)
4. Responsibilities (utilities, maintenance, repairs)
5. Consent and notices (landlord's consent, notice requirements)
6. Special provisions (furniture, parking, subletting restrictions)

Provide the summary in bullet points nested within the XML header for each section. For example:

<parties involved>
- Sublessor: [Name]
// Add more details as needed
</parties involved>

If any information is not explicitly stated in the document, note it as "Not specified".

Sublease agreement text:
{content[:2000]}...

Summary:
```

**Prompt for Final Meta-Summary:**
```
You are looking at the chunked summaries of multiple documents that are all related. Combine the following summaries of the document from different truthful sources into a coherent overall summary:

{combined_chunk_summaries}

1. Parties involved (sublessor, sublessee, original lessor)
2. Property details (address, description, permitted use)
3. Term and rent (start date, end date, monthly rent, security deposit)
4. Responsibilities (utilities, maintenance, repairs)
5. Consent and notices (landlord's consent, notice requirements)
6. Special provisions (furniture, parking, subletting restrictions)

Provide the summary in bullet points nested within the XML header for each section. For example:

<parties involved>
- Sublessor: [Name]
// Add more details as needed
</parties involved>

If any information is not explicitly stated in the document, note it as "Not specified".

Summary:
```

**System Message (Chunks):**
```
You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.
```

**System Message (Final):**
```
You are a legal expert that summarizes notes on one document.
```

**Purpose:**
Handles documents longer than typical token limits through a two-stage process: (1) Chunk the document and summarize each chunk individually using a faster/cheaper model (Haiku), (2) Combine chunk summaries into a coherent meta-summary using a more powerful model (Sonnet). This approach balances cost, speed, and quality for long documents.

---

## Summary Indexed Documents - Ranking Query

**Prompt:**
```
Legal document summary: {summary}

Legal query: {query}

Rate the relevance of this legal document to the query on a scale of 0 to 10. Only output the numeric value:
```

**Purpose:**
Part of an advanced RAG (Retrieval-Augmented Generation) approach. This prompt ranks document summaries by relevance to a user query. Uses a lightweight model (Haiku) for fast, cost-effective ranking. The constraint to output only a numeric value ensures easy parsing. Summaries are pre-generated at ingestion time, making retrieval efficient.

---

## Summary Indexed Documents - Clause Extraction

**Prompt:**
```
Given the following legal query and document content, extract the most relevant clauses or sections and write the answer to the query.
Provide each relevant clause or section separately, preserving the original legal language:

Legal query: {query}

Document content: {doc_content}...

Relevant clauses or sections (separated by '---'):
```

**Purpose:**
After ranking documents by relevance, this prompt extracts specific clauses or sections that answer the user's query. It preserves original legal language for accuracy and separates clauses with '---' for easy parsing. Used in the second stage of the Summary Indexed Documents RAG approach.

---

## LLM-Based Evaluation Prompt

**Prompt:**
```
Evaluate the following summary based on these criteria:
1. Conciseness (1-5)
2. Accuracy (1-5)
3. Completeness (1-5)
4. Clarity (1-5)
5. Explanation - a general description of the way the summary is evaluated

Here are some things to think about as you go about grading.

1. Does the summary accurately capture the key provisions of the legal document?
2. Does the summary omit any important details from the legal document?
3. Does the summary contain any inaccuracies or misrepresentations of the legal document?
4. Does the summary fairly represent the legal document as a whole, or does it unduly emphasize certain provisions over others?
5. Does the summary accurately reflect the language and tone of the legal document?
6. Does the summary capture the key concepts and principles embodied in the legal document?
7. Does the summary omit any important ideas that should be captured to make decisions using the document?

Provide a score for each criterion in JSON format. Here is the format you should follow always:

<json>
{
"conciseness": <number>,
"accuracy": <number>,
"completeness": <number>,
"clarity": <number>,
"explanation": <string>,
}
</json>

Original Text: {input}

Summary to Evaluate: {summary}

Evaluation (JSON format):
```

**Assistant Preamble:**
```
<json>
```

**Purpose:**
Uses Claude to evaluate summary quality across multiple dimensions (conciseness, accuracy, completeness, clarity). Provides detailed evaluation criteria and requires structured JSON output for programmatic analysis. Part of the Promptfoo evaluation framework. This LLM-as-judge approach is valuable because traditional metrics like ROUGE/BLEU don't capture nuanced quality aspects.