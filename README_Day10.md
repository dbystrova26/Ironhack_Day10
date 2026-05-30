# DocInsight — AI-Powered Document Q&A via RAG Pipeline

Ask questions, get answers grounded in your documents. DocInsight combines smart document chunking with a Retrieval-Augmented Generation (RAG) pipeline built on OpenAI — so the AI answers only from what's in your files, not from the internet or general knowledge.

---

## What It Does

Most AI tools answer from general knowledge and can make things up. DocInsight works differently: you give it documents, it reads and indexes them, and when you ask a question it finds the most relevant sections and generates an answer grounded exclusively in your content.

**The pipeline in plain terms:**
1. **Chunk** — cut documents into optimal pieces so retrieval works accurately
2. **Embed** — convert each piece into a numerical representation the AI can search
3. **Retrieve** — when you ask a question, find the most relevant pieces
4. **Answer** — generate a response using only those pieces as context

Applied here to two Trustworthy AI sources: the NIST AI Risk Management Framework PDF and a Reid Blackman podcast transcript.

---

## Repository Structure

```
Ironhack_Day10/
│
├── chunking_strategies.ipynb        # Lab 1: chunking strategy comparison
├── rag_openai.ipynb                 # Lab 2: RAG pipeline — LangChain implementation
├── rag_openai_native.ipynb          # Lab 2: RAG pipeline — OpenAI native implementation
├── 00_dynamic_prompting.ipynb       # Dynamic prompting experiments
├── chunking_recommendations.md      # Final chunking recommendations per content type
│
├── trustworthy_ai_podcast.txt       # Podcast transcript — Trustworthy AI (Reid Blackman)
├── AIQB-ReidBlackman.mp3            # Source podcast audio
├── AIQB-ReidBlackman-compressed.mp3 # Compressed version for API use
├── nist.ai.100-1.pdf                # NIST AI Risk Management Framework PDF
├── nist_ai_rmf.txt                  # Extracted text from NIST PDF
│
├── fixed_size_chunking.png          # Visualisation — fixed-size chunk distribution
├── recursive_chunking.png           # Visualisation — recursive chunk distribution
├── token_chunking.png               # Visualisation — token-based chunk distribution
├── chunk_quality.png                # Visualisation — boundary quality comparison
├── all_distributions.png            # Combined strategy comparison
└── .gitignore
```

---

## Part 1 — Smart Document Chunking

Before a RAG system can search a document, it needs to split it into pieces. Too large and the AI gets irrelevant context. Too small and it loses meaning. Different content types need different strategies.

### Strategies Implemented

| Strategy | How It Works | Best For |
|---|---|---|
| **Fixed-Size** | Splits at fixed character intervals | Uniform, clean text |
| **Recursive Character** | Splits on paragraphs → sentences → words | Structured PDFs |
| **Token-Based** | Splits by token count (how LLMs actually measure text) | LLM integration |
| **Semantic** | Splits when topic similarity drops between sentences | Complex mixed content |

### Key Finding

- **PDFs (NIST framework):** Recursive character chunking at 1,000 tokens with 200 overlap — respects section headers and paragraph structure
- **Podcast transcripts:** Token-based chunking at 500 tokens with 50 overlap — conversational text has no paragraph structure so token accuracy matters more

Full reasoning in `chunking_recommendations.md`.

### Visualisations

Four chunk distribution charts are exported as PNGs — showing how each strategy splits the same content differently in terms of chunk size, overlap, and boundary quality.

---

## Part 2 — RAG Pipeline (Document Q&A)

Two implementations of the full pipeline — same logic, different levels of abstraction:

### `rag_openai.ipynb` — LangChain Implementation
Uses LangChain abstractions for embedding, retrieval, and generation. Faster to build, easier to extend.

### `rag_openai_native.ipynb` — OpenAI Native Implementation
Built directly on OpenAI APIs without LangChain — cosine similarity search in-memory, batch embedding, and Chat Completions for generation. More control, better for understanding what's happening under the hood.

**Pipeline steps:**
1. Load and chunk documents
2. Generate embeddings using `text-embedding-3-small`
3. Store chunks + embeddings in memory
4. On query: embed the question → cosine similarity search → retrieve top-k chunks
5. Pass retrieved chunks as context to `gpt-4o` → return grounded answer with source tracking

```python
# Batch embedding for efficiency
def get_embeddings_batch(texts, model="text-embedding-3-small", batch_size=100):
    all_embeddings = []
    for i in range(0, len(texts), batch_size):
        batch = texts[i:i + batch_size]
        response = client.embeddings.create(model=model, input=batch)
        all_embeddings.extend([item.embedding for item in response.data])
    return all_embeddings
```

---

## Setup

**Requirements:** Python 3.10+, OpenAI API key

```bash
pip install openai langchain langchain-community pypdf2 python-dotenv tiktoken numpy sentence-transformers
```

```bash
export OPENAI_API_KEY="your_api_key_here"
```

---

## Run Order

1. `chunking_strategies.ipynb` — understand and compare chunking strategies
2. `chunking_recommendations.md` — read findings
3. `rag_openai_native.ipynb` — RAG pipeline without abstractions (start here)
4. `rag_openai.ipynb` — RAG pipeline with LangChain
5. `00_dynamic_prompting.ipynb` — dynamic prompting experiments

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Embeddings slow to generate | Use batch processing; switch to `text-embedding-3-small` |
| Retrieved chunks not relevant | Try different chunk sizes or increase `top_k` |
| Answers too generic | Improve prompt or switch to `gpt-4o` from `gpt-4o-mini` |
| Running out of tokens | Reduce chunk size or limit `top_k` |

---

## Built With

- [OpenAI API](https://platform.openai.com) — embeddings (`text-embedding-3-small`) and generation (`gpt-4o`)
- [LangChain](https://python.langchain.com) — RAG pipeline abstractions
- [tiktoken](https://github.com/openai/tiktoken) — token counting
- [Sentence Transformers](https://www.sbert.net) — semantic chunking
- Python 3.10+
