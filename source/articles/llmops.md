# LLMOps: what changes for large language models

**Covers:** prompt versioning, LLM-as-judge, token cost monitoring, LLM, RAG

## The same loop, with new parts {#overview}
Large language models change the practice, but not the idea. You still need to version what you ship, test it, release it carefully and watch it. What changes is what the "model" is made of. Often you do not train anything, and the behavior comes from a prompt, some retrieved documents and a model run by someone else.

## LLM {#llm}
An LLM, a large language model, is a neural network trained on huge amounts of text to predict and generate language. Examples are the models behind chat assistants. Most teams use one through a service, and pay by usage, rather than training their own.

## RAG {#rag}
RAG, retrieval-augmented generation, is a method in which a model looks up relevant documents first and uses them to write its answer. A support assistant, for instance, searches the company's help pages for the customer's question, and then answers based on what it found. It keeps answers up to date without retraining, and ties them to sources.

## Prompt versioning {#prompt-versioning}
Prompt versioning means tracking changes to a prompt over time, the same way you would track changes to code or a registered model, so that you know exactly which prompt text produced which behavior. A one-word change can alter the output a lot, so a prompt belongs in version control, with a note on why it changed.

## LLM-as-judge {#llm-as-judge}
LLM-as-judge means using one language model to score the quality of another's output, instead of comparing against one fixed expected answer. It is useful because the same question can have many good answers, so exact matching fails. The judge is given the question, the answer and a scoring guide. Its scores must themselves be checked against human ratings from time to time, since judges have biases.

## Token cost monitoring {#token-cost-monitoring}
Token cost monitoring means tracking how much each model call actually costs. Usage is billed by tokens, small pieces of text, both in and out. A subtle change in the prompt, or a longer conversation history, can multiply the bill without any visible change in quality.

## A worked example {#example}
A company runs a support assistant using RAG.

**Prompt versioning.** The prompt is stored in the repository as `support_prompt.txt`. Version 7 adds the sentence "Answer only from the provided documents". Version 8 adds a long list of examples. Each version has a commit and a short note. When quality drops, the team compares versions 7 and 8 in minutes.

**Evaluation with a judge.** The team keeps 200 test questions. For each version, the judge model scores answers from 1 to 5 on correctness and on "uses only the documents". Results:

| Prompt | Average score | Share scoring 4 or 5 |
|---|---|---|
| Version 7 | 4.1 | 82% |
| Version 8 | 4.3 | 88% |

Version 8 looks better. But first check the judge: a human rates a sample of 50 answers. The humans and the judge agree within one point on 46 of 50 (92%), so the judge can be trusted for this task. Had the agreement been 60%, the scores would not mean much.

**Cost.** Each call to version 7 uses 1,200 input tokens and 300 output tokens. Version 8, with its examples, uses 2,000 input and 300 output. Suppose the price is 3 dollars per million input tokens and 15 dollars per million output tokens, and there are 100,000 calls a day.

- Version 7: input 1,200 × 100,000 = 120 million tokens, costing 360 dollars; output 300 × 100,000 = 30 million tokens, costing 450 dollars. Total 810 dollars a day.
- Version 8: input 2,000 × 100,000 = 200 million tokens, costing 600 dollars; output the same 450. Total 1,050 dollars a day.

The quality gain of 6 points in the share of good answers costs 240 dollars more a day, about 7,200 a month. Is it worth it? That is a business decision, and only possible because both numbers are measured. Monitoring also catches accidents: a bug that sends the whole conversation history on every call could triple the input tokens overnight.

RAG adds a new place to look. When an answer is wrong, the cause may be that the retrieval step found the wrong documents. Evaluate retrieval on its own, for example by checking whether the right page appears among the top 5 results for each test question.

| Classical MLOps | LLMOps equivalent |
|---|---|
| Model version | Prompt version plus model name |
| Accuracy on labels | Judge scores and human spot checks |
| Compute cost | Token cost |
| Feature store | Document index for RAG |

## A common mistake {#mistake}
Changing a prompt in a web console with no record, so no one can say what changed when quality falls. Another mistake is trusting a judge model without ever comparing it to people.

## Key points {#key-points}
- Treat prompts like code: version them.
- An LLM judge scales evaluation, but must be checked against humans.
- Track tokens, since cost can change without a visible change in quality.
- With RAG, evaluate the retrieval step separately.
