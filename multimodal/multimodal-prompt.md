# Multimodal Prompts - Claude AI Use Cases

This document contains all prompts extracted from the Claude multimodal cookbook examples, organized by their purpose and use case.

---

## Basic Image Description

### Simple Image Narration
**Prompt:**
```
Write a sonnet based on this image.
```
**Purpose:** Generate creative content (poetry) based on visual input. Tests Claude's ability to understand imagery and create artistic responses.

### Concise Image Description
**Prompt:**
```
Describe this image in two sentences.
```
**Purpose:** Get brief, factual descriptions of images. Useful for quick summaries and alt-text generation.

### Visual Content Identification
**Prompt:**
```
What's in this document? Answer in a single sentence.
```
**Purpose:** Quick document overview to understand content before detailed analysis.

---

## Counting and Quantification

### Basic Object Counting
**Prompt:**
```
How many dogs are in this picture?
```
**Purpose:** Count specific objects in images. May produce inaccurate results without proper prompt engineering.

### Enhanced Counting with Chain of Thought
**Prompt:**
```
You have perfect vision and pay great attention to detail which makes you an expert at counting objects in images. How many dogs are in this picture? Before providing the answer in <answer> tags, think step by step in <thinking> tags and analyze every part of the image.
```
**Purpose:** Improve counting accuracy through role assignment and chain-of-thought reasoning. Significantly reduces hallucination in counting tasks.

---

## Visual Prompting (Text/Annotations in Images)

### Mathematical Problem Solving from Image
**Prompt:**
```
[No text prompt - question written directly in image: "What is the area of the circle if the radius is 12?"]
```
**Purpose:** Demonstrate visual prompting where instructions are embedded in the image itself. Claude can read and respond to questions written in images.

### Comparative Analysis from Highlighted Data
**Prompt:**
```
What's the difference between these two numbers?
```
**Purpose:** Analyze specific highlighted portions of images (like table cells). Shows Claude can understand visual indicators like circles/highlights.

---

## Speedometer and Gauge Reading

### Basic Speed Reading
**Prompt:**
```
What speed am I going?
```
**Purpose:** Read analog displays. May misinterpret units without examples.

### Few-Shot Speed Reading (with examples)
**Prompt:**
```
[After providing examples of 70 mph and 100 mph speedometers]
What speed am I going?
```
**Purpose:** Use few-shot learning to improve accuracy in reading analog gauges and establish unit conventions (mph vs km/h).

---

## Multi-Image Processing

### Receipt Data Extraction
**Prompt:**
```
Output the name of the restaurant and the total.
```
**Purpose:** Extract specific information from multiple image fragments. Shows Claude can process and combine information from several images simultaneously.

---

## Object Identification from Examples

### Visual Classification with Reference Images
**Prompt:**
```
These pants are (in order) WRINKLE-RESISTANT DRESS PANT, ITALIAN MELTON OFFICER PANT, SLIM RAPID MOVEMENT CHINO. What pant is shown in the last image?
```
**Purpose:** Classify objects by providing labeled examples. Enables visual few-shot learning for product identification and categorization.

---

## Text Transcription

### Code Transcription from Screenshot
**Prompt:**
```
Transcribe the code in the answer. Only output the code.
```
**Purpose:** Extract code snippets from screenshots while filtering out other content. Demonstrates selective transcription.

### Handwritten Text Transcription
**Prompt:**
```
Transcribe this text. Only output the text and nothing else.
```
**Purpose:** Convert handwritten notes to digital text. Claude excels at reading cursive and varied handwriting styles.

### Form Transcription
**Prompt:**
```
Transcribe this form exactly.
```
**Purpose:** Extract both typed and handwritten content from structured forms. Preserves formatting and structure.

---

## Document Question Answering

### Complex Document Analysis
**Prompt:**
```
Which is the most critical issue for live rep support?
```
**Purpose:** Answer questions requiring understanding of hierarchical information (like pyramids) in documents. Goes beyond simple transcription to interpretation.

---

## Structured Data Extraction

### Visual to JSON Conversion
**Prompt:**
```
Turn this org chart into JSON indicating who reports to who. Only output the JSON and nothing else.
```
**Purpose:** Convert unstructured visual information (diagrams, charts) into structured data formats. Useful for data migration and analysis.

---

## Financial Document Analysis

### Revenue Query from Charts
**Prompt:**
```
What was CVNA revenue in 2020?
```
**Purpose:** Extract specific numerical data from financial charts and graphs.

### Market Growth Calculation
**Prompt:**
```
How many additional markets has Carvana added since 2014?
```
**Purpose:** Calculate differences and growth metrics from visual data.

### Revenue Per Unit Calculation
**Prompt:**
```
What was 2016 revenue per retail unit sold?
```
**Purpose:** Perform calculations combining multiple data points from charts. Tests arithmetic capabilities with visual data.

### Year-over-Year Growth Analysis
**Prompt:**
```
What was Twilio y/y revenue growth for fiscal year 2023?
```
**Purpose:** Extract growth metrics from multi-page financial presentations.

---

## Slide Deck Narration

### Complete Presentation Narration
**Prompt:**
```
You are the Twilio CFO, narrating your Q4 2023 earnings presentation.

The entire earnings presentation document is provided to you.
Please narrate this presentation from Twilio's Q4 2023 Earnings as if you were the presenter. Do not talk about any things, especially acronyms, if you are not exactly sure you know what they mean.

Do not leave any details un-narrated as some of your viewers are vision-impaired, so if you don't narrate every number they won't know the number.

Structure your response like this:
<narration>
    <page_narration id=1>
    [Your narration for page 1]
    </page_narration>

    <page_narration id=2>
    [Your narration for page 2]
    </page_narration>

    ... and so on for each page
</narration>

Use excruciating detail for each page, ensuring you describe every visual element and number present. Show the full response in a single message.
```
**Purpose:** Convert entire slide decks into detailed text narrations. Enables accessibility and text-based analysis of visual presentations. Creates searchable text from presentation materials.

### Segment Revenue Analysis
**Prompt:**
```
What percentage of q4 total revenue was the Segment business line?
```
**Purpose:** Analyze narrated deck content to answer specific financial questions.

### Revenue Trend Analysis
**Prompt:**
```
Has the rate of growth of quarterly revenue been increasing or decreasing? Give just an answer.
```
**Purpose:** Identify trends from narrated financial data.

### Acquisition Revenue Calculation
**Prompt:**
```
What was acquisition revenue for the year ended december 31, 2023 (including negative revenues)?
```
**Purpose:** Perform complex calculations from narrated presentation data.

---

## Sub-Agent Orchestration

### Generate Sub-Agent Prompts
**Prompt:**
```
Based on the following question, please generate a specific prompt for an LLM sub-agent to extract relevant information from an earning's report PDF. Each sub-agent only has access to a single quarter's earnings report. Output only the prompt and nothing else.

Question: {QUESTION}
```
**Purpose:** Use a more powerful model (Opus) to generate optimized prompts for sub-agents (Haiku). Enables intelligent task decomposition.

### Sub-Agent Information Extraction
**Prompt (Generated by Opus):**
```
Extract the following information from the Apple earnings report PDF for the quarter:
1. Apple's net sales for the quarter
2. Quarter-over-quarter change in net sales
3. Key product categories, services, or regions that contributed significantly to the change in net sales
4. Any explanations provided for the changes in net sales

Organize the extracted information in a clear, concise format focusing on the key data points and insights related to the change in net sales for the quarter.
```
**Purpose:** Specific extraction task for Haiku sub-agents processing individual quarterly reports.

### Synthesis and Visualization
**Prompt:**
```
Based on the following extracted information from Apple's earnings releases, please provide a response to the question: {QUESTION}

Also, please generate Python code using the matplotlib library to accompany your response. Enclose the code within <code> tags.

Extracted Information:
{extracted_info}
```
**Purpose:** Combine information from multiple sub-agents, synthesize an answer, and generate visualization code. Demonstrates multi-agent coordination.

---

## Summary Statistics

**Total Prompts Extracted:** 27 distinct prompts across 10 major use case categories

**Use Case Categories:**
1. Basic Image Description (3 prompts)
2. Counting and Quantification (2 prompts)
3. Visual Prompting (2 prompts)
4. Speedometer/Gauge Reading (2 prompts)
5. Multi-Image Processing (1 prompt)
6. Object Identification (1 prompt)
7. Text Transcription (3 prompts)
8. Document QA (1 prompt)
9. Structured Data Extraction (1 prompt)
10. Financial Analysis (8 prompts)
11. Sub-Agent Orchestration (3 prompts)