# Local AI Optimization Showcase

## Executive Summary

This project demonstrates a local-only AI workflow for company documents. It compares multiple optimization techniques using the same input document, the same task, and the same local model.

The main message:

```text
We can reduce prompt size, add citations, preserve privacy, and measure tradeoffs without sending documents to online AI APIs.
```

Latest sample result:

```text
Normal:       442 prompt tokens
Prompt Caching: 336 prompt tokens
RAG:          157 prompt tokens
Advanced RAG: 286 prompt tokens
```

## What It Is

This project is a local AI comparison system that shows how different prompt and retrieval techniques affect token usage, latency, privacy, and answer quality.

It compares the same document and the same task across multiple local strategies:

1. **Normal prompting**: send broad document context directly to the local model.
2. **Prompt Caching**: compress the document before sending it to the model.
3. **RAG**: retrieve only the most relevant document sections and cite them.
4. **Advanced RAG**: run multiple retrieval queries, fuse evidence, dedupe overlaps, and produce a richer cited evidence pack.
5. **Local Cache**: optionally reuse previous local outputs for repeated work.

Everything runs locally through free/open-source tooling such as Ollama. The goal is to prove that private company documents can be processed without sending data to online APIs.

## How It Works

The input is one document and one task.

Example task:

```text
Summarize this document for engineering leadership
```

The system sends that same input through several local paths.

### 1. Normal Prompting

Normal prompting sends a large part of the document directly to the local model.

This is the baseline.

It is simple, but it often sends more context than needed.

### 2. Prompt Caching

Prompt Caching is a local prompt compression method inspired by LLMLingua.

Before the prompt reaches the model, the compressor:

- Scores lines by task relevance.
- Keeps headings, bullets, paths, and useful structure.
- Removes lower-value or repetitive lines.
- Builds a smaller compressed context.

This reduces prompt size without using RAG.

### 3. RAG

RAG means Retrieval-Augmented Generation.

Instead of sending the full document, the system:

- Splits the document into chunks.
- Creates local embeddings or hash vectors.
- Searches for chunks relevant to the task.
- Compresses the best evidence into short snippets.
- Sends only that evidence to the model.
- Includes source line citations.

RAG is usually best for token savings and fast focused answers.

### 4. Advanced RAG

Advanced RAG is a stronger retrieval path for harder questions.

It:

- Runs multiple retrieval queries.
- Collects a larger candidate evidence set.
- Fuses duplicate or repeated results.
- Removes overlapping source ranges.
- Sends broader cited evidence to the model.

Advanced RAG may use more tokens than compact RAG, but it is designed for better coverage and more complete answers.

### 5. Local Cache

The cache is also optional.

By default, the comparison runs fresh. Cache is enabled only with:

```powershell
--use-cache
```

When cache is enabled, the system stores outputs for the same:

- Document hash.
- Task.
- Model.
- Embedding provider.
- RAG settings.
- Prompt template version.

On a repeat run, it skips model generation and reuses the cached local outputs.

## Steps To Run

### Step 1: Go To The Project

```powershell
cd "<project-root>\local-ai-runner"
```

### Step 2: Install Dependencies

```powershell
npm install
```

### Step 3: Install Local Models

Install Ollama, then pull the local model:

```powershell
ollama pull llama3.2:3b
```

Optional embedding model:

```powershell
ollama pull nomic-embed-text
```

### Step 4: Build

```powershell
npm run build
```

### Step 5: Run The Main Comparison

```powershell
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-advanced-rag.md"
```

This creates a report comparing:

- Normal prompting.
- Prompt Caching.
- RAG.
- Advanced RAG.

### Step 6: Run Cache Demo

First cache run:

```powershell
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-cache-miss.md" --use-cache --clear-cache
```

Second cache run:

```powershell
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-cache-hit.md" --use-cache
```

## Pros

### Privacy

All model calls go to local Ollama:

```text
http://127.0.0.1:11434
```

Company documents do not need to leave the machine.

### Lower Token Usage

Prompt Caching and RAG reduce the amount of context sent to the model.

In the latest run:

```text
Normal prompt: 442 tokens
Prompt Caching: 336 tokens
RAG: 157 tokens
Advanced RAG: 286 tokens
```

### Better Traceability

RAG and Advanced RAG include source line citations.

This makes answers easier to audit.

### Multiple Optimization Techniques

The project is not only RAG.

It demonstrates:

- Prompt compression.
- Semantic retrieval.
- Advanced retrieval fusion.
- Local result caching.

### Good Company Demo Story

The story is easy to explain:

```text
Same document, same task, same local model.
We compare multiple local optimization techniques.
We measure tokens, latency, citations, and privacy.
```

## Cons

### Local Models Are Smaller

Small local models may be weaker than large online models.

They are private and cheap, but answer quality depends on the local model.

### Prompt Caching Can Remove Useful Context

Prompt compression saves tokens, but if too aggressive, it can remove details the model needs.

This is why the report compares Prompt Caching against RAG and Advanced RAG.

### Compact RAG Can Miss Coverage

Compact RAG is very token-efficient, but it may retrieve too little evidence for complex questions.

### Advanced RAG Uses More Tokens Than Compact RAG

Advanced RAG is not always the cheapest option.

It is better for broader coverage, not maximum token savings.

### Cache Must Be Used Carefully

Cache is useful for repeated work, but it should not hide fresh model behavior.

That is why cache is opt-in with:

```powershell
--use-cache
```

### More Metrics Are Needed For Quality

Token savings alone does not prove answer quality.

For a stronger production benchmark, add:

- Citation coverage.
- Unsupported claim count.
- Answer completeness score.
- Human review score.
- Risk/action-item extraction accuracy.

## Recommended Demo Order

1. Show `summary-advanced-rag.md`.
2. Explain Normal vs Prompt Caching vs RAG vs Advanced RAG.
3. Show token savings and latency.
4. Show citations from RAG and Advanced RAG.
5. Run the cache miss/hit demo.
6. Open the Canvas dashboard.
7. Explain that everything runs locally.

## Important Files

- `local-ai-runner/src/compare-local.ts`: comparison orchestrator.
- `local-ai-runner/src/prompt-compressor.ts`: Prompt Caching compression.
- `local-ai-runner/src/retriever.ts`: RAG retrieval.
- `local-ai-runner/src/local-cache.ts`: local cache.
- `local-ai-runner/src/local-llm.ts`: Ollama client.
- `showcase-setup.md`: demo runbook.
- `doc.md`: learning document.
- `summary-advanced-rag.md`: latest comparison report.
