 # AI Security on AWS Workshop

 This repository is based on this [workshop by AWS](https://catalog.us-east-1.prod.workshops.aws/workshops/0720c7c4-fb23-4e43-aa9f-036fc07f46b2).

 ## Instructions

 Ask the Workshop instructors to provide you with a temporary AWS user. The base infrastructure will already be deployed, so you can play with the Sample Web App (playground app) and start adding guardrails.

 The playground app will be already deployed in a public URL, ask the Workshop instructors for it so you can start experimenting with it.

### Guardrail Creation

 Go to Guardrails for Bedrock console, and choose **Create guardrail**.

Enter the following information:

Name: `fiduciary-advice`

Description: `Prevents the model from providing fiduciary advice.`

Messaging for blocked prompts: `I can provide general info about Acme Financial's products and services, but can't fully address your request here. For personalized help or detailed questions, please contact our customer service team directly. For security reasons, avoid sharing sensitive information through this channel. If you have a general product question, feel free to ask without including personal details.`

Choose **Next**.

On **Configure** content filters page:

Turn on **Enable harmful categories filters**

Turn on **Enable prompt attacks filter**

Choose **Next**.

On **Add denied topics** page, choose **Add denied topic**.

Enter the following information:

Name: `Fiduciary Advice`

Definition for topic: `Providing personalized advice or recommendations on managing financial assets, investments, or trusts in a fiduciary capacity or assuming related obligations and liabilities.`

Expand **Add sample phrases - optional**, and add the following examples.

Click **Add phrase** for each sentence:

`What stocks should I invest in for my retirement?`

`Is it a good idea to put my money in a mutual fund?`

`How should I allocate my retirement savings plan investments?`

`What type of trust fund should I set up for my children?`

`Should I hire a financial advisor to manage my investments? denied-topic`

`Choose Confirm, and then choose Next.`

On **Add word filters** page, select **Filter profanity**.

Scroll down to **View and edit words and phrases** panel, choose button **Add**, and choose **Add words and phrases in bulk**.

Enter the following phrases:

```
fiduciary advice
investment recommendations
stock picks
financial planning guidance
portfolio allocation advice
retirement fund suggestions
wealth management tips
trust fund setup
investment strategy
financial advisor recommendations
```

Choose **Confirm**, and then choose **Next**.

On **Add sensitive information filters** page, choose **Add new PII**.

Configure the following behaviors.

PII type	Guardrails behavior

- Name ->	Mask
- Phone ->	Mask
- Email ->	Mask
- Credit/Debit card number	-> Block
- Credit/Debit card expiry	-> Block
- CVV ->	Block

Scroll down to **Regex patterns**, choose **Add regex pattern**.

Enter the following information:

Name: `Account Number`

Regex pattern: `\b[A-Za-z]{6}\d{4}\b`

Guardrail behavior: `Mask`

Add description - optional: `Matches account numbers in the format XXXXXX1234`

Choose **Confirm**, and then choose **Next**.

On **Add contextual grounding check** page, turn on both **Enable grounding check** and **Enable relevance check**. Choose **Next**. 

On **Review** and **create** page, choose **Create guardrail**.

Take note the guardrail ID in Guardrail Overview panel, we will need it later.

Scroll down to **Versions** panel at the bottom of the page, and choose **Create version**. Enter `First guardrails version` for **Description**, and choose **Create version**.

### Adding guardrail to Lambda

Open **Lambda** console, and choose function that ends with **guardrails-backend-lambda**.

Scroll down to the **Code source** pane. Open file `lambda_function.py` from the left sidebar. On the code block below Enable Guardrail comment, enter your guardrail ID and version as appropriate.

```python
# Enable Guardrail
guardrail_id = "i1smdsl7puon" # replace with your own
guardrail_version = "1" # replace with your own
```

Read the sample implementation in file `inference/converse.py`. Pay attention to Guardrail configuration, and how easy it is to attach Guardrail to LLM invocation. Once you understand, without changing anything, choose **Deploy** on top of the **Code Source** pane.

```python
# If Guardrail has been configured, include it in Converse API to evaluate prompt and response
if guardrail_enabled:
  guardrail_config = {
    "guardrailIdentifier": guardrail_id,
    "guardrailVersion": guardrail_version,
    "trace": "enabled"
  }
  kwargs["guardrailConfig"] = guardrail_config

# Invoke LLM
print(kwargs)
response = bedrock_runtime.converse(**kwargs)
print(response)
```

### Testing Guardrails

Go back to the playground app, try to load the following prompts: Harmful categories, Prompt attacks, Denied topics (input), Denied topics (output), Profanity, Word filters, Sensitive information. Choose Submit on each prompt and observe how Guardrail works.

Guardrails for Amazon Bedrock supports contextual grounding check to detect and filter hallucinations in model responses when a reference source and a user query is provided. Contextual grounding check evaluates for hallucinations across two paradigms:

 - **Grounding** – This checks if the model response is factually accurate based on the source and is grounded in the source. Any new information introduced in the response will be considered un-grounded.
- **Relevance** – This checks if the model response is relevant to the user query.

To use contextual grounding with Converse API:

First, convert our message content formatting by specifying guardContent (GuardrailConverseContentBlock) field. Specifying guardContent is useful to do selective evaluation, for example if we want to have guardrail assess the latest message in a conversation. Read more about guardContent at Guarding a message and Guarding a system prompt sections at AWS docs Use a guardrail with the Converse API .
Next, add qualifiers field to the guardContent field. Include both the query qualifier (for the prompt) and grounding_source qualifier (for source).

Go back to **Lambda** console, read the sample implementation for formatting the message in file `helpers/format.py`.

```python
messages = [{
  "role": "user",
  "content": [
    {
      "guardContent": {
        "text": {
          "text": source,
          "qualifiers": ["grounding_source"]
        }
      }
    },
    {
      "guardContent": {
        "text": {
          "text": prompt,
          "qualifiers": ["query"]
        }
      }
    }
  ]
}]
```

Go back to the playground app, load the sample prompts for **Grounding & relevance** check (both blocked and passed). Choose Submit on each prompt and observe how Guardrail works.

### Cleanup

Open **Amazon Bedrock** console.
Delete your guardrails.

 
