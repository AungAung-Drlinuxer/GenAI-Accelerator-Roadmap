# Week 4 — Retrieval Augmented Generation

## ဒီပတ်မှာ ဘာသင်မလဲ

ဒီပတ်က production-grade RAG (Retrieval Augmented Generation) system တစ်ခုကို အခြေခံကနေ တည်ဆောက်နည်းကို အဓိက သင်ကြားပါမယ်။ အောက်ပါ အကြောင်းအရာတွေ ပါဝင်ပါမယ်-

- Document ingestion နဲ့ chunking strategy တွေ ရွေးချယ်နည်း (fixed, semantic, overlap)
- Embedding model ရွေးချယ်ခြင်း၊ vector store မှာ storing နဲ့ versioning လုပ်နည်း
- Keyword search နဲ့ vector search ကို ပေါင်းစပ်ထားတဲ့ hybrid search နဲ့ Reciprocal Rank Fusion (RRF)
- RAG system quality ကို တိုင်းတာနိုင်ဖို့ evaluation metrics (recall@k, MRR, nDCG) နဲ့ grounding checks

## Modules

- `01_ingestion_chunking/` — **Ingestion & Chunking Strategies** — Source parsing၊ semantic vs fixed chunking၊ overlap၊ metadata design၊ idempotent re-ingestion နဲ့ chunk size က recall နဲ့ cost ပေါ်သက်ရောက်ပုံကို သင်ကြားပါတယ်။
- `02_embeddings_vector_store/` — **Embeddings & Vector Store** — Embedding model နဲ့ dimensionality၊ cosine/L2/inner product distance ရွေးချယ်နည်း၊ normalization၊ batch embedding နဲ့ vector storing/versioning ကို pgvector နဲ့ လက်တွေ့လုပ်ကြည့်ပါတယ်။
- `03_hybrid_search_rrf/` — **Hybrid Search, RRF & Reranking** — Keyword vs vector recall၊ RRF fusion၊ SQL hybrid search patterns၊ reranker pass၊ top-k tuning နဲ့ jsonb metadata filters ကို သင်ကြားပါတယ်။
- `04_rag_evaluation/` — **RAG Evaluation & Grounding** — Golden question set တည်ဆောက်ခြင်း၊ recall@k / MRR / nDCG၊ faithfulness နဲ့ citation checks၊ CI regression gates နဲ့ hallucination mitigation နည်းတွေကို လေ့လာပါတယ်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ် ပြီးဆုံးရင် သင်ဟာ အောက်ပါအတိုင်း လုပ်ဆောင်နိုင်မယ်-

- ဒေတာ source တစ်ခုကနေ chunk တွေ ထုတ်ပြီး metadata နဲ့အတူ vector store ထဲ ထည့်နိုင်မယ်
- Chunk size၊ overlap နဲ့ embedding model selection တွေကို use case အလိုက် ဆုံးရှုံးနည်း ဆုံးဖြတ်နိုင်မယ်
- Hybrid search query တစ်ခုကို SQL နဲ့ရေးပြီး RRF နဲ့ reranking လုပ်နိုင်မယ်
- RAG pipeline တစ်ခုရဲ့ quality ကို golden set နဲ့ တိုင်းတာပြီး CI ထဲ regression gate သွင်းနိုင်မယ်

## လေ့လာရန် အစီအစဉ်

- **Day 1–2:** `01_ingestion_chunking/` — chunking basics နဲ့ metadata design
- **Day 3–4:** `02_embeddings_vector_store/` — embedding နဲ့ pgvector setup
- **Day 5:** `03_hybrid_search_rrf/` — hybrid search နဲ့ reranking
- **Day 6–7:** `04_rag_evaluation/` — evaluation set တည်ဆောက်ခြင်း နဲ့ metrics တိုင်းတာခြင်း

## Checkpoint

1. Chunk size ကြီးရင် ရင်ဘယ်အားသာချက်တွေ၊ ဘယ်အားနည်းချက်တွေ ရှိလဲ။ Recall နဲ့ cost ပေါ် ဘယ်လိုသက်ရောက်လဲ။
2. Cosine similarity နဲ့ inner product ကို normalization လုပ်ပြီးတဲ့အခါ ဘာကွာခြားလဲ။
3. RRF က ဘယ်လိုလုပ်သလဲ ဆိုတာ ရှင်းပြပါ — keyword ranking နဲ့ vector ranking ကနေ score တစ်ခုထွက်လာပုံကို ဥပမာနဲ့ ပြပါ။
4. Golden question set တစ်ခုကို RAG system evaluate ဖို့ ဘယ်လို design လုပ်မလဲ။ Faithfulness ကို ဘယ်လိုစစ်မလဲ။
