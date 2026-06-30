# Local AI Optimization Showcase

A local-first AI optimization project for comparing how different prompt and retrieval strategies affect cost, latency, privacy, and answer quality.

The showcase runs the same document and task through:

- **Normal prompting**: broad context sent directly to a local model.
- **Prompt Caching**: LLMLingua-style programmatic prompt compression before generation.
- **RAG**: compact semantic retrieval with citations.
- **Advanced RAG**: multi-query retrieval with fused, deduped evidence.
- **Explicit cache**: repeated-run optimization that reuses local results when enabled.

Everything can run locally through free/open-source tools such as Ollama. No document content needs to leave the machine.

## Latest Demo Snapshot

Measured with `llama3.2:3b` on `Project Orbit` docs:

| Strategy | Prompt Tokens | Savings vs Normal | Generation Latency | Best For |
| --- | ---: | ---: | ---: | --- |
| Normal | 442 | 0% | 14370 ms | Baseline comparison |
| Prompt Caching | 336 | 23.98% | 11373 ms | Prompt compression without retrieval |
| RAG | 157 | 64.48% | 5380 ms | Lowest context and fastest focused summary |
| Advanced RAG | 286 | 35.29% | 8468 ms | Broader evidence coverage and richer citations |

## Project Outputs

- `summary-advanced-rag.md`: latest four-way local comparison report.
- `company-showcase-document.md`: company-facing explanation.
- `showcase-setup.md`: step-by-step demo runbook.
- `doc.md`: learning guide for the local AI pipeline.
- `local-ai-showcase.canvas.tsx`: visual dashboard.
- `new-user-guide.canvas.tsx`: onboarding guide for first-time users.

## Quick Start

```powershell
cd "<project-root>\local-ai-runner"
npm install
npm run build
ollama pull llama3.2:3b
```

Run the main fresh comparison:

```powershell
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-advanced-rag.md"
```

Open the report:

```powershell
code "..\summary-advanced-rag.md"
```

## Demo Flow

1. Open the Canvas dashboard.
2. Show the latest report: `summary-advanced-rag.md`.
3. Explain the four strategies: Normal, Prompt Caching, RAG, Advanced RAG.
4. Show token savings and latency.
5. Show RAG/Advanced RAG citations.
6. Show repeated-run caching: re-run with `--use-cache` to reuse local results.
7. Explain local-only privacy: Ollama endpoint is `127.0.0.1`.

## Run The Cache Demo

Cache is off by default so fresh comparisons are honest. Enable it only when demonstrating repeated-work savings.

```powershell
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-cache-miss.md" --use-cache --clear-cache
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership" --out "..\summary-cache-hit.md" --use-cache
```

On the second run, the cached local results are reused, so the four model generations are skipped entirely.

## Run The Web App

A small Node server exposes the same comparison through a browser UI. Paste or upload a `.txt` document, enter a task, and read the formatted report. Reports can be saved in the browser or downloaded as Markdown.

```powershell
cd "<project-root>\local-ai-runner"
npm run serve
```

Then open `http://localhost:3000`. Ollama must be running locally for generation to work.

## Share The Web App With ngrok

To let someone outside your network use the app running on your desktop, expose port 3000 with ngrok:

```powershell
ngrok http 3000
```

ngrok prints a public `https://<random>.ngrok-free.app` URL that forwards to your local server. Generation still runs entirely on your machine through Ollama; only the request and the rendered report cross the tunnel. Close the ngrok process to stop sharing.

## Optional Ollama Embeddings

The default embedding provider is a zero-cost local hash method. For stronger local semantic retrieval:

```powershell
ollama pull nomic-embed-text
$env:RAG_EMBEDDING_PROVIDER="ollama"
$env:RAG_OLLAMA_MODEL="nomic-embed-text"
npm run compare -- --document "<path-to-document.md>" --task "Summarize this document for engineering leadership"
```

## Key Files

| File | Purpose |
| --- | --- |
| `local-ai-runner/src/compare-local.ts` | Main comparison orchestrator |
| `local-ai-runner/src/prompt-compressor.ts` | Prompt Caching compression |
| `local-ai-runner/src/retriever.ts` | RAG and retrieval scoring |
| `local-ai-runner/src/local-llm.ts` | Ollama generation |
| `local-ai-runner/src/local-cache.ts` | Explicit local result cache |
| `showcase-setup.md` | Demo runbook |
| `company-showcase-document.md` | Company-facing explanation |
