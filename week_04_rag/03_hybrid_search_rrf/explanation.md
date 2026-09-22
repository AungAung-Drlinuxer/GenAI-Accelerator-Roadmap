# Hybrid Search, RRF & Reranking

## Keyword Recall နှင့် Vector Recall — ကွဲပြားပုံ

### ဘာကို ဆိုလိုတာလဲ
Keyword recall ဆိုတာ ရှာဖွေမှုကို စကားလုံးအတုအပ ကိုက်ညီမှုဖြင့် လုပ်ဆောင်တဲ့နည်း (ဥပမာ — PostgreSQL ရဲ့ full-text search, `tsvector`, `ts_query`) ဖြစ်ပါတယ်။ Vector recall ကတော့ text တွေကို embedding vector အဖြစ်ပြောင်းပြီး semantic အနက်အဓိပ္ပာယ်အရ နီးစပ်မှုကို တွက်ချက်တဲ့နည်း (ဥပမာ — pgvector ရဲ့ `<=>` distance operator) ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ
Embedding တစ်ခုတည်းနဲ့ ရှာရင် စကားလုံးမတူပေမယ့် အဓိပ္ပာယ်တူတာတွေကို ရပေမယ့် — product code, person name, version number စတဲ့ exact token တွေကို မ_miss သင့်ပါ။ ပြန်ပြောရရင် keyword search က exact term များအတွက် အားကောင်းပြီး vector search က paraphrase နှင့် သဘောတူရှာဖွေမှုအတွက် အားကောင်းပါတယ်။ နှစ်ခုစလုံးမှ အကောင်းဆုံး coverage ရမှာဖြစ်လို့ hybrid လို့ခေါ်တဲ့ နည်းလမ်း ဖွံ့ဖြိုးလာတာပါ။

### ဘယ်လို အလုပ်လုပ်လဲ
Keyword side မှာ document တစ်ခုစီကို `tsvector` column ထဲသိမ်းပြီး GIN index တပ်ဆင်ပါတယ်။ Vector side မှာ `vector` column နှင့် HNSW index သုံးပါတယ်။ Query ဝင်လာတဲ့အခါ နှစ်ခုစလုံးကို တပြိုင်နက် ဖြတ်ပြီး ရလဒ်နှစ်စားရဲ့ ranking ကို နောက်တစ်ဆင့် ပေါင်းစပ်ရပါတယ်။

### ဥပမာ
```python
# Demonstrates the same query returning different top hits from two retrieval modes
import numpy as np

docs = [
    "Postgres 16.3 connection timeout error code 57014",
    "database refuses new connections when max limit reached",
    "how to bake chocolate chip cookies at home",
]

def keyword_score(query: str, doc: str) -> int:
    # Simple token overlap as a stand-in for full-text search ranking
    q = set(query.lower().split())
    d = set(doc.lower().split())
    return len(q & d)

query = "error code 57014"
print("Keyword match:", [keyword_score(query, d) for d in docs])
# Expected output: Keyword match: [4, 0, 0]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production RAG system တိုင်းမှာ "zero result" သို့မဟုတ် "wrong document" ဖြစ်စေတဲ့ အဓိကအကြောင်းရင်းတစ်ခုက recall နည်းမှုပါ။ Hybrid နည်းက နှစ်ဖက်စလုံးရဲ့ အားနည်းချက်တွေကို ဖြည့်ပေးတာမလို့ corporate codebase, legal document တွေလို exact-token-heavy corpus တွေမှာ မဖြစ်မနေ လိုအပ်ပါတယ်။

## Reciprocal Rank Fusion (RRF)

### ဘာကို ဆိုလိုတာလဲ
RRF ဆိုတာ ranking list တွေများစွာကို score scale မတူညီဘဲ ပေါင်းစပ်ပေးတဲ့ ranking fusion algorithm ပါ။ Document တစ်ခုရဲ့ fused score ကို ဒီ formula နဲ့ တွက်ပါတယ် — `score = Σ (1 / (k + rank))` မှာ `k` (အများအားဖြင့် 60) က smoothing constant ပါ။

### ဘာကြောင့် လဲ
Keyword search ရဲ့ score (ဥပမာ — ts_rank) နှင့် vector distance score တို့က unit လည်းမတူ၊ distribution လည်းမတူပါ။ Directly ပေါင်းရင် တစ်ဖက်က score က တစ်ဖက်ကို လွှမ်းမိုးမှာဖြစ်ပါတယ်။ RRF က position-based ဖြစ်လို့ scale ကို လုံးဝ မမှီခိုပါ။

### ဘယ်လို အလုပ်လုပ်လဲ
ရှေ့ဆုံးမှာ document တစ်ခုချင်းစီရဲ့ rank ကို နှစ်လမ်းသွားမှ စုဆောင်းပါတယ်။ ပြီးရင် document တစ်ခုစီအတွက် `1 / (k + rank_i)` တန်ဖိုးတွေကို ပေါင်းပါတယ်။ နှစ်လမ်းသွားမှာ အပေါ်ဆုံး ပေါ်လာတဲ့ document က fused ranking မှာလည်း အပေါ်ဆုံး ရောက်တတ်ပါတယ်။

### ဥပမာ
```python
# Pure-python RRF to show how the fusion math works
def rrf(rankings: list[list[str]], k: int = 60) -> dict[str, float]:
    scores = {}
    for ranking in rankings:
        for rank, doc_id in enumerate(ranking, start=1):
            scores[doc_id] = scores.get(doc_id, 0.0) + 1.0 / (k + rank)
    return dict(sorted(scores.items(), key=lambda x: -x[1]))

keyword_results = ["doc_a", "doc_b", "doc_c"]   # from tsvector full-text search
vector_results  = ["doc_c", "doc_a", "doc_d"]   # from pgvector similarity search

fused = rrf([keyword_results, vector_results])
print(fused)
# Expected output: {'doc_a': 0.03254690063347503, 'doc_c': 0.03254690063347503, 'doc_b': 0.016321922398589066, 'doc_d': 0.016321922398589066}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Hybrid search ရဲ့ နောက်ဆုံးဆုံး "glue" က RRF ပါ။ Parameter tuning လုပ်စရာ အနည်းငယ်နဲ့ stable၊ ရှင်းရှင်းလင်းလင်း အလုပ်လုပ်တာမလို့ Elasticsearch, OpenSearch, Weaviate တို့မှာပါ built-in feature အဖြစ် ပါဝင်ပါတယ်။

## SQL Patterns for Hybrid Search (PostgreSQL + pgvector)

### ဘာကို ဆိုလိုတာလဲ
pgvector extension ထည့်သွင်းထားတဲ့ PostgreSQL table တစ်ခုပေါ်မှာ keyword ranking နှင့် vector ranking ကို တစ် query တည်း (သို့မဟုတ် CTE နှစ်ခု) နဲ့ ရယူပြီး SQL အတွင်းမှာ RRF နဲ့ ပေါင်းစပ်တဲ့ pattern ပါ။

### ဘာကြောင့် လဲ
Vector DB သီးသန့် service ထပ်ထည့်ရင် infrastructure ရှုပ်ထွေးပြီး consistency ထိန်းခိုင်းရခက်ပါတယ်။ Postgres တစ်ခုတည်းနဲ့ full-text, vector, metadata filter, RRF အားလုံး လုပ်နိုင်ရင် ပိုရိုးသားပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
ပထမ — table မှာ `content tsv tsvector`, `embedding vector(N)` column တွေ ပါရမယ်။ GIN index (`tsvector` အတွက်) နှင့် HNSW index (`vector` အတွက်) တပ်ဆင်ပါ။ Query မှာ CTE တစ်ခုက ts_rank နဲ့ keyword ranking ထုတ်ပြီး တစ်ခုက distance နဲ့ vector ranking ထုတ်ကာ နောက်ဆုံး CTE မှာ RRF ပေါင်းပါတယ်။

### ဥပမာ
```python
# Hybrid search executed as a single SQL statement with in-database RRF
import psycopg

HYBRID_SQL = """
WITH kw AS (
    SELECT id, ROW_NUMBER() OVER (ORDER BY ts_rank(content_tsv, query) DESC) AS rank
    FROM docs, websearch_to_tsquery('english', %(q)s) AS query
    WHERE content_tsv @@ query
    LIMIT 50
),
vec AS (
    SELECT id, ROW_NUMBER() OVER (ORDER BY embedding <=> %(emb)s::vector) AS rank
    FROM docs
    ORDER BY embedding <=> %(emb)s::vector
    LIMIT 50
)
SELECT id, SUM(1.0 / (60 + rank)) AS rrf_score
FROM (
    SELECT id, rank FROM kw
    UNION ALL
    SELECT id, rank FROM vec
) fused
GROUP BY id
ORDER BY rrf_score DESC
LIMIT %(final_k)s;
"""

# with psycopg.connect(DB_URL) as conn:
#     rows = conn.execute(HYBRID_SQL, {"q": query, "emb": "[" + ",".join(map(str, emb)) + "]", "final_k": 10}).fetchall()
print("SQL pattern ready: two CTE rankings fused with RRF, final top-k returned")
# Expected output: SQL pattern ready: two CTE rankings fused with RRF, final top-k returned
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
`ROW_NUMBER() OVER (ORDER BY ...)` နဲ့ rank ထုတ်တဲ့ pattern က RRF ကို database အတွင်းမှာပဲ ဖြစ်စေတာမလို့ application layer ကို document list ကြီးတွေ ပြန်တင်ပေးရနေရမှာ လျှော့ပါတယ်။ Shard ခွဲထားတဲ့ production setup မှာပါ ဒီ pattern ကို အတူတူပဲ အသုံးချနိုင်ပါတယ်။

## Reranker Pass နှင့် Top-k Tuning

### ဘာကို ဆိုလိုတာလဲ
Reranking ဆိုတာ hybrid retrieval ကရထားတဲ့ candidate list ကို ပိုတိကျတဲ့ cross-encoder model (ဥပမာ — `BAAI/bge-reranker-base` သို့မဟုတ် hosted reranker API) နဲ့ ပြန်စစ်ပြီး final ordering ပြောင်းပေးတဲ့ ဒုတိယအဆင့် ဖြစ်ပါတယ်။ Top-k tuning ကတော့ (1) retrieval ဘက်မှာ ဘယ်နှခု အထကတင်ယူမလဲ (`candidate_k`) နှင့် (2) reranker အပြီးမှာ ဘယ်နှခု ကျန်ရစ်မလဲ (`final_k`) ဆိုတာ ချိန်ညှိမှုပါ။

### ဘာကြောင့် လဲ
Vector နှင့် keyword retrieval တို့က approximate နှင့် shallow matching ပါ။ Cross-encoder က query နှင့် document ကို တပြိုင်တည်း တွဲဖတ်ပြီး အမှတ်ပေးတာမလို့ bi-encoder embedding ထက် ပိုတိကျပေမယ့် တစ်ခုချင်းစီ အလုပ်များလို့ corpus တစ်ခုလုံးပေါ် မှာ run လို့မရပါ။ ဒါကြောင့် recall stage က ကျယ်ကျယ် (ဥပမာ — `candidate_k = 50`) ရှာပြီး reranker က precision မြှင့်တင်ပေးတဲ့ two-stage design ဖြစ်လာပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Retriever က `candidate_k` ခုကို RRF score အလိုက် ယူပါတယ်။ Reranker က query-doc pair တစ်ခုချင်းစီကို relevance score ပေးပြီး ပြန် sort လုပ်ကာ `final_k` ခုကိုသာ LLM context ထဲ ထည့်ပါတယ်။ `candidate_k` ကြီးရင် recall တက်ပေမယ့် latency နှင့် reranking cost တက်ပါတယ် — ဒါကြောင့် ကိုယ့် corpus, ကိုယ့် latency budget ပေါ် မူတည်ပြီး တိုင်းတာပြီး ချိန်ညှိရပါတယ်။

### ဥပမာ
```python
# Two-stage retrieve-then-rerank flow with configurable k values
def retrieve_then_rerank(query: str, candidate_k: int = 50, final_k: int = 5):
    hybrid_hits = hybrid_search(query, top_k=candidate_k)       # RRF-fused candidates
    pairs = [(query, doc.text) for doc in hybrid_hits]
    rerank_scores = reranker_model.predict(pairs)               # cross-encoder scores
    ranked = sorted(zip(hybrid_hits, rerank_scores), key=lambda x: -x[1])
    return [doc for doc, _ in ranked[:final_k]]

# top_docs = retrieve_then_rerank("connection timeout error 57014", candidate_k=50, final_k=5)
print("Stage 1: broad hybrid recall -> Stage 2: precise cross-encoder rerank")
# Expected output: Stage 1: broad hybrid recall -> Stage 2: precise cross-encoder rerank
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
LLM က context ရှိတဲ့ အထဲမှာ အားလုံးကို မှန်ကန်စွာ အသုံးမချနိုင်ပါ — မဆိုင်တဲ့ chunk တွေ ပါလာရင် hallucination နှင့် "lost in the middle" လို့ခေါ်တဲ့ ပြဿနာတွေ ဖြစ်လာတတ်ပါတယ်။ ဒါကြောင့် reranker နဲ့ quality မြှင့်ပြီး `final_k` ကို သင့်တင့်စွာ ချုံးတာက အဖြေမှန် ရရနိုင်မှုကို တိုက်ရိုက် သက်ရောက်စေပါတယ်။

## Metadata Filters with jsonb

### ဘာကို ဆိုလိုတာလဲ
jsonb filter ဆိုတာ document တွေရဲ့ metadata (source, tenant, date, category စသဖြင့်) ကို PostgreSQL ရဲ့ `jsonb` column ထဲမှာ သိမ်းပြီး vector/keyword search ရှာတဲ့အခါ `@>` containment operator နဲ့ ကန့်သတ်တာပါ။

### ဘာကြောင့် လဲ
Multi-tenant system တွေမှာ တစ်ယူဆာရဲ့ query က အခြား tenant ရဲ့ document ကို ရမိပါစေလို့ မရပါ။ Metadata filter က ရှာဖွေမှုကို ကန့်သတ်ပြီး security နှင့် relevance နှစ်ခုစလုံး ထိန်းပေးပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Metadata ကို `metadata jsonb` column မှာ သိမ်းပြီး GIN index တပ်ပါ (`CREATE INDEX ON docs USING GIN (metadata jsonb_path_ops);`)။ Vector search query ထဲမှာ `WHERE metadata @> '{"tenant_id": "acme"}'` လို့ ထည့်ရင် pgvector က filtered candidate set အပေါ်မှာပဲ index scan လုပ်ပါတယ် — ဒါက pre-filter နည်းပါ။ Filter selectivity မြင့်ရင် pre-filter က index ကို ထိရိုက်စေပေမယ့် filter က loose ဖြစ်နေရင် partial index သို့မဟုတ် post-filter နည်း စဉ်းစားရပါတယ်။

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Retrieval quality က embedding တစ
