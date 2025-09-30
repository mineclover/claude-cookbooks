# Classification Prompts for Insurance Support Tickets

This document contains all AI prompts extracted from the classification cookbook examples, organized by their purpose and use case.

---

## 1. Simple Classification (Zero-Shot)

**Prompt:**
```
You will classify a customer support ticket into one of the following categories:
<categories>
    <category>
        <label>Billing Inquiries</label>
        <content> Questions about invoices, charges, fees, and premiums Requests for clarification on billing statements Inquiries about payment methods and due dates
        </content>
    </category>
    <category>
        <label>Policy Administration</label>
        <content> Requests for policy changes, updates, or cancellations Questions about policy renewals and reinstatements Inquiries about adding or removing coverage options
        </content>
    </category>
    <category>
        <label>Claims Assistance</label>
        <content> Questions about the claims process and filing procedures Requests for help with submitting claim documentation Inquiries about claim status and payout timelines
        </content>
    </category>
    <category>
        <label>Coverage Explanations</label>
        <content> Questions about what is covered under specific policy types Requests for clarification on coverage limits and exclusions Inquiries about deductibles and out-of-pocket expenses
        </content>
    </category>
    <category>
        <label>Quotes and Proposals</label>
        <content> Requests for new policy quotes and price comparisons Questions about available discounts and bundling options Inquiries about switching from another insurer
        </content>
    </category>
    <category>
        <label>Account Management</label>
        <content> Requests for login credentials or password resets Questions about online account features and functionality Inquiries about updating contact or personal information
        </content>
    </category>
    <category>
        <label>Billing Disputes</label>
        <content> Complaints about unexpected or incorrect charges Requests for refunds or premium adjustments Inquiries about late fees or collection notices
        </content>
    </category>
    <category>
        <label>Claims Disputes</label>
        <content> Complaints about denied or underpaid claims Requests for reconsideration of claim decisions Inquiries about appealing a claim outcome
        </content>
    </category>
    <category>
        <label>Policy Comparisons</label>
        <content> Questions about the differences between policy options Requests for help deciding between coverage levels Inquiries about how policies compare to competitors' offerings
        </content>
    </category>
    <category>
        <label>General Inquiries</label>
        <content> Questions about company contact information or hours of operation Requests for general information about products or services Inquiries that don't fit neatly into other categories
        </content>
    </category>
</categories>

Here is the customer support ticket:
<ticket>
    {{ticket}}
</ticket>

Respond with just the label of the category between category tags.
```

**Purpose:**
Basic zero-shot classification approach where Claude categorizes customer support tickets into predefined categories without any examples. This is the baseline approach that achieved ~70% accuracy.

**Assistant Prefill:**
```
<category>
```

**Stop Sequence:** `</category>`

**Configuration:**
- Model: claude-3-haiku-20240307
- Temperature: 0.0
- Max Tokens: 4096

---

## 2. RAG-Enhanced Classification (Few-Shot with Retrieval)

**Prompt:**
```
You will classify a customer support ticket into one of the following categories:
<categories>
    {{categories}}
</categories>

Here is the customer support ticket:
<ticket>
    {{ticket}}
</ticket>

Use the following examples to help you classify the query:
<examples>
    {{examples}}
</examples>

Respond with just the label of the category between category tags.
```

**Examples Format:**
```xml
<example>
    <query>
        "[example query text]"
    </query>
    <label>
        [example label]
    </label>
</example>
```

**Purpose:**
Leverages Retrieval-Augmented Generation (RAG) to provide Claude with semantically similar examples from the training data. The system:
1. Embeds the query using VoyageAI's voyage-2 model
2. Searches a vector database for the top 5 most similar examples (similarity threshold: 0.75 in notebook, 0.85 in evaluation)
3. Includes these examples in the prompt to guide classification

This approach improved accuracy to ~92-94%, demonstrating the power of few-shot learning with relevant examples.

**Assistant Prefill:**
```
<category>
```

**Stop Sequence:** `</category>`

**Configuration:**
- Model: claude-3-haiku-20240307
- Temperature: 0.0
- Max Tokens: 4096
- RAG: Top 5 similar examples from vector database
- Embedding Model: voyage-2 (VoyageAI)

---

## 3. RAG with Chain-of-Thought Classification (Most Advanced)

**Prompt:**
```
You will classify a customer support ticket into one of the following categories:
<categories>
    {{categories}}
</categories>

Here is the customer support ticket:
<ticket>
    {{ticket}}
</ticket>

Use the following examples to help you classify the query:
<examples>
    {{examples}}
</examples>

First you will think step-by-step about the problem in scratchpad tags.
You should consider all the information provided and create a concrete argument for your classification.

Respond using this format:
<response>
    <scratchpad>Your thoughts and analysis go here</scratchpad>
    <category>The category label you chose goes here</category>
</response>
```

**Purpose:**
Combines RAG with Chain-of-Thought (CoT) reasoning. This approach:
1. Uses the same RAG retrieval mechanism as approach #2
2. Asks Claude to explicitly reason through the classification in a scratchpad
3. Requires Claude to create a concrete argument before making the final classification

This is the most accurate approach, achieving ~95.6% accuracy. The reasoning step helps Claude make more nuanced distinctions between similar categories.

**Assistant Prefill:**
```
<response><scratchpad>
```

**Stop Sequence:** `</category>`

**Response Extraction:**
The category is extracted from the response by splitting on `<category>` tags.

**Configuration:**
- Model: claude-3-haiku-20240307
- Temperature: 0.0 (best performing)
- Max Tokens: 4096
- RAG: Top 5 similar examples from vector database
- Embedding Model: voyage-2 (VoyageAI)

---

## Performance Comparison

Based on the evaluation results with 68 test cases:

| Approach | Accuracy Range | Best Configuration |
|----------|---------------|-------------------|
| Simple Classification | 69-72% | T=0.0, 0.2, 0.6, 0.8 |
| RAG Classification | 89-94% | T=0.0, 0.4, 0.6 |
| RAG + Chain-of-Thought | 94-96% | T=0.0, 0.2, 0.8 |

**Winner:** RAG with Chain-of-Thought at T=0.0 achieved 95.588235% accuracy.

---

## Categories Used in Classification

The system classifies insurance support tickets into 10 categories:

1. **Billing Inquiries** - Questions about invoices, charges, fees, and premiums
2. **Policy Administration** - Requests for policy changes, updates, or cancellations
3. **Claims Assistance** - Questions about the claims process and filing procedures
4. **Coverage Explanations** - Questions about what is covered under specific policies
5. **Quotes and Proposals** - Requests for new policy quotes and price comparisons
6. **Account Management** - Requests for login credentials or account updates
7. **Billing Disputes** - Complaints about unexpected or incorrect charges
8. **Claims Disputes** - Complaints about denied or underpaid claims
9. **Policy Comparisons** - Questions about differences between policy options
10. **General Inquiries** - General questions that don't fit other categories