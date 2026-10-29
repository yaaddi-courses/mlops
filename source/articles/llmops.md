# LLMOps: what changes for large language models

**Covers:** prompt versioning, LLM-as-judge, token cost monitoring, LLM, RAG

## The same loop, with new parts {#overview}
Large language models change the practice, but not the idea. You still version what you ship, test it, release it carefully and watch it. What changes is what the "model" is made of. Often you train nothing at all, and the behavior comes from a prompt, some retrieved documents and a model that someone else operates. Let's go through the new parts and see how each one fits the loop you already know.

## LLM {#llm}
An LLM, a large language model, is a neural network trained on huge amounts of text to predict and generate language. The models behind chat assistants are examples. Most teams use one through a service and pay by usage, instead of training their own.

## RAG {#rag}
RAG stands for retrieval-augmented generation. The model first looks up relevant documents and then uses them to write its answer. Picture a support assistant that searches the company's help pages for the customer's question and answers from what it found. This keeps answers up to date without retraining, and ties them to sources.

## Prompt versioning {#prompt-versioning}
With an LLM, the prompt is part of your product. Prompt versioning means tracking its changes over time, the way you track code or a registered model, so you know exactly which prompt text produced which behavior. A one-word change can alter the output a lot, so a prompt belongs in version control, with a note on why it changed.

## LLM-as-judge {#llm-as-judge}
How do you score answers that have no single correct form? The same question can have many good answers, so exact matching fails. In LLM-as-judge, one language model scores the quality of another's output. The judge is given the question, the answer and a scoring guide. Judges have biases, so you must check their scores against human ratings from time to time.

## Token cost monitoring {#token-cost-monitoring}
Usage is billed by tokens, small pieces of text, both going in and coming out. Token cost monitoring tracks what each model call really costs. A subtle change in the prompt, or a longer conversation history, can multiply the bill with no visible change in quality.

## A worked example {#example}
Let's follow a company that operates a support assistant using RAG.

**Prompt versioning.** The prompt lives in the repository as `support_prompt.txt`. Version 7 adds the sentence "Answer only from the provided documents". Version 8 adds a long list of examples. Each version has a commit and a short note. When quality drops, the team compares versions 7 and 8 in minutes.

**Evaluation with a judge.** The team keeps 200 test questions. For each version, the judge model scores answers from 1 to 5 on correctness and on "uses only the documents". The results:

| Prompt | Average score | Share scoring 4 or 5 |
|---|---|---|
| Version 7 | 4.1 | 82% |
| Version 8 | 4.3 | 88% |

Version 8 looks better. Before you trust that, check the judge. A human rates a sample of 50 answers, and the humans and the judge agree within one point on 46 of the 50, which is 92%. The judge is good enough for this task. Had the agreement been 60%, the scores would not mean much.

**Cost.** Each call to version 7 uses 1,200 input tokens and 300 output tokens. Version 8, with its examples, uses 2,000 input and 300 output. Suppose the price is 3 dollars per million input tokens and 15 dollars per million output tokens, with 100,000 calls a day.

- Version 7: input 1,200 × 100,000 = 120 million tokens, costing 360 dollars. Output 300 × 100,000 = 30 million tokens, costing 450 dollars. Total 810 dollars a day.
- Version 8: input 2,000 × 100,000 = 200 million tokens, costing 600 dollars. Output is the same 450. Total 1,050 dollars a day.

The gain of 6 points in the share of good answers costs 240 dollars more a day, about 7,200 a month. Is it worth it? That is a business decision, and you can make it only because both numbers are measured. Monitoring also catches accidents: a bug that sends the whole conversation history with every call could triple the input tokens overnight.

RAG adds a new place to look. When an answer is wrong, the retrieval step may have found the wrong documents. Evaluate retrieval on its own, for example by checking whether the right page appears among the top 5 results for each test question.

| Classical MLOps | LLMOps equivalent |
|---|---|
| Model version | Prompt version plus model name |
| Accuracy on labels | Judge scores and human spot checks |
| Compute cost | Token cost |
| Feature store | Document index for RAG |

## A common mistake {#mistake}
A common slip is editing a prompt in a web console with no record, so nobody can say what changed when quality falls.

Another slip is trusting a judge model without ever comparing it to people.

## Key points {#key-points}
- Treat prompts like code: version them.
- An LLM judge scales evaluation, but check it against humans.
- Track tokens, since cost can change without a visible change in quality.
- With RAG, evaluate the retrieval step separately.
