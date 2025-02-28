# Module 2 

This course introduces three types of LLM functions available in Snowflake Cortex:

1- Cortex Complete Function:
   - Generates text using models like Claude 3.5 Sonnet.
   - Requires a well-structured prompt for optimal performance.

2- Task-Specific Functions (pre-built for common NLP tasks):

   -  Translate: Converts text between languages (supports auto-detection of source language).
   -  Sentiment Analysis: Assigns a sentiment score (-1 to 1) to text.
   -  Summarize: Creates a concise, high-level summary of a text.
   -  ClassifyText: Categorizes text based on predefined labels, enabling zero-shot classification.

3- Helper Functions

   -  TryComplete: Works like a try-except block, returning null instead of an error if an LLM operation fails.
   - CountTokens: Calculates token usage, helping estimate Snowflake credits consumption.
  
Access control using Cortex User Database Role, ensuring only authorized users can access Cortex LLM functions.
-- -------------------------------------------------------------------------------------------------------------
# 1- Cortex Complete - Multi-Turn Chat Application

## Overview
This document provides an in-depth guide on using the `Cortex Complete` function to build a multi-turn chat application in Snowflake.

## Prerequisites
Before getting started, ensure you have the following:
- A Snowflake account
- Access to Snowflake Notebooks
- Installed packages: `snowflake`, `snowflake-ml-python`

## Getting Started

### 1. Importing Required Packages
First, import the active Snowpark session and set the context to the appropriate database and schema:

```python
from snowflake.snowpark import Session

# Set database and schema context
session.sql("USE DATABASE SkiGear_Support_DB").collect()
session.sql("USE SCHEMA SkiGear_Support_Schema").collect()
```

### 2. Using `complete` Function
The `complete` function is called using Python and can be used with different models. For multilingual applications, the `LLAMA 3.1 405 billion` parameter model is recommended. It supports multiple languages including:
- French, German, Spanish, Italian, Portuguese
- Arabic, Hindi, Russian, Chinese, Japanese, Korean

For summarization or large-context applications, consider:
- `Claude 3.5 Sonnet` (200K token context window)
- `Jamba Instruct` (256K token context window)

For low-latency tasks requiring high throughput, smaller models like `LLAMA 3.2 1B` or `3B` parameters are more suitable.

**Example Usage:**

```python
response = session.sql(
    """
    SELECT AI.COMPLETE(
        model => 'llama-3-1-405b',
        prompt => 'How do snowflakes get their unique patterns?'
    ) AS response
    """
).collect()
```

### 3. Stateless Nature of `complete`
`complete` is **stateless**, meaning it does not retain memory across calls. To create a multi-turn conversation, pass previous inputs in chronological order:

```python
conversation_history = [
    {"role": "system", "content": "You are an AI assistant."},
    {"role": "user", "content": "What is the role of semicolons in JavaScript?"},
    {"role": "assistant", "content": "Semicolons help separate statements."}
]

response = session.sql(
    """
    SELECT AI.COMPLETE(
        model => 'llama-3-1-405b',
        messages => PARSE_JSON(:conversation_history)
    ) AS response
    """,
    params={"conversation_history": conversation_history}
).collect()
```

### 4. Role Definitions in `complete`
Each `message` object must have:
- `role`: Defines the type of message (`system`, `user`, or `assistant`)
- `content`: The actual message text

### 5. Enabling Streaming Responses
For real-time responses, use **streaming mode**, which provides output token-by-token:

```python
response = session.sql(
    """
    SELECT AI.COMPLETE(
        model => 'llama-3-1-405b',
        prompt => 'Explain recursion in simple terms.',
        stream => TRUE
    ) AS response
    """
).collect()
```

> **Note:** Streaming is available in Python but not in SQL.

### 6. Adjusting Model Output
You can control the behavior of the model using the following parameters:
- **`temperature`**: Controls randomness (range 0-1). Higher values = more diverse responses.
- **`topP`**: Restricts token generation to a cumulative probability.
- **`maxTokens`**: Limits the number of generated tokens.

Example of setting a high temperature for creativity:

```python
response = session.sql(
    """
    SELECT AI.COMPLETE(
        model => 'llama-3-1-405b',
        prompt => 'Explain JavaScript like a cowboy.',
        temperature => 0.9
    ) AS response
    """
).collect()
```

### 7. Batch Processing with SQL
For batch processing, you can use `complete` on a full column in a Snowflake table:

```sql
SELECT AI.COMPLETE(
    model => 'llama-3-1-405b',
    prompt => transcript_column
) AS ai_response
FROM call_transcripts;
```

### 8. Response Format
When `complete` is called with `options`, it returns a JSON response with keys:
- **`choices`**: Array containing the model’s response
- **`created`**: Timestamp of response generation
- **`model`**: The model used
- **`usage`**: Token consumption details (prompt, completion, total tokens)

```json
{
  "choices": [
    {
      "messages": [{"role": "assistant", "content": "Snowflakes form unique patterns due to molecular bonds."}]
    }
  ],
  "created": 1700000000,
  "model": "llama-3-1-405b",
  "usage": {
    "completion_tokens": 50,
    "prompt_tokens": 10,
    "total_tokens": 60
  }
}
```

### 9. Choosing the Right Model
When selecting a model, consider:
- **Large models** for complex tasks (LLAMA 3.1 405B, Claude 3.5 Sonnet)
- **Small models** for simple tasks with low latency (LLAMA 3.2 1B, 3B)
- **Long context models** when working with extensive input (Jamba Instruct, Claude 3.5)

## Summary
- `complete` generates text-based responses using LLMs.
- Use `temperature` and `topP` to fine-tune creativity.
- Use system, user, and assistant roles to create context-aware conversations.
- Enable streaming for real-time responses.
- Select models based on task complexity, latency, and budget constraints.

## Next Steps
- Explore `Cortex` task-specific functions for optimized performance.
- Implement batch processing for large-scale inference.
- Optimize token usage to control cost and performance.

---

For more information, visit [Snowflake Cortex Documentation](https://docs.snowflake.com/en/developer-guide/cortex).

# Task-Specific LLM Functions

## Overview
Welcome back! Here’s a quick update on where we are in this module. We introduced three types of LLM functions:

1. **Complete**
2. **Task-Specific**
3. **Helper Functions**

We previously reviewed each of these types at a high level and explored the **Complete** function with hands-on work. Now, we’ll dive deeper into **Task-Specific Functions** with practical examples.

## Task-Specific Functions
Task-Specific functions are designed to handle common AI tasks efficiently. These include:

- **Translate**: Convert text from one language to another.
- **Sentiment**: Analyze sentiment on a scale from -1 (negative) to 1 (positive).
- **Summarize**: Condense large text into key insights.
- **Classify Text**: Categorize text into predefined labels.

### Translate Function
The `translate` function processes an entire column of multilingual text and converts it into a target language.

#### Example in Python:
```python
translated_text = translate(data['transcript'], target_language='en')
```

#### Example in SQL:
```sql
SELECT translate(transcript, '') AS translated_text FROM call_transcripts;
```
Using an empty string (`''`) lets the function infer the source language automatically.

### Sentiment Analysis
The `sentiment` function determines sentiment scores for a given text.

#### Example in Python:
```python
sentiment_score = sentiment(data['transcript'])
```

#### Example in SQL:
```sql
SELECT sentiment(transcript) AS sentiment_score FROM call_transcripts;
```
The function provides a floating-point value between -1 and 1, where:
- **1** represents a highly positive sentiment
- **-1** represents a highly negative sentiment

### Summarization
The `summarize` function extracts key points from long-form text, such as meeting notes or call transcripts.

#### Example in Python:
```python
summary = summarize(data['transcript'])
```

#### Example in SQL:
```sql
SELECT summarize(transcript) AS summary FROM call_transcripts;
```
Summarizing call transcripts helps customer service teams save time and extract actionable insights.

### Text Classification
The `classify_text` function categorizes text based on predefined labels.

#### Example in Python:
```python
classification = classify_text(data['transcript'], categories=['recommendation', 'complaint', 'inquiry'])
```

#### Example in SQL:
```sql
SELECT classify_text(transcript, ['recommendation', 'complaint', 'inquiry']) AS category FROM call_transcripts;
```
If no category matches, the model may return an `unclassified` label.

## Conclusion
Task-Specific functions streamline AI workflows by eliminating the need for prompt engineering and model selection. They work **out of the box** to provide:
- Seamless translation
- Quick sentiment analysis
- Efficient summarization
- Accurate text classification

In the next section, we will explore **Helper Functions**, which assist in both early prototyping and production-level applications.

Stay tuned!
