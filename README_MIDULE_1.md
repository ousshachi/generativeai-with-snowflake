## Module 1: Building a Simple AI App with Snowflake
The course describes a specific use case for a simple AI application in the context of a ski gear company. Here are the key details:

### Scenario
Use Case: Customer Support Call Analysis
Scenario: You are a data scientist at a ski gear company with customers across different countries. The task is to analyze customer support call transcripts to identify top customer complaints.
This use case demonstrates how to quickly build a simple AI application that can extract structured information from unstructured text data, providing valuable insights for customer support analysis in the ski gear industry1

### Data

Raw, unstructured call transcript data:
- Date of customer support call
- Country and language of the customer
- Product name and category
- Type of damage to the product
- Transcript of the conversation between the customer and support representative

### Objective

Use natural language processing to process call transcripts and find the root cause of each complaint.

### Steps

1.  **Load Data into Snowflake:**
    -   Retrieve raw customer support call transcripts from an AWS S3 bucket.
    -   Import the data into a Snowflake table.

2.  **Analyze Call Transcripts using Snowflake Cortex LLM Functions:**
    -   Use Cortex LLM functions to perform common LLM tasks such as summarization.

3.  **Deliver Insights through an AI App:**
    -   Build a simple UI using Streamlit to present the summarized information.

### AI Application Functionality:

Summarize call transcripts into JSON format with three key fields:
- Product name
- Defect

### Technologies Used:
- Snowflake Cortex LLM functions (specifically the 'complete' function)
- Foundation models: LLAMA 3.2 and Mistral-7B
- Streamlit for building a simple UI
- Python and SQL for data processing and interaction with Snowflake

### Detailed Steps and Code Snippets

1.  **Import Libraries and Create Snowpark Session:**

    -   Import `pandas` and `streamlit` packages.
    -   Create a Snowpark session to process the call transcripts.
    ```
    import pandas as pd
    import streamlit as st
    from snowflake.snowpark import Session
    # Create Snowpark session (example, adjust as needed)
    session = Session.builder.configs(connection_parameters).create()
    ```

2.  **Create Snowflake Objects (Database, Warehouse, Schema):**

    -   Use SQL to create a new Snowflake database, virtual warehouse, and schema to store the call transcript data.
    ```
    -- SQL cell in Snowflake notebook
    CREATE OR REPLACE DATABASE SkiGearSupportDB;
    CREATE OR REPLACE WAREHOUSE ComputeWH WAREHOUSE_SIZE = 'XSMALL';
    CREATE OR REPLACE SCHEMA SkiGearSupportSchema;
    USE DATABASE SkiGearSupportDB;
    USE SCHEMA SkiGearSupportSchema;
    ```

3.  **Create CSV File Format and Load Data:**

    -   Define a CSV file format object to read CSV files from S3.
    -   Use an external stage in Snowflake to point to the S3 bucket.
    -   Create a table (`call_transcripts`) in Snowflake to load raw transcript data.
    -   Use the `COPY INTO` command to bulk copy data from the external stage into the Snowflake table.
    ```
    -- SQL cell in Snowflake notebook
    CREATE OR REPLACE FILE FORMAT csv_format
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;

    CREATE OR REPLACE STAGE s3_stage
    URL = 's3://your-s3-bucket/path/'
    CREDENTIALS = (AWS_KEY_ID = 'YOUR_AWS_KEY' AWS_SECRET_KEY = 'YOUR_AWS_SECRET');

    CREATE OR REPLACE TABLE call_transcripts (
        call_date DATE,
        country VARCHAR(50),
        language VARCHAR(50),
        product_name VARCHAR(100),
        product_category VARCHAR(100),
        damage_type VARCHAR(100),
        transcript VARCHAR(1000)
    );

    COPY INTO call_transcripts
    FROM @s3_stage
    FILE_FORMAT = (FORMAT = csv_format);
    ```

4.  **Sample Transcript and Snowflake Cortex LLM Function (Complete):**

    -   Use the Snowflake Cortex LLM function named `complete` to process and summarize the transcript.
    -   Import the `complete` function and create a prompt instructing the large language model to summarize the call transcript into JSON format.
    ```
    from snowflake.ml.snowpark.llm import complete

    prompt_template = """
    Summarize the following call transcript into JSON format with the following keys:
    - product_name: The name of the product.
    - defect: The specific defect or issue reported.
    - summary: A brief summary of the call.

    Transcript:
    {transcript}
    """
    ```

5.  **Invoke the LLM and Summarize the Transcript:**

    -   Invoke a foundation model (e.g., LLAMA 3.2) and pass the prompt and call transcript to the `complete` function.
    ```
    sample_transcript = "Example transcript from call_transcripts table"

    prompt = prompt_template.format(transcript=sample_transcript)

    response = complete(
        session=session,
        prompt=prompt,
        model="snowflake-llama3-2b-instruct"  # Specify LLM name as a string
    )

    print(response)
    ```

6.  **Experiment with Different LLMs:**

    -   Switch to a different language model (e.g., Mistral 7B) by changing the argument of the `complete` function.
    ```
    response = complete(
        session=session,
        prompt=prompt,
        model="snowflake-mistral-7b"  # Specify LLM name as a string
    )

    print(response)
    ```

7.  **Build Streamlit UI:**

    -   Wrap the `complete` function call into a `summarize` function in Python.
    -   Use Streamlit to build a UI for the summarization app with a text box and a summarize button.
    ```
    import streamlit as st
    from snowflake.ml.snowpark.llm import complete

    def summarize(transcript):
        prompt = prompt_template.format(transcript=transcript)
        response = complete(
            session=session,
            prompt=prompt,
            model="snowflake-mistral-7b"  # Specify LLM name as a string
        )
        return response

    st.title("Call Transcript Summarizer")
    transcript_input = st.text_area("Enter Call Transcript:", height=200)

    if st.button("Summarize"):
        if transcript_input:
            summary = summarize(transcript_input)
            st.json(summary)  # Display the summary
        else:
            st.warning("Please enter a call transcript.")

    ```

### Bonus

-   **Invoke Cortex Complete Function with SQL:**
    ```
    -- Snowflake SQL
    SELECT SNOWFLAKE.ML.SNOWPARK.LLM.COMPLETE(
      MODEL_NAME => 'snowflake-mistral-7b',
      INPUT => 'Summarize this call transcript: ...'
    );
    ```

### Key Takeaways

-   Easy integration of Snowflake and AI using Cortex LLM functions.
-   Ability to quickly analyze unstructured data (call transcripts).
-   Use of Snowpark sessions for efficient data processing.
-   Flexibility in experimenting with different foundation models.
-   Simple UI creation using Streamlit.
