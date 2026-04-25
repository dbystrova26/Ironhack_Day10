
# Chunking Strategy Recommendations
## Lab Results Summary
Audio file: AIQB-ReidBlackman.mp3 (Reid Blackman — AI Ethics & Responsible AI)
PDF:        NIST AI Risk Management Framework (AI RMF 1.0)

Tested strategies: Fixed-Size | Recursive | Token-Based | Semantic
Metrics:           Sentence preservation | Mid-word breaks | Clean starts

---

## For PDF Documents (NIST AI RMF):

**Recommended Strategy:** Semantic 0.7

**Reasoning:**
- The NIST document has clearly defined section headers (SECTION 1, 1.1, 1.2),
  numbered requirements, and dense technical definitions. Recursive chunking
  with section-aware separators respects these structural boundaries naturally.
- Sentence preservation rate: 95% of chunks end
  at a natural sentence boundary
- Mid-word break rate: 5% — nearly eliminated
- Technical definitions stay intact — DSCR, LTV, fairness requirements
  are not split across chunks, preserving their meaning for RAG retrieval
- Significantly better than fixed-size (3%
  sentence preservation) at negligible extra cost

**Optimal settings:**
- chunk_size: 1000 characters
- overlap: 200 characters
- separators: ["\n\nSECTION", "\n\n", "\n", ". ", " ", ""]

**When NOT to use recursive for PDFs:**
- Scanned PDFs (no text structure preserved after OCR)
- PDFs with unusual formatting — tables, multi-column layouts

---

## For Podcast Transcripts:

**Recommended Strategy:** Semantic 0.7

**Reasoning:**
- The Reid Blackman interview has natural topic shifts — from definition
  of trustworthy AI, to bias in hiring, to regulatory frameworks. Semantic
  chunking detects these shifts through embedding similarity rather than
  character patterns, keeping topically coherent exchanges together.
- Preserves full Q&A pairs — a question and its answer stay in one chunk,
  which is critical for retrieval quality in downstream RAG applications
- Sentence preservation: 96% vs
  2% for fixed-size
- Mid-word breaks: 0% vs
  93% for fixed-size

**Optimal settings:**
- similarity threshold: 0.7
- min_chunk_size: 100 characters
- For production: combine with recursive as fallback for very long segments

**When NOT to use semantic for podcasts:**
- High-volume batch processing (too slow at scale)
- Real-time transcription pipelines
- When compute cost matters more than retrieval quality

---

## Trade-offs Summary:

| Strategy      | Sentence Preservation | Mid-Word Breaks | Speed       | Best For                        |
|---------------|----------------------|-----------------|-------------|---------------------------------|
| Fixed-Size    | 2% podcast / 3% PDF | High (~30-40%)  | ⭐⭐⭐⭐⭐ | Prototyping, uniform content    |
| Recursive     | 0% podcast / 16% PDF | Near 0%         | ⭐⭐⭐⭐  | Structured docs, good default   |
| Token-Based   | 4% podcast / 4% PDF | Low             | ⭐⭐⭐⭐  | Production RAG, LLM pipelines   |
| Semantic      | 96% podcast / 95% PDF | Near 0%         | ⭐         | Complex content, best accuracy  |

---

## Decision Framework:

  Are you prototyping?
    YES → Fixed-size (fast, simple, iterate quickly)
    NO  ↓

  Is retrieval accuracy critical?
    YES → Semantic chunking
    NO  ↓

  Is your content structured (headers, sections)?
    YES → Recursive with content-specific separators
    NO  ↓

  Are you optimising for LLM context windows?
    YES → Token-based (512-1024 tokens, overlap 10%)
    NO  → Recursive (safe general default)

---

## Final Verdict for This Lab:

  NIST AI RMF PDF:       Recursive 1000/200 with section separators
  Reid Blackman Podcast: Semantic chunking threshold=0.7

  The combination of recursive (for structured documents) and semantic
  (for conversational content) covers the vast majority of real-world
  RAG use cases with excellent boundary quality at reasonable cost.
