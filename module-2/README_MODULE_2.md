## Module 2: Enhancing Customer Support Call Analysis with Cortex LLM Functions

### Scenario

Building upon the basic call transcript summarization in Module 1, this module focuses on enhancing the analysis by incorporating more advanced natural language processing techniques and features offered by Snowflake Cortex LLM functions.

### Objective

To develop a more comprehensive AI solution for analyzing customer support tickets that can:

*   Handle multilingual data.
*   Determine customer sentiment.
*   Summarize call transcripts concisely.
*   Classify call transcripts into predefined categories.
*   Estimate inference costs.
*   Gracefully handle potential processing failures.

### Key Concepts and Functions

1.  **Snowflake Cortex LLM Functions:** Overview of the three main categories:
    *   **Complete**:  Using this allows users to specify the LLM, and a prompt to accomplish tasks.
    *   **Task-Specific**: Out-of-the-box function for common tasks like translation, sentiment analysis, summarization, and text classification.
    *   **Helper**: Functions for estimating costs and handling potential errors (e.g., `count_tokens`, `try_complete`).

2.  **Cortex Complete:**

    *   LLMs can be invoked using the `complete` function, which takes in the model name and a prompt.

3.  **Choosing the right LLM:**
    -Different models range from smaller models to larger models
        -Larger Models- Great accuracy for complex tasks.
        -Smaller Models- Great for time-sensitive tasks.

4.  **Streamlining Function Calls**:
    * Cortex has an endpoint optimized for batch processing.
    * When Cortex LLMs are called in Python, on the other hand, they will hit an endpoint specifically optimized for low latency.
    * Task-specific LLM functions (Translate, Sentiment, Summarize and Classify Text)
  5. **Translate:**

    *   Enables easy translation of strings or columns in batch from one language to another.
    *   Automatically detects the source language if it's unknown.
    *   Example: Translating German call transcripts to English.

    ```
    from snowflake.ml.snowpark.llm import translate

    df_translated = df.filter(df["language"] == "German")
    df_translated["english_transcript"] = translate(text=df_translated["transcript"], source_language="de", target_language="en")
    ```

5.  **Sentiment:**

    *   Analyzes a text string or column and returns a sentiment value (negative, positive, or neutral).
    *   Useful for understanding customer emotions in feedback.
    *   Billed only for input tokens.

    ```
    from snowflake.ml.snowpark.llm import sentiment

    df["sentiment_score"] = sentiment(df["english_transcript"])
    ```

6.  **Summarize:**

    *   Generates a concise, high-level summary of a large body of text.
    *   Ideal for distilling key points from call transcripts.

    ```
    from snowflake.ml.snowpark.llm import summarize

    df["summary"] = summarize(df["english_transcript"])
    ```

7.  **Classify Text:**

    *   Classifies free-form text into categories that you provide (zero-shot classification).
    *   Useful for routing user queries or categorizing support tickets.
    *   Can include descriptions and examples for each label to improve performance.

    ```
    from snowflake.ml.snowpark.llm import classify_text

    categories = ["product defect", "shipping issue", "warranty claim"]
    df["category"] = classify_text(text=df["transcript"], classes=categories)
    ```

8.  **Helper Functions:**

    *   **countTokens:**  Calculates the total number of tokens in a prompt or input text. Essential for estimating costs and ensuring inputs don't exceed context window limits. Each model/function may have different tokenizers
    *   **tryComplete:** Works the same as `complete` but returns NULL instead of an error if the operation fails (e.g., input text exceeds context window). Prevents app breakage and isn't billed for token use when it returns NULL.

9.  **Role-Based Access Controls (RBAC):**

    *   The `CORTEX_USER` database role ensures only authorized users can use Cortex LLM functions or other Cortex features.

### Implementation

The implementation involves the following steps:

1.  **Setup Snowflake Environment:**
    *   Log in to Snowflake.
    *   Import required libraries (Snowflake Snowpark, Streamlit, etc.).
    *   Set the context to use the SkiGear support database and schema.

2.  **Data Preparation:**
    *   Ensure the call transcript data is loaded into a Snowflake table.
    *   The table should include columns for:
        *   `transcript` (the raw text of the call).
        *   `language` (the language of the transcript).

3.  **Enhance the Streamlit Application:**
    *   Modify the Streamlit app from Module 1 to:
        *   Allow users to select different LLM functions (translate, sentiment, summarize, classify text).
        *   Display the results of each function in the UI.

### Code Examples

The provided code snippets showcase how to use the various Cortex LLM functions in Python.  Adapt these examples to work with your specific data and requirements. For example:

Example function to analyze a single transcript
def analyze_transcript(transcript, language):
# Translate to English if necessary
if language != "en":
    translated_text = translate(text=transcript, source_language=language, target_language="en")
else:
    translated_text = transcript

# Perform sentiment analysis
sentiment_score = sentiment(translated_text)

# Summarize the transcript
summary = summarize(translated_text)

# Classify the transcript
categories = ["product defect", "shipping issue", "warranty claim"]
category = classify_text(text=translated_text, classes=categories)

return {
    "translated_text": translated_text,
    "sentiment_score": sentiment_score,
    "summary": summary,
    "category": category
}

Apply the analysis to each row in the DataFrame
df["analysis_results"] = df.apply(lambda row: analyze_transcript(row["transcript"], row["language"]), axis=1)

Print for review
df.show()


### Key Improvements and Outcomes

*   **Multilingual Support:** The application can now process call transcripts in various languages by automatically translating them to English before analysis.
*   **Sentiment Analysis:**  Understanding customer sentiment provides valuable insights into customer satisfaction levels.
*   **Efficient Summarization:** Summarizing transcripts saves time and effort in identifying the key issues.
*   **Automated Categorization:** Classifying transcripts into predefined categories helps route support tickets and identify common problems.
*   **Cost Optimization:** Using `countTokens` helps estimate and manage the costs associated with LLM function calls.
*   **Robust Error Handling:** `tryComplete` gracefully handles potential errors, preventing application failures.

By implementing these enhancements, the ski gear company can significantly improve its customer support operations and gain a deeper understanding of customer needs and concerns.
