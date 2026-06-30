# Local Normal vs RAG Demo

This project demonstrates how the same document can be sent to a local open-source LLM in two ways:

1. **Normal mode**: send a large part of the document directly to the model.
2. **Prompt Caching mode**: use programmatic prompt compression inspired by LLMLingua before calling the model.
3. **RAG mode**: index the document, retrieve only the most relevant evidence, compress it, then send the smaller evidence pack to the same model.
4. **Advanced RAG mode**: run multiple retrieval queries, fuse/dedupe evidence, and send a richer cited evidence pack.
5. **Cache mode**: optionally reuse a previous local result when the same document, task, model, and settings are requested again.

The goal is to show better accuracy and lower context cost without sending company data online.

## Why This Matters

Large language models are expensive because every prompt token costs time, memory, and money. If we send an entire document every time, the model reads many irrelevant tokens.

RAG means **Retrieval-Augmented Generation**. Instead of sending everything, we first retrieve the most relevant parts of the document and only send those parts to the model.

For a company demo, this shows:

- Better control over private data.
- Lower prompt size.
- Better source citations.
- Comparison between prompt compression and semantic retrieval.
- Comparison between compact RAG and broader Advanced RAG evidence coverage.
- Optional model routing with deterministic validation.
- Faster local workflows after retrieval is optimized.
- Avoided model calls when a cached result is available.
- No dependency on online APIs for the comparison flow.

## Local-Only Architecture

```text
One document input
        |
        +--> Normal path
        |       |
        |       +--> Send large document context to local Ollama model
        |       +--> Generate normal summary
        |
        +--> Prompt Caching path
        |       |
        |       +--> Compress prompt locally with LLMLingua-style scoring
        |       +--> Send compressed document context to local Ollama model
        |       +--> Generate compressed-prompt summary
        |
        +--> RAG path
                |
                +--> Split document into chunks
                +--> Create local embeddings
                +--> Retrieve relevant chunks for the task
                +--> Compress snippets to important lines
                +--> Send compact evidence to local Ollama model
                +--> Generate RAG summary with citations

        +--> Advanced RAG path
                |
                +--> Run multiple retrieval queries
                +--> Fuse and dedupe evidence
                +--> Preserve broader cited coverage
                +--> Generate advanced cited summary

Repeated run
        |
        +--> Local cache lookup
        +--> Reuse previous normal and RAG outputs
        +--> Report avoided model calls, tokens, and latency
```

Both paths use the **same local model**. The only difference is the context sent to the model.

## Local Model

The default local LLM is:

```text
llama3.2:3b
```

It runs through [Ollama](https://ollama.com/) on your PC or Mac.

No document content is sent to an online service. The model call goes to:

```text
http://127.0.0.1:11434
```

## Setup

Install Ollama, then pull a small local model:

```powershell
ollama pull llama3.2:3b
```

Optional stronger local embedding model:

```powershell
ollama pull nomic-embed-text
```

Install and build the TypeScript project:

```powershell
cd "<project-root>\local-ai-runner"
npm install
npm run build
```

## Run One Document Through Normal and RAG

Example:

```powershell
npx tsx src/compare-local.ts --document "/Users/chetagup/Downloads/porjecti0x-main/essay.md" --task "Summarize this document for english test checkup" --out "../summary-report.md"
```

This creates a Markdown report with:

- The input document and task.
- Normal LLM output.
- Prompt Caching compressed-prompt LLM output.
- RAG LLM output.
- Advanced RAG LLM output.
- RAG evidence and citations.
- Prompt token estimates.
- Context token savings.
- Prompt Caching compression ratio.

## Show The Cache Optimization

The normal command runs fresh by default. Use that for the honest normal-vs-RAG comparison:

```powershell
npx tsx src/compare-local.ts --document "C:\Users\namit\Downloads\porjecti0x\essay.md" --task "Summarize this document for english test checkup" --out "..\summary-report.md"
```

Use cache only when you want to show repeated-work optimization.

First run with a clean cache:

```powershell
npx tsx src/compare-local.ts --document "C:\Users\namit\Downloads\porjecti0x\essay.md" --task "Summarize this document for english test checkup" --out "..\summary-cache-miss.md" --use-cache --clear-cache
```

Second run with the same document, task, model, and settings:

```powershell
npx tsx src/compare-local.ts --document "C:\Users\namit\Downloads\porjecti0x\essay.md" --task "Summarize this document for english test checkup" --out "..\summary-cache-hit.md" --use-cache
```

On the second run, the cached results are reused, so the local generations are skipped.

This demonstrates an optimization beyond RAG: repeated work can be skipped locally.

## Use a Different Local Model

```powershell
npm run compare -- --document "C:\path\to\document.md" --task "Summarize risks and action items" --model "qwen2.5:3b"
```

Before running, pull the model:

```powershell
ollama pull qwen2.5:3b
```

## Use Ollama Embeddings

By default, embeddings use a zero-cost local hash method. For better semantic retrieval, use Ollama embeddings:

```powershell
$env:RAG_EMBEDDING_PROVIDER="ollama"
$env:RAG_OLLAMA_MODEL="nomic-embed-text"
npm run compare -- --document "C:\path\to\document.md" --task "Summarize the architecture"
```

This is still local-only.

## Important Options

```text
--document              Document path to compare.
--task                  Question or summary instruction.
--out                   Markdown report output path.
--model                 Ollama LLM model. Default: llama3.2:3b.
--max-context-tokens    RAG evidence budget. Default: 700.
--normal-max-tokens     Normal baseline document budget. Default: 4000.
--top-k                 Optional fixed number of RAG results.
--dry-run               Build prompts and report without calling the LLM.
--use-cache             Enable cache lookup and write for cache demos.
--no-cache              Disable cache lookup and write.
--clear-cache           Clear the local cache before running.
```

## What The Report Proves

The report compares token usage:

```text
Normal prompt tokens vs RAG prompt tokens
Normal prompt tokens vs Advanced RAG prompt tokens
Normal prompt tokens vs Prompt Caching prompt tokens
Normal context tokens vs RAG context tokens
Token savings percentage
Prompt Caching compression ratio
Advanced RAG query count, candidate count, fused evidence count
```

If RAG uses fewer tokens while producing a useful summary with citations, the demo proves that retrieval can reduce model workload and improve traceability.

## Company Demo Script

Use this explanation:

> We send the same document to the same local model in four modes. Normal mode gives the model a large document context. Prompt Caching compresses the prompt before generation. RAG retrieves compact cited evidence. Advanced RAG uses multiple retrieval queries and fused evidence for stronger coverage. Then the local cache shows how repeated work can avoid model calls entirely. The report shows output quality, citations, and token savings. This means we can keep documents private, reduce processing cost, and make model answers easier to audit.

## Where The Code Lives

- `local-ai-runner/src/compare-local.ts`: one-document normal vs RAG runner.
- `local-ai-runner/src/prompt-compressor.ts`: Prompt Caching compression.
- `local-ai-runner/src/local-cache.ts`: local result cache for repeated document/task runs.
- `local-ai-runner/src/local-llm.ts`: local Ollama generation client.
- `local-ai-runner/src/retriever.ts`: hybrid RAG retrieval, compression, citations, token budget.
- `local-ai-runner/src/embedding.ts`: hash embeddings and optional Ollama embeddings.
- `local-ai-runner/src/benchmark.ts`: benchmark for repo-level RAG vs normal retrieval.

## Notes

This demo is standalone and local. It is designed to run with local files and local models without online AI processing.
