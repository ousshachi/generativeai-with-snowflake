## Fine-Tuning Large Language Models in Snowflake

### Objectives

1.  Understand the benefits and rationale behind fine-tuning LLMs for specific tasks.
2.  Learn how to prepare and structure data for fine-tuning within the Snowflake environment.
3.  Execute and manage fine-tuning jobs using Snowflake's Cortex fine-tune function.
4.  Monitor the progress and status of fine-tuning jobs.
5.  Utilize fine-tuned models for inference and integrate them into applications.

### 1. Why Fine-Tune LLMs?

#### a. General Purpose vs. Specialized Models

*   General-purpose LLMs (e.g., ChatGPT) have broad knowledge but lack expertise in specific domains.
*   Fine-tuning transforms general models into highly skilled specialists, similar to a high school student attending trade school.

#### b. Prompt Engineering vs. Fine-Tuning

| Feature               | Prompt Engineering                                       | Fine-Tuning                                                                      |
| --------------------- | ------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Knowledge Scope       | Limited to original model's training data               | Can introduce new knowledge from fine-tuning data                                 |
| Customization         | Basic, limited by prompt design                        | Highly customizable to specific styles, formats, and tones                          |
| Accuracy              | Can be inconsistent, requires creative prompting       | Higher accuracy and consistency in the fine-tuned domain                            |
| Use Case              | Quick experiments, general tasks                         | Specialized tasks, consistent formatting requirements, domain-specific knowledge |

#### c. Fine-Tuning Benefits

*   **New Task Accomplishment**: Teaches the model to complete new tasks it couldn't do before.
*   **Response Style Customization**: Tailors the model's response style (brand voice, JSON format).
*   **Reliable Output Formatting**: Ensures consistent output formats (e.g., JSON, bullet points).
*   **Distillation:** Transfers the capabilities of a large model to a smaller, more cost-effective model.

#### d. Distillation

*  Transfers the capabilities of a large model to a smaller, more cost-effective model.
*  Leverages the benefits of the larger model at a reduced cost.
*  Large models are better for high-accuracy, low-volume tasks due to higher expense.
*  Fine-tuning a smaller model balances performance with cost-efficiency.

### 2. Snowflake Cortex Fine-tune Function

#### a. Key Features

*   Fully managed service for customizing LLMs with your own data.
*   Supports parameter-efficient fine-tuning (PEFT) to minimize costs and resources.
*   Uses PEFT under the hood.

#### b. Parameter Efficient Fine-Tuning (PEFT)

*   Improves performance on specific tasks by fine-tuning a small subset of parameters.
*   Reduces the number of parameters to tune and leverage examples.

#### c. Supported Models

*   Models from the LLAMA and Mistral families, ranging in size.
*   Model availability may vary by region; check Snowflake documentation.

### 3. Cost Considerations

#### a. Factors Influencing Fine-Tuning Costs

*   **Training Tokens**: Number of input tokens multiplied by the number of epochs.

*   **Epochs**: The number of times the model trains on the training data; controls the number of training cycles the model runs for

    *   Cortex automatically selects optimal epochs (1-10).
    *   More epochs may be needed for small datasets or high accuracy requirements.

*   **Additional Cost**: Costs are also incurred for warehouse usage, compute resources, and storage for custom adapters.

### 4. Permissions and Access Controls

#### a. Required Permissions

1.  **Snowflake Cortex User Role**: Necessary to invoke the `fine_tune` function. Ask your admin to grant this role.
2.  **Usage Privilege**: Needed for the database where the training/evaluation data is stored.
3.  **Create Model Privilege (or Ownership)**: Required to create and manage models within a specific schema.

### 5. Step-by-Step Fine-Tuning Process

#### a. Preparing the Data

1.  **Data Loading**:  Load customer support tickets from an S3 bucket into a Snowflake table.
2.  **Categorization**: Use the `classify_text` function from the previous module to classify customer support tickets into predetermined categories.
3.  **Initial Response Generation**: Use an LLM (Mistral 7B, Mistral Large) to generate initial custom responses for the support tickets based on their type and customer preferences.
4.   **Fine-Tuning Data**:
    *   Filter the Mistral Large responses that adhere to formatting guidelines.
    *   Create a table with columns for `ticket_id`, `prompt`, and `mistral_large_response`.
5.  **Splitting the Data**:  Split the data set into training (80%) and evaluation (20%) using the `random_split` function.

#### b. Running the Fine-Tuning Job

1.  **Invoke `fine_tune` Function**:
    *   From a Snowflake notebook or the AI/ML Studio, call the `fine_tune` function.

#### c. Monitor Progress

1.  **Using `SHOW` function**:  Use to list all fine-tuning jobs you have access to, and the status for those jobs.
2.  **Using `DESCRIBE` Function**:  Use to get back a lot of information about the model used, the database, schema, and model names, the progress of the job and its status.
3.   **Using AI/ML Studio Interface**: Use the no-code interface to streamline the LLM customization and AI app development.

from snowflake.ml.snowpark.llm import fine_tune

Example fine-tuning call
fine_tune(
create=True,
model_name='support_messages_fine_tuned_mistral_7b',
foundation_model='mistral-7b',
training_data=training_data,
evaluation_data=evaluation_data
)


#### d. Executing the Fine-Tuning Job

1.  You can start running fine-tuned jobs from Snowflake notebooks, or the Snowflake Studio Interface.
2.  The code to run this is:

from snowflake.ml.snowpark.llm import fine_tune

fine_tune(
create=True,
model_name="support_messages_fine_tuned_mistral_7b",
foundation_model="mistral-7b",
training_data=training_data,
evaluation_data=evaluation_data,
)


#### e. Managing the Job

1.  **To Check Progress in Snowflake Cortex**, pass the show argument to Snowflake Cortex:
    * This command allows you to see a list of all running/completed jobs.
2.  **Can also monitor it using `describe` function**

### 6. Monitoring and Managing Fine-Tuning Jobs

*  Fine-tuned jobs run in the background, so you'll use `show` and `describe` commands to track their status.

*  Job status is checked using Snowflake Cortex

*   Job status = pending, in progress, success, error, or canceled.

    #### a.  Managing the Job

    1.  **To Check Progress in Snowflake Cortex**, pass the show argument to Snowflake Cortex:
        * This command allows you to see a list of all running/completed jobs.
    2.  **Can also monitor it using `describe` function**

-- Show Command (SQL)
SELECT SYSTEM$GET_PREDICTIONS('show')

### Next Steps:

*   Incorporate the fine-tuned model to improve support tickets and streamline operations.
*   Reflect on the content taught in the course.

Overall Takeaways

Fine-tuning enables specialization and customization of LLMs for specific business needs.

Snowflake's Cortex fine-tune provides a managed and cost-effective way to train LLMs using your data.

Careful data preparation, cost management, and access control are essential for successful fine-tuning.