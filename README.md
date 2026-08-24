# Applied AI Engineering Project Roadmap

## Purpose

This repository develops application-engineering skills for systems that use hosted language models, embeddings, retrieval, tool calling, streaming, evaluation, caching, and batch workflows.

The focus is not on training a model from scratch. The focus is on building reliable software systems that use AI models as application dependencies.

## How to Use This Roadmap

For each unfinished project:

1. Read the project specification.
2. Build the project yourself before looking for a complete implementation.
3. Use official documentation freely for syntax and APIs.
4. Run every **Required Completion Criterion / Test**.
5. Do not move on until the required tests pass.
6. Complete the **Understanding Check** without relying on your code.

### General Testing Rules

Unless a project says otherwise:

- Prefer automated tests over manual inspection.
- Test the happy path, invalid input, and at least one relevant failure path.
- Keep tests deterministic and independent of execution order.
- Use isolated test data/state.
- Do not require manually starting the server before the test suite.
- When external services are involved, use mocks/fakes for deterministic tests and add a separate real-service smoke test where useful.
- Record any required environment variables and setup commands in the project README.
- When measuring performance, define the workload and run repeated measurements rather than reporting one cherry-picked result.
- A project is not complete merely because it worked once.

## Completion Tracker

- [ ] Project 1 --- Hosted LLM API Integration
- [ ] Project 2 --- Embeddings
- [ ] Project 3 --- Cosine Similarity
- [ ] Project 4 --- Chunking
- [ ] Project 5 --- Vector Search
- [ ] Project 6 --- Retrieval-Augmented Generation
- [ ] Project 7 --- Function / Tool Calling
- [ ] Project 8 --- Structured Output
- [ ] Project 9 --- Reusable AI Harness
- [ ] Project 10 --- Streaming Responses
- [ ] Project 11 --- Prompt Caching
- [ ] Project 12 --- AI Application Evaluation
- [ ] Project 13 --- Prompt Versioning
- [ ] Project 14 --- Batch LLM Requests
- [ ] Project 15 --- Guardrails and Safety Evaluation

------------------------------------------------------------------------

## Project 1 --- Hosted LLM API Integration

### Objective

Integrate a hosted language model as an external dependency using a clean provider abstraction.

### Required Implementation

- Create a small provider interface.
- Implement one hosted-LLM provider.
- Handle credentials, timeout, and provider errors.
- Return usage/token metadata when available.

### Required Completion Criteria / Tests

1. A mocked provider test verifies the exact request shape sent by your application.
2. A real smoke test returns a model response using environment-provided credentials.
3. A provider timeout produces a controlled application error.
4. Provider credentials never appear in source code or logs.
5. Business logic can use the provider interface without knowing which vendor implementation is active.

### Understanding Check

Explain:

- Why an external LLM should be treated like any other unreliable network dependency.
- Why provider-specific code should be isolated.

------------------------------------------------------------------------

## Project 2 --- Embeddings

### Objective

Generate vector representations for documents and use them as application data.

### Required Implementation

- Generate embeddings for a small fixed document corpus.
- Persist the embedding together with source metadata.
- Inspect dimensions and basic vector properties.

### Required Completion Criteria / Tests

1. Every input document produces an embedding of the expected fixed dimension.
2. Repeated deterministic embedding calls for identical input produce the expected same or numerically equivalent result for the chosen provider/model.
3. Every stored embedding can be traced back to its source document.
4. Malformed/empty input follows a documented policy.

### Understanding Check

Explain:

- What an embedding is operationally.
- Why the embedding dimension is fixed for a model.

------------------------------------------------------------------------

## Project 3 --- Cosine Similarity

### Objective

Implement cosine similarity yourself and use it to compare embedding vectors.

### Required Implementation

- Implement cosine similarity without calling a library cosine helper.
- Compare your implementation against a trusted reference.
- Rank a fixed set of document embeddings for several query embeddings.

### Required Completion Criteria / Tests

1. A vector compared with itself returns approximately `1.0`.
2. Orthogonal synthetic vectors return approximately `0.0`.
3. Opposite synthetic vectors return approximately `-1.0`.
4. Your implementation matches a trusted reference within `1e-6` across at least 100 random vector pairs.
5. For five hand-authored query/document cases, the expected relevant item ranks above an unrelated control.

### Understanding Check

Explain:

- Why cosine similarity measures angle rather than vector magnitude.
- Why high embedding similarity is not the same as factual equivalence.

------------------------------------------------------------------------

## Project 4 --- Chunking

### Objective

Split long documents into retrievable units while preserving enough metadata to reconstruct their source.

### Required Implementation

- Implement at least one fixed-size chunker and one structure-aware or sentence/paragraph-aware chunker.
- Support configurable overlap.
- Attach document ID and source offsets/section metadata to every chunk.

### Required Completion Criteria / Tests

1. Every chunk references a valid source document.
2. Chunk source offsets map back to the expected original text.
3. No text is silently dropped under the documented chunking policy.
4. Changing overlap produces the expected repeated boundary content.
5. A fixed retrieval experiment compares the two chunking strategies on at least 10 queries.

### Understanding Check

Explain:

- Why chunk size is a retrieval/context tradeoff.
- Why overlap can help and why too much overlap can hurt.

------------------------------------------------------------------------

## Project 5 --- Vector Search

### Objective

Store embeddings in a searchable index and retrieve semantically similar chunks.

### Required Implementation

- Use a vector-capable database/index.
- Implement top-k nearest-neighbor retrieval.
- Support metadata filtering.
- Create a brute-force cosine-search baseline for a small fixed corpus.

### Required Completion Criteria / Tests

1. Top-k results from the vector index match brute-force search on the fixed small corpus.
2. A metadata filter never returns a chunk outside the allowed metadata.
3. Adding a new document makes it retrievable without breaking existing records.
4. Deleting a document removes its chunks from retrieval.
5. At least 20 fixed queries can be run automatically and their result IDs recorded.

### Understanding Check

Explain:

- Exact versus approximate nearest-neighbor search.
- Why retrieval quality needs a test set rather than one convincing demo.

------------------------------------------------------------------------

## Project 6 --- Retrieval-Augmented Generation

### Objective

Answer questions using retrieved document context and return the supporting sources.

### Required Implementation

- Retrieve top-k chunks for a query.
- Construct a context-grounded prompt.
- Generate an answer with a hosted LLM.
- Return source/chunk identifiers.
- Define behavior when evidence is insufficient.

### Required Completion Criteria / Tests

1. Create at least 30 answerable and 10 unanswerable questions from a fixed corpus.
2. For answerable questions, the expected supporting chunk appears in top-k retrieval at least 90% of the time.
3. Returned source IDs correspond exactly to chunks actually supplied to the model.
4. For unanswerable questions, the system follows the documented no-evidence/abstention behavior.
5. Removing the relevant source document causes the associated retrieval test to fail as expected, proving the test is meaningful.

### Understanding Check

Explain:

- The difference between retrieval failure and generation failure.
- Why RAG can still hallucinate with correct context.

------------------------------------------------------------------------

## Project 7 --- Function / Tool Calling

### Objective

Allow the model to request deterministic application functions while keeping execution under application control.

### Required Implementation

- Register at least two tools.
- Define JSON schemas for tool arguments.
- Validate tool names and arguments before execution.
- Execute tools in application code and return the result to the model.

### Required Completion Criteria / Tests

1. A fixed evaluation set of at least 20 prompts selects the expected tool for at least 90% of cases designed to require one of the tools.
2. Invalid tool arguments are rejected before execution.
3. The model cannot execute an unregistered tool.
4. A tool exception becomes a controlled tool result/error rather than crashing the request.
5. A prompt that requires no tool does not force a tool call.

### Understanding Check

Explain:

- Why the model chooses a tool but should not directly execute arbitrary code.
- Why tool inputs need normal validation and authorization.

------------------------------------------------------------------------

## Project 8 --- Structured Output

### Objective

Convert model output into validated data that deterministic application code can safely consume.

### Required Implementation

- Define a Zod or JSON Schema output contract.
- Request structured output.
- Validate every response.
- Use a bounded retry/repair policy if the provider does not guarantee the schema.

### Required Completion Criteria / Tests

1. At least 50 fixed inputs produce schema-valid outputs or a controlled failure.
2. Malformed mocked model output never reaches downstream business logic.
3. Retry attempts are bounded.
4. A schema change breaks an intentionally stale test, proving the contract is enforced.
5. Valid JSON that violates semantic business rules is separately rejected where appropriate.

### Understanding Check

Explain:

- Why valid JSON is not necessarily valid application data.
- Why structured output is preferable to ad-hoc text parsing.

------------------------------------------------------------------------

## Project 9 --- Reusable AI Harness

### Objective

Create a reusable wrapper that standardizes model calls across the application.

### Required Implementation

- Centralize provider selection, retries/timeouts, model settings, logging, usage metadata, and error translation.
- Support both plain text and structured responses.
- Allow callers to supply a prompt/template without duplicating provider code.

### Required Completion Criteria / Tests

1. Two different application features can use the same harness.
2. Provider errors map to one consistent application error interface.
3. Changing a default model setting in one place affects all callers that inherit the default.
4. Per-call overrides do not mutate global configuration.
5. A mocked harness test verifies logging/usage metadata without calling a real provider.

### Understanding Check

Explain:

- Why shared AI infrastructure should be separate from feature-specific prompts.
- Which behavior belongs in the harness and which belongs in the feature.

------------------------------------------------------------------------

## Project 10 --- Streaming Responses

### Objective

Stream LLM output incrementally to a client and handle cancellation and mid-stream failure.

### Required Implementation

- Use SSE or a documented chunked-streaming approach.
- Forward generated chunks/tokens incrementally.
- Handle client disconnect/cancellation.
- Measure time to first token/chunk and total request duration.

### Required Completion Criteria / Tests

1. The client receives multiple chunks before completion.
2. The first chunk arrives before the full response is complete.
3. Disconnecting the client stops or cancels downstream work where supported.
4. A simulated provider error after streaming begins follows the documented stream-error behavior.
5. Time-to-first-token/chunk is measured for at least 20 requests.

### Understanding Check

Explain:

- Why streaming improves perceived latency without necessarily reducing compute.
- Why HTTP error handling changes after the response has begun.

------------------------------------------------------------------------

## Project 11 --- Prompt Caching

### Objective

Cache safe repeated LLM requests without serving incorrect responses across users or configurations.

### Required Implementation

- Define a cache key that includes normalized input, prompt version, model identifier, and material generation settings.
- Set a TTL.
- Define which requests are safe to cache.
- Keep user-scoped content isolated.

### Required Completion Criteria / Tests

1. Two identical safe requests produce a cache miss followed by a hit.
2. Changing the prompt version causes a cache miss.
3. Changing a material model setting causes a cache miss.
4. User A cannot receive User B's user-scoped cached response.
5. A cache outage follows a documented fallback policy.

### Understanding Check

Explain:

- Why cache-key design determines correctness.
- Why semantic caching and exact caching have different risks.

------------------------------------------------------------------------

## Project 12 --- AI Application Evaluation

### Objective

Create a repeatable evaluation harness for the application rather than judging a few demos manually.

### Required Implementation

- Create a versioned evaluation dataset.
- Measure retrieval hit rate where retrieval is used.
- Measure structured-output validity where applicable.
- Measure tool-selection accuracy where applicable.
- Define one answer-quality rubric and document its limitations.
- Save results in machine-readable form.

### Required Completion Criteria / Tests

1. The full evaluation runs from one command.
2. Deterministic metrics reproduce exactly for the same application version and evaluation set.
3. An intentionally degraded configuration produces worse measured results.
4. Results record prompt/config version and model identifier.
5. Failures are categorized so retrieval, schema, tool, and generation failures can be distinguished.

### Understanding Check

Explain:

- Why evaluation should isolate component failures.
- Why a single aggregate score can hide important regressions.

------------------------------------------------------------------------

## Project 13 --- Prompt Versioning

### Objective

Treat prompts and model configuration as versioned application dependencies.

### Required Implementation

- Store prompts outside route handlers.
- Assign prompt versions.
- Record prompt version, model identifier, and generation configuration with evaluation runs.
- Keep at least two prompt versions available for comparison.

### Required Completion Criteria / Tests

1. Two prompt versions can be run against the exact same evaluation set.
2. A regression can be traced to the prompt/config version that produced it.
3. Production code has one authoritative copy of each prompt version.
4. Changing a prompt does not silently overwrite historical evaluation metadata.

### Understanding Check

Explain:

- Why prompt changes can behave like code changes.
- Why reproducibility requires prompt and configuration metadata.

------------------------------------------------------------------------

## Project 14 --- Batch LLM Requests

### Objective

Process many independent LLM tasks efficiently while controlling concurrency, retries, and partial failure.

### Required Implementation

- Accept a batch of independent model requests.
- Limit concurrency.
- Retry only retryable failures.
- Return per-item success/failure rather than failing the whole batch automatically.

### Required Completion Criteria / Tests

1. A 100-item test batch completes with every item accounted for.
2. Configured maximum concurrency is never exceeded.
3. One permanently failing item does not erase successful results from other items.
4. Retryable mocked failures recover according to policy.
5. Duplicate job submission follows a documented idempotency policy.

### Understanding Check

Explain:

- Why batch processing is different from one huge prompt.
- Why partial-failure semantics must be explicit.

------------------------------------------------------------------------

## Project 15 --- Guardrails and Safety Evaluation

### Objective

Add application-level controls for clearly disallowed inputs/outputs and evaluate their behavior.

### Required Implementation

- Define a small explicit policy relevant to the demo application.
- Implement pre- and/or post-generation checks.
- Log safety decisions without storing unnecessary sensitive content.
- Create a fixed adversarial evaluation set.

### Required Completion Criteria / Tests

1. Every test case has an expected allow/block outcome.
2. The evaluation reports false positives and false negatives.
3. At least 50 fixed cases run automatically.
4. A blocked input cannot reach the protected downstream action.
5. Changing the guardrail threshold/configuration produces measurable evaluation changes.

### Understanding Check

Explain:

- Why application guardrails are not the same as model alignment.
- Why false positives and false negatives both matter.

------------------------------------------------------------------------

# Recommended Repository Structure

```text
applied-ai-engineering/
├── 01-<project-name>/
├── 02-<project-name>/
├── ...
└── README.md
```

For each unfinished project, use approximately:

```text
README.md
src/
tests/
```

Add `results/` for benchmarks or evaluation outputs.

Each project README should record:

- objective;
- setup and run commands;
- implementation notes;
- required test results;
- relevant metrics;
- design decisions/tradeoffs;
- what you learned;
- remaining questions.
