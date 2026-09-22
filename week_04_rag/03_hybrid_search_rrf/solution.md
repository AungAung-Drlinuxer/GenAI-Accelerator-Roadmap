# အဖြေများ — Hybrid Search (Keyword + Vector + Rerank)

## လေ့ကျင့်ခန်း ၁ — Keyword recall နှင့် Vector recall နှိုင်းယှဉ်ခြင်း

```python
import os
import psycopg2
from psycopg2 import sql
from openai import OpenAI

# Connect to Postgres with pgvector enabled
conn = psycopg2.connect(os.environ["DATABASE_URL"])
cur = conn.cursor()

# Create the documents table with a 1536-dim vector column
cur.execute("""
CREATE EXTENSION IF NOT EXISTS vector;
DROP TABLE IF EXISTS documents;
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536)
);
""")

client = OpenAI()

def embed(text: str) -> list[float]:
    # Call the embedding API for a single text
    resp = client.embeddings.create(model="text-embedding-3-small", input=[text])
    return resp.data[0].embedding

docs = [
    "Please send me the invoice for March.",
    "How do I reset my password?",
    "The billing statement is overdue.",
    "Invoice payment terms are net 30.",
    "Where can I download the receipt?",
    "Refund policy for damaged goods.",
]

# Insert documents with their embeddings
for d in docs:
    cur.execute(
        "INSERT INTO documents (content, embedding) VALUES (%s, %s)",
        (d, embed(d)),
    )
conn.commit()

query = "how to pay a bill I received"
query_vec = embed(query)

# (a) Keyword recall: ILIKE with a bound parameter to avoid SQL injection
cur.execute(
    "SELECT id, content FROM documents WHERE content ILIKE %s LIMIT 5",
    ("%invoice%",),
)
kw_results = cur.fetchall()

# (b) Vector recall: cosine distance via the <=> operator
cur.execute(
    """
    SELECT id, content FROM documents
    ORDER BY embedding <=> %s::vector
    LIMIT 5
    """,
    (query_vec,),
)
vec_results = cur.fetchall()

print("Keyword results:", kw_results)
print("Vector results:", vec_results)

# Example: for the paraphrase query "how to pay a bill I received",
# the vector search finds "The billing statement is overdue."
# even though no keyword overlaps, while ILIKE finds nothing.
```

**အဓိကအယူအဆ** — ခြုံငုံပြောလုံးမေးခွန်းများအတွက် vector recall က အဓိပ္ပာယ်တူသော document များကို ရှာတွေ့ပေးပြီး အတိအက technical term ပါသော ရှာဖွေမှုများအတွက် keyword recall က ပိုတိကျသဖြင့် နှစ်မျိုးလုံးကို တွဲဖက်အသုံးပြုသင့်သည်။

## လေ့ကျင့်ခန်း ၂ — Reciprocal Rank Fusion (RRF) ကို Python ဖြင့် အလုပ်လုပ်စေခြင်း

```python
def rrf(rankings: dict[str, list[str]], k: int = 60) -> list[str]:
    # Each ranking maps a source name to its ordered document IDs
    scores = {}
    for _, ids in rankings.items():
        for rank, doc_id in enumerate(ids, start=1):
            # Accumulate RRF contribution 1/(k + rank) per list
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank)
    # Sort document IDs by descending fused score
    return [doc_id for doc_id, _ in sorted(scores.items(), key=lambda x: -x[1])]


# Demo: two ranked lists with partial overlap
keyword_ranking = ["doc_a", "doc_c", "doc_e"]
vector_ranking = ["doc_b", "doc_a", "doc_d", "doc_e"]

fused = rrf({"keyword": keyword_ranking, "vector": vector_ranking})
print(fused)
# doc_a and doc_e appear in both lists, so their scores combine
# and they rise to the top of the fused ranking.
```

**အဓိကအယူအဆ** — RRF က list နှစ်ခုစလုံးတွင် ပါဝင်သော document များ၏ rank အချက်အလက်များကို ပေါင်းစည်းပေးသဖြင့် ဖောက်ထွင်းရှာဖွေမှုနှစ်မျိုးလုံးမှ အာရုံစိုက်ခံရသော document များ ရလဒ်ထိပ်ဆုံးသို့ တက်လာစေသည်။

## လေ့ကျင့်ခန်း ၃ — SQL pattern အသုံးပြု၍ hybrid search တစ်ခုတည်းဖြင့် ရေးခြင်း

```python
import os
import psycopg2

conn = psycopg2.connect(os.environ["DATABASE_URL"])
cur = conn.cursor()

# Single SQL statement combining keyword and vector recall via CTEs and RRF
HYBRID_SQL = """
WITH kw AS (
    SELECT id, ROW_NUMBER() OVER (ORDER BY id) AS rank
    FROM documents
    WHERE content ILIKE %(term)s
    LIMIT 20
),
vec AS (
    SELECT id, ROW_NUMBER() OVER () AS rank
    FROM documents
    ORDER BY embedding <=> %(query_embedding)s::vector
    LIMIT 20
)
SELECT d.id, d.content,
       COALESCE(1.0 / (60 + kw.rank), 0)
     + COALESCE(1.0 / (60 + vec.rank), 0) AS rrf_score
FROM documents d
LEFT JOIN kw ON kw.id = d.id
LEFT JOIN vec ON vec.id = d.id
WHERE kw.id IS NOT NULL OR vec.id IS NOT NULL
ORDER BY rrf_score DESC
LIMIT 10;
"""


def hybrid_search(query: str, term: str, query_embedding: list[float]) -> list[tuple]:
    # Execute the fused query with bound parameters
    cur.execute(HYBRID_SQL, {"term": f"%{term}%", "query_embedding": query_embedding})
    return cur.fetchall()


# Example usage
results = hybrid_search("how to pay a bill", "invoice", embed("how to pay a bill"))
for row in results:
    print(row)
```

**အဓိကအယူအဆ** — Keyword ranking နှင့် vector ranking ကို SQL CTE များဖြင့် ခွဲကာ RRF score များဖြင့် ပေါင်းစည်းခြင်းက hybrid search ကို database ထဲမှာပဲ တစ်ခါတည်း ပြီးစော်များပြီး application-side merge လုပ်စရာ မလိုတော့ပါ။

## လေ့ကျင့်ခန်း ၄ — Metadata filter များအတွက် jsonb pattern

```python
import os
import psycopg2

conn = psycopg2.connect(os.environ["DATABASE_URL"])
cur = conn.cursor()

# Add a jsonb metadata column and a GIN index for containment queries
cur.execute("""
ALTER TABLE documents ADD COLUMN IF NOT EXISTS metadata JSONB;
DROP INDEX IF EXISTS idx_documents_metadata;
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);
""")
cur.execute(
    "UPDATE documents SET metadata = %s WHERE id = 1",
    ('{"source": "handbook", "lang": "en", "year": 2024}',),
)
conn.commit()

# Filtered vector search: the filter runs before the distance ordering
cur.execute(
    """
    SELECT id, content FROM documents
    WHERE metadata @> '{"source": "handbook"}'::jsonb
    ORDER BY embedding <=> %s::vector
    LIMIT 5
    """,
    (embed("how to pay a bill"),),
)
containment_results = cur.fetchall()

# Array filter example: match any of the allowed languages
cur.execute(
    """
    SELECT id, content FROM documents
    WHERE metadata->>'lang' = ANY(%(langs)s)
    ORDER BY embedding <=> %(emb)s::vector
    LIMIT 5
    """,
    {"langs": ["en", "my"], "emb": embed("how to pay a bill")},
)
array_results = cur.fetchall()

print(containment_results)
print(array_results)
```

**အဓိကအယူအဆ** — jsonb metadata ကို `@>` containment operator နှင့် GIN index ဖြင့် `ORDER BY` မတိုင်ခင် စစ်ထုတ်ပါက distance တွက်ချက်မည့် document အရေအတွက် လျှော့သွားပြီး ရလဒ်များအားလုံးလည်း သတ်မှတ် metadata နှင့် ကိုက်ညီစေသည်။

## လေ့ကျင့်ခန်း ၅ — Reranker pass ထည့်သွင်းခြင်း

```python
from dataclasses import dataclass


@dataclass
class RerankerAPI:
    # Stub for an external cross-encoder reranker service
    def score(self, query: str, docs: list[str]) -> list[float]:
        # In production this calls a rerank endpoint in a single batch
        return [0.1 * i for i in range(len(docs))]


reranker_api = RerankerAPI()


def rerank(query: str, docs: list[str], top_k: int = 5) -> list[str]:
    # Cap the candidate count to control reranker cost
    candidates = docs[:20]
    # Send all candidates in one batch call
    results = reranker_api.score(query, candidates)
    # Sort (doc, score) pairs by descending relevance score
    ranked = sorted(zip(candidates, results), key=lambda x: -x[1])
    return [doc for doc, _ in ranked[:top_k]]


# Demo: rerank the hybrid top-20 candidates down to top-5
hybrid_top20 = hybrid_search("how to pay a bill", "invoice",
                             embed("how to pay a bill"))
candidate_texts = [row[1] for row in hybrid_top20][:20]
final = rerank("how to pay a bill", candidate_texts, top_k=5)
print(final)
```

**အဓိကအယူအဆ** — Reranker က embedding model ထက် ပိုစရိတ်ကြီးသဖြင့် hybrid top-k ကို အရင်တိကျစေပြီး candidate ၂၀ ခန့်သာ batch တစ်ခုတည်းနှင့် ပို့ခြင်းက quality နှင့် cost အကြား ညှိနှိုင်းပေးသည်။

## လေ့ကျင့်ခန်း ၆ — Top-k tuning pipeline တည်ဆောင်ခြင်း

```python
def run_pipeline(query: str, n1: int, n2: int) -> str:
    # Stage 1: hybrid recall of n1 candidates
    candidates = hybrid_search(query, "invoice", embed(query))
    candidate_texts = [row[1] for row in candidates][:n1]
    # Stage 2: rerank down to n2 documents
    final_docs = rerank(query, candidate_texts, top_k=n2)
    # Stage 3: assemble context for the LLM
    context = "\n".join(final_docs)
    return ask_llm(query, context)


def ask_llm(query: str, context: str) -> str:
    # Stub for the final LLM answer step
    return f"answer for: {query} (context chars: {len(context)})"


# Sweep n1 and n2 on your own small set of test queries
test_queries = [
    "how to pay a bill",
    "refund policy for damaged goods",
    "reset my password",
]

for n1 in [20, 50, 100]:
    for n2 in [3, 5, 10]:
        for q in test_queries:
            answer = run_pipeline(q, n1, n2)
            # Grade each answer yourself on a fixed 1-5 scale
            print(f"n1={n1} n2={n2} query={q!r} -> {answer}")
```

**အဓိကအယူအဆ** — `n1` ကြီးလာသလောက် rerank ကုန်ကျစရိတ် တိုးသဖြင့် မိမိ test queries အနည်းငယ်ပေါ်မှာ `n1`/`n2` ပြောင်းလွှဲ၍ ကိုယ်တိုင်အမှတ်ပေးနှိုင်းယှဉ်ခြင်းသည် ကိုယ်ပိုင် data အတွက် cost နှင့် quality သင့်တော်ဆုံး ချိန်ညှိချက်ကို ရွေးနိုင်စေသည်။
