## လေ့ကျင့်ခန်း ၁ — ရင်းမြစ်ဖိုင်များ Parsing လုပ်ခြင်း

```python
import pathlib

def parse_text_file(file_path: str) -> str:
    # Read a plain-text source and normalize whitespace
    text = pathlib.Path(file_path).read_text(encoding="utf-8")
    return " ".join(text.split())

def parse_markdown_sections(file_path: str) -> list[dict]:
    # Split a markdown file into sections using headings as boundaries
    lines = pathlib.Path(file_path).read_text(encoding="utf-8").splitlines()
    sections, current_title, current_body = [], "preface", []
    for line in lines:
        if line.startswith("#"):
            if current_body:
                sections.append({
                    "title": current_title,
                    "text": "\n".join(current_body).strip()
                })
            current_title = line.lstrip("# ").strip()
            current_body = []
        else:
            current_body.append(line)
    if current_body:
        sections.append({"title": current_title, "text": "\n".join(current_body).strip()})
    return [s for s in sections if s["text"]]

if __name__ == "__main__":
    demo = pathlib.Path("demo.md")
    demo.write_text(
        "# Project Overview\nRAG systems need clean input data.\n"
        "## Setup\nInstall pgvector and create the extension.",
        encoding="utf-8",
    )
    for section in parse_markdown_sections("demo.md"):
        print(section["title"], "->", section["text"][:50])
```

**အဓိကအယူအဆ** — embedding မတင်မီ ရင်းမြစ်ဖိုင်များကို heading၊ heading boundary စသည့် ဖွဲ့စည်းမှုအရ ခွဲခြား parsing လုပ်ခြင်းသည် နောက်ပိုင်း chunk များ၏ အနက်အဓိပ္ပာယ်ဆိုင်ရာ တိကျမှုကို အခြေခံအားဖြင့် တိုးတက်စေသည်။

## လေ့ကျင့်ခန်း ၂ — Fixed Chunking နှင့် Overlap ထည့်ခြင်း

```python
def fixed_chunks(text: str, size: int = 512, overlap: int = 64) -> list[str]:
    # Cut text into fixed-size chunks with an overlap window
    if size <= overlap:
        raise ValueError("overlap must be smaller than chunk size")
    chunks, start = [], 0
    while start < len(text):
        chunks.append(text[start:start + size])
        if start + size >= len(text):
            break
        start += size - overlap  # step forward by the non-overlapping part
    return chunks

if __name__ == "__main__":
    sample = ("PostgreSQL with pgvector stores embeddings as vectors. "
              "The extension supports approximate nearest neighbor search. "
              "HNSW indexes improve query speed for large collections.") * 3
    result = fixed_chunks(sample, size=120, overlap=20)
    print(f"total chunks: {len(result)}")
    for i, chunk in enumerate(result[:3]):
        print(i, chunk[:60])
```

**အဓိကအယူအဆ** — fixed chunking သည် ရိုးရိုးရှင်းရှင်းဖြစ်သော်လည်း overlap ကို ထည့်ပေးခြင်းအားဖြင့် စာကြောင်းတစ်ခု chunk နှစ်ခုကြား မဖြတ်တောက်စေပါက အဓိပ္ပာယ် ဆုံးရှုံးမှုကို လျှော့ချပေးနိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — Semantic Chunking နှင့် Fixed Chunking နှိုင်းယှဉ်ခြင်း

```python
import re

def split_sentences(text: str) -> list[str]:
    # A simple sentence splitter good enough for English prose
    return [s.strip() for s in re.split(r"(?<=[.!?])\s+", text) if s.strip()]

def semantic_chunks_by_boundary(sentences: list[str],
                                target: int = 300,
                                tolerance: int = 100) -> list[str]:
    # Group sentences into one chunk until adding the next sentence
    # would exceed target + tolerance, then start a new chunk
    chunks, current, length = [], [], 0
    for sentence in sentences:
        if current and length + len(sentence) > target + tolerance:
            chunks.append(" ".join(current))
            current, length = [], 0
        current.append(sentence)
        length += len(sentence)
    if current:
        chunks.append(" ".join(current))
    return chunks

if __name__ == "__main__":
    doc = split_sentences(
        "Vector databases store embeddings. pgvector extends PostgreSQL. "
        "HNSW indexes speed up retrieval. Recall depends on chunk quality. "
        "Metadata filters narrow search scope. Re-ingestion must be safe."
    )
    for chunk in semantic_chunks_by_boundary(doc, target=60, tolerance=20):
        print("-", chunk)
```

**အဓိကအယူအဆ** — semantic chunking သည် စာကြောင်းအတွဲအကြောင်းအရ နယ်နိမိတ်ခွဲသဖြင့် အဓိပ္ပာယ်ဖြတ်တောက်မှု လျှော့ပေးပြီး fixed chunking ထက် ပိုမိုသင့်လျော်သော်လည်း ကုန်ကျစရိတ်နှင့် ရှုပ်ထွေးမှု ပိုများသည်။

## လေ့ကျင့်ခန်း ၄ — Metadata ဒီဇိုင်းချမှတ်ခြင်း

```python
import hashlib
import datetime

def build_chunk_record(source_id: str, chunk_index: int,
                       text: str, extra: dict | None = None) -> dict:
    # Attach stable, filterable metadata to each chunk
    record = {
        "content": text,
        "metadata": {
            "source_id": source_id,
            "chunk_index": chunk_index,
            "char_count": len(text),
            "ingested_at": datetime.datetime.now(datetime.timezone.utc).isoformat(),
            "content_hash": hashlib.sha256(text.encode("utf-8")).hexdigest(),
        }
    }
    if extra:
        record["metadata"].update(extra)
    return record

if __name__ == "__main__":
    rec = build_chunk_record(
        source_id="docs/pgvector-intro.md",
        chunk_index=0,
        text="pgvector is an open-source extension for PostgreSQL.",
        extra={"section": "Setup", "language": "en"},
    )
    print(rec["metadata"]["content_hash"][:16])
    print(rec["metadata"]["source_id"])
```

**အဓိကအယူအဆ** — source identifier၊ chunk index၊ content hash စသည့် metadata များကို chunk တိုင်းတွင် သေချာစွာ ထည့်သွင်းခြင်းသည် filtering၊ tracking နှင့် နောက်ပိုင်း idempotent re-ingestion အတွက် မရှိမဖြစ်လိုအပ်သည်။

## လေ့ကျင့်ခန်း ၅ — Idempotent Re-ingestion ဖန်တီးခြင်း

```python
import hashlib

def compute_document_hash(text: str) -> str:
    # A stable fingerprint used to detect unchanged documents
    return hashlib.sha256(text.encode("utf-8")).hexdigest()

def sync_ingest(store: dict, source_id: str, text: str) -> dict:
    # store maps source_id -> {"doc_hash": str, "chunks": list}
    new_hash = compute_document_hash(text)
    existing = store.get(source_id)
    if existing and existing["doc_hash"] == new_hash:
        return {"action": "skipped", "source_id": source_id}  # no duplicate work
    # Content changed (or new): replace old chunks entirely
    from itertools import count
    chunks = [text[i:i + 100] for i in range(0, len(text), 80)]
    store[source_id] = {
        "doc_hash": new_hash,
        "chunks": [
            {"chunk_index": i, "content": c,
             "content_hash": hashlib.sha256(c.encode("utf-8")).hexdigest()}
            for i, c in zip(count(), chunks)
        ],
    }
    return {"action": "replaced", "source_id": source_id, "chunks": len(chunks)}

if __name__ == "__main__":
    store: dict = {}
    doc = "Idempotent ingestion avoids duplicate rows in pgvector."
    print(sync_ingest(store, "a.md", doc))      # replaced
    print(sync_ingest(store, "a.md", doc))      # skipped
    print(sync_ingest(store, "a.md", doc + " Updated."))  # replaced
```

**အဓိကအယူအဆ** — document hash နှင့် source_id ကို စစ်ဆေးပြီး ပြောင်းလဲမှုမရှိလျှင် ကျော်သွားပြီး ပြောင်းလဲလျှင် ဟောင်းသော chunk များကို အစားထိုးခြင်းဖြင့် re-ingestion ကို ထပ်တူဒုဘီမဖြစ်စေဘဲ လုံခြုံစေသည်။

## လေ့ကျင့်ခန်း ၆ — Chunk Size ၏ Recall နှင့် Cost အပေါ် သက်ရောက်မှု

```python
def estimate_tokens(text: str) -> int:
    # Rough token estimate: ~4 characters per token for English
    return max(1, len(text) // 4)

def compare_chunk_sizes(text: str, sizes: list[int]) -> None:
    # Show how chunk size changes chunk count, overlap overhead and cost
    overlap_ratio = 0.1
    for size in sizes:
        overlap = int(size * overlap_ratio)
        step = max(1, size - overlap)
        chunks = [text[i:i + size] for i in range(0, len(text), step)]
        total_tokens = sum(estimate_tokens(c) for c in chunks)
        print(f"size={size:4d} chunks={len(chunks):3d} "
              f"total_tokens~{total_tokens:5d} "
              f"avg_tokens={total_tokens // max(1, len(chunks)):4d}")

if __name__ == "__main__":
    corpus = ("Retrieval quality depends on chunk size. "
              "Small chunks lose context. Large chunks dilute relevance. "
              "Overlap adds redundant tokens and cost. ") * 40
    compare_chunk_sizes(corpus, sizes=[256, 512, 1024])
```

**အဓိကအယူအဆ** — chunk အရွယ်အစားသည် recall နှင့် cost ကြားရှိ ကိန်းဂဏန်းရိုးရိုး ဆန့်ကျင်ဘက်ဆက်ဆံရေးတစ်ခုဖြစ်သဖြင့် ကိုယ်ပိုင် corpus ပေါ်တွင် တိုင်းတာစမ်းသပ်ပြီး ရွေးချယ်ရမည်။
