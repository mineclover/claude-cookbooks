# Tool Use Prompts - Claude Cookbooks

This document contains AI prompts extracted from the tool_use cookbook examples, organized by use case and purpose.

---

## Customer Service Agent

### Use Case: Customer Information Retrieval
**Prompt:**
```
Can you tell me the email address for customer C1?
```
**Purpose:** Demonstrates Claude using a `get_customer_info` tool to retrieve customer data based on customer ID. Claude identifies the parameter from the query and calls the appropriate tool to fetch email, name, and phone information.

**Context:** Part of a customer service chatbot with tools for customer lookup, order details, and order cancellation.

---

### Use Case: Order Status Inquiry
**Prompt:**
```
What is the status of order O2?
```
**Purpose:** Shows Claude using a `get_order_details` tool to retrieve order information including product, quantity, price, and status. The model extracts the order ID from the natural language query.

**Context:** Customer service scenario where users inquire about their order status without using structured commands.

---

### Use Case: Order Cancellation
**Prompt:**
```
Please cancel order O1 for me.
```
**Purpose:** Demonstrates Claude calling a `cancel_order` tool to process order cancellations. The model understands the user intent, extracts the order ID, and executes the cancellation action.

**Context:** Empowering customer service bots to handle transactional requests through natural language.

---

## Calculator Tool

### Use Case: Large Number Multiplication
**Prompt:**
```
What is the result of 1,984,135 * 9,343,116?
```
**Purpose:** Tests Claude's ability to recognize when it needs computational tools for calculations beyond its native capabilities. Shows the model formatting numbers correctly and calling the calculator tool.

**Context:** Demonstrates tool use for mathematical operations that require precision beyond LLM approximations.

---

### Use Case: Complex Expression Evaluation
**Prompt:**
```
Calculate (12851 - 593) * 301 + 76
```
**Purpose:** Shows Claude parsing complex mathematical expressions with multiple operations and parentheses, then passing them to a calculator tool while preserving operator precedence.

**Context:** Handling multi-step arithmetic that requires exact computation.

---

### Use Case: Division with Decimals
**Prompt:**
```
What is 15910385 divided by 193053?
```
**Purpose:** Demonstrates Claude using tools for division operations that result in floating-point numbers requiring precision.

**Context:** Mathematical operations where approximate LLM computation would be insufficient.

---

## Structured JSON Extraction

### Use Case: Article Summarization
**Prompt:**
```xml
<article>
{article}
</article>

Use the `print_summary` tool.
```
**Purpose:** Extracts structured information from articles including author, topics, summary, coherence score (0-100), and persuasion score (0.0-1.0). Uses tool use as a method to force structured JSON output.

**Context:** Converting unstructured text into structured data with specific schemas including strings, arrays, integers, and floats.

---

### Use Case: Named Entity Recognition
**Prompt:**
```xml
<document>
John works at Google in New York. He met with Sarah, the CEO of Acme Inc., last week in San Francisco.
</document>

Use the print_entities tool.
```
**Purpose:** Extracts named entities (PERSON, ORGANIZATION, LOCATION) from text with context about where each entity appears. Returns structured array of entity objects.

**Context:** NER tasks where you need structured output with entity type classification and contextual information.

---

### Use Case: Sentiment Analysis
**Prompt:**
```xml
<text>
The product was okay, but the customer service was terrible. I probably won't buy from them again.
</text>

Use the print_sentiment_scores tool.
```
**Purpose:** Analyzes sentiment and returns positive, negative, and neutral scores (0.0-1.0 range) as structured JSON. Forces quantitative sentiment analysis through tool schema.

**Context:** Converting qualitative text into quantitative sentiment metrics with normalized scores.

---

### Use Case: Text Classification
**Prompt:**
```xml
<document>
The new quantum computing breakthrough could revolutionize the tech industry.
</document>

Use the print_classification tool. The categories can be Politics, Sports, Technology, Entertainment, Business.
```
**Purpose:** Classifies text into predefined categories with confidence scores (0.0-1.0) for each category. Demonstrates multi-label classification with probability distribution.

**Context:** Categorizing content when you need confidence scores across multiple categories rather than single-label classification.

---

### Use Case: Open-Ended Characteristic Extraction
**Prompt:**
```
Given a description of a character, your task is to extract all the characteristics of the character and print them using the print_all_characteristics tool.

The print_all_characteristics tool takes an arbitrary number of inputs where the key is the characteristic name and the value is the characteristic value (age: 28 or eye_color: green).

<description>
The man is tall, with a beard and a scar on his left cheek. He has a deep voice and wears a black leather jacket.
</description>

Now use the print_all_characteristics tool.
```
**Purpose:** Extracts arbitrary key-value pairs when the exact schema is not known in advance. Uses `additionalProperties: True` in the tool schema to allow flexible JSON structure.

**Context:** Scenarios where the output structure cannot be predetermined and must adapt to the input content.

---

## Memory & Context Management

### Use Case: Code Review with Learning
**Prompt:**
```python
I'm reviewing a multi-threaded web scraper that sometimes returns fewer results than expected. The count is inconsistent across runs. Can you find the issue?

```python
{code_to_review}
```
```
**Purpose:** Demonstrates Claude analyzing code for concurrency issues (race conditions) and storing learned patterns in memory for future sessions. Shows cross-conversation learning capability.

**Context:** Building AI agents that learn debugging patterns over time and apply them to new code reviews automatically.

**System Prompt:**
```
You are a code reviewer.
```

---

### Use Case: Applying Learned Patterns
**Prompt:**
```python
Review this API client code:

```python
{code_to_review}
```
```
**Purpose:** Shows Claude checking memory first, finding previously learned concurrency patterns, and immediately applying them to new code without re-learning. Demonstrates knowledge persistence across conversations.

**Context:** Second session (new conversation) where Claude uses stored knowledge from previous reviews to accelerate analysis.

---

### Use Case: Long Review Session with Context Clearing
**Prompt:**
```python
Review this data processor:

```python
{data_processor_code}
```
```
**Purpose:** Demonstrates automatic context management during long sessions. Old tool results are cleared to save tokens while memory files persist, allowing Claude to maintain learned patterns while managing context limits.

**Context:** Multi-file code review sessions where context would otherwise overflow. Shows short-term vs long-term memory distinction.

---

## Parallel Tools

### Use Case: Forcing Parallel Tool Calls with Batch Tool
**Prompt:**
```
What's the weather and time in San Francisco?
```
**Purpose:** Demonstrates using a "batch tool" to encourage Claude 3.7 Sonnet to make multiple tool calls simultaneously rather than sequentially. The batch tool acts as a meta-tool that wraps multiple tool invocations.

**Context:** Optimizing latency when multiple independent tools can be called in parallel. Workaround for Claude 3.7 Sonnet's tendency toward sequential tool calls.

---

## Tool Choice

### Use Case: Auto Tool Choice with Guidance
**Prompt:**
```
What color is the sky?
```
**Purpose:** Tests Claude's ability to answer questions without tools when it has sufficient knowledge. With `tool_choice: auto`, Claude should not call the web_search tool for this query.

**System Prompt:**
```python
Answer as many questions as you can using your existing knowledge.
Only search the web for queries that you can not confidently answer.
Today's date is {date.today().strftime("%B %d %Y")}
If you think a user's question involves something in the future that hasn't happened yet, use the search tool.
```

**Context:** Preventing over-eager tool calling by providing clear guidance on when tools are necessary.

---

### Use Case: Auto Tool Choice - Requiring Tools
**Prompt:**
```
Who won the 2024 Miami Grand Prix?
```
**Purpose:** Tests Claude recognizing when it needs external information. With `tool_choice: auto`, Claude should call web_search for future events or information beyond its knowledge cutoff.

**Context:** Balancing between using existing knowledge and fetching new information based on query requirements.

---

### Use Case: Forcing Specific Tool
**Prompt:**
```
Holy cow, I just made the most incredible meal!
```
**Purpose:** Demonstrates `tool_choice: {"type": "tool", "name": "print_sentiment_scores"}` to force Claude to always use a specific tool regardless of the query content. Used for guaranteed structured output.

**Context:** When you need to ensure Claude calls a particular tool every time, such as for JSON extraction or mandatory processing steps.

---

### Use Case: Any Tool Choice
**Prompt:**
```
Hey there! How are you?
```
**Purpose:** With `tool_choice: {"type": "any"}`, Claude must call one of the available tools but can choose which. For an SMS bot with `send_text_to_user` and `get_customer_info` tools, Claude chooses to send a text response.

**System Prompt:**
```python
All your communication with a user is done via text message.
Only call tools when you have enough information to accurately call them.
Do not call the get_customer_info tool until a user has provided you with their username. This is important.
If you do not know a user's username, simply ask a user for their username.
```

**Context:** Ensuring Claude never returns raw text responses and always uses tools to communicate, useful for chatbots with specific output channels.

---

### Use Case: Any Tool - Smart Tool Selection
**Prompt:**
```
I need help looking up an order.  My username is jenny76
```
**Purpose:** Shows Claude selecting the appropriate tool (`get_customer_info`) when given sufficient information, while still being constrained to use one of the available tools via `tool_choice: any`.

**Context:** SMS chatbot that must always use tools but needs to intelligently choose which tool based on available information.

---

## Pydantic Integration

### Use Case: Structured Note Saving with Validation
**Prompt:**
```
Can you save a private note with the following details?
Note: Remember to buy milk and eggs.
Author: John Doe (johndoe@gmail.com)
Priority: 4
```
**Purpose:** Demonstrates using Pydantic models to validate tool input and output. The Note model includes nested Author object with email validation, priority range validation (1-5), and optional fields with defaults.

**Context:** Adding type safety and validation to tool use by defining Pydantic schemas for complex nested data structures with validation rules.

---

## Vision with Tools

### Use Case: Nutrition Label Extraction
**Prompt:**
```
Please print the nutrition information from this nutrition label image.
```
**Purpose:** Combines Claude's vision capabilities with tool use to extract structured data from images. The `print_nutrition_info` tool defines a schema for calories, total_fat, cholesterol, total_carbs, and protein.

**Context:** Extracting structured information from visual inputs like labels, forms, receipts, or documents. Shows multimodal input (image + text) with structured output (JSON via tool).

**Message Format:**
```python
{
    "role": "user",
    "content": [
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": base64_string}},
        {"type": "text", "text": "Please print the nutrition information from this nutrition label image."}
    ]
}
```

---

## Summary Statistics

**Total Prompts Found:** 23
**Main Use Cases Covered:**
1. Customer Service Agents (3 prompts)
2. Mathematical Calculations (3 prompts)
3. Structured JSON Extraction (5 prompts)
4. Memory & Learning (3 prompts)
5. Parallel Tool Execution (1 prompt)
6. Tool Choice Control (5 prompts)
7. Type Validation with Pydantic (1 prompt)
8. Vision + Tools (1 prompt)

**Key Pattern Categories:**
- Information retrieval and lookup
- Mathematical computation
- Structured data extraction
- Cross-conversation learning
- Tool execution control
- Input validation
- Multimodal analysis