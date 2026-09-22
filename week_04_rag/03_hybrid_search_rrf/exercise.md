## လေ့ကျင့်ခန်း ၁ — Keyword recall နှင့် Vector recall နှိုင်းယှဉ်ခြင်း

pgvector ဖြင့် `documents` table တစ်ခု ပြင်ဆင်ပါ။ `content TEXT`, `embedding VECTOR(1536)` column များ ပါရှိရမည်။ မေးခွန်းတစ်ခုအတွက် (a) ILIKE keyword query နှင့် (b) cosine distance vector query ကို အသီးသီး လုပ်ဆောင်ပြီး ရလဒ် top-5 များကို နှိုင်းယှဉ်ပါ။

```sql
-- Keyword recall
SELECT id, content FROM documents
WHERE content ILIKE '%invoice%'
LIMIT 5;

-- Vector recall (query_embedding is computed in application code)
SELECT id, content FROM documents
ORDER BY embedding <=> :query_embedding
LIMIT 5;
```

**Hints:** ILIKE query မှာ parameter binding သုံးပြီး SQL injection ရှောင်ပါ။ Vector query မတိုင်ခင် query text ကို embedding API ဖြင့် ပြောင်းပါ။
**Expected behavior:** ခြုံငုံပြောလုံး (paraphrase) မေးခွန်းများမှာ vector recall က ပိုကောင်းပြီး၊ အတိအက technical term များမှာ keyword recall က ပိုမှန်ကန်တွေ့ရမည်။

## လေ့ကျင့်ခန်း ၂ — Reciprocal Rank Fusion (RRF) ကို Python ဖြင့် အလုပ်လုပ်စေခြင်း

Function တစ်ခု ရေးပါ — input မှာ ranked list များ (ခေါင်းစဉ်တစ်ခု၊ document ID list) ဖြစ်ပြီး output မှာ RRF score အလိုက် စီထားသော list ဖြစ်ရမည်။

```python
def rrf(rankings: dict[str, list[str]], k: int = 60) -> list[str]:
    # Each ranking maps a source name to its ordered document IDs
    scores = {}
    for _, ids in rankings.items():
        for rank, doc_id in enumerate(ids, start=1):
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank)
    return [doc_id for doc_id, _ in sorted(scores.items(), key=lambda x: -x[1])]
```

**Hints:** `k=60` သည် သတင်းအချက်အလက် များသော list များအတွက် သုံးသော နည်းပညာ parameter ဖြစ်သည်။ `sorted` ထဲမှာ lambda ဖြင့် score အလိုက် အနိမ့်မှမြင့် ပြောင်းပါ။
**Expected behavior:** list နှစ်ခုစလုံးမှာ ရှိသော document များက score ပိုမိုရပြီး ရလဒ်ထိပ်ဆုံးသို့ တက်လာရမည်။

## လေ့ကျင့်ခန်း ၃ — SQL pattern အသုံးပြု၍ hybrid search တစ်ခုတည်းဖြင့် ရေးခြင်း

လေ့ကျင့်ခန်း ၁–၂ ကို ပေါင်းပြီး SQL CTE များနှင့် hybrid search query တစ်ခု ရေးပါ။ Keyword ranking နှင့် vector ranking ကို CTE ခွဲ၍ RRF ဖြင့် ပေါင်းစည်းပါ။

```sql
WITH kw AS (
  SELECT id, ROW_NUMBER() OVER (ORDER BY id) AS rank
  FROM documents WHERE content ILIKE :term LIMIT 20
),
vec AS (
  SELECT id, ROW_NUMBER() OVER () AS rank
  FROM documents ORDER BY embedding <=> :query_embedding LIMIT 20
)
SELECT d.id, d.content,
       COALESCE(1.0/(60 + kw.rank), 0) + COALESCE(1.0/(60 + vec.rank), 0) AS rrf_score
FROM documents d
LEFT JOIN kw ON kw.id = d.id
LEFT JOIN vec ON vec.id = d.id
WHERE kw.id IS NOT NULL OR vec.id IS NOT NULL
ORDER BY rrf_score DESC LIMIT 10;
```

**Hints:** `ROW_NUMBER()` ဖြင့် rank ထုတ်ပါ။ Document တစ်ခုက list တစ်ခုတည်းမှာပဲ ရှိလျှင် `COALESCE` က ကာကွယ်ပေးမည်။
**Expected behavior:** SQL တစ်ခုတည်းနှင့် hybrid result ရရှိပြီး application-side merge မလိုတော့ပါ။

## လေ့ကျင့်ခန်း ၄ — Metadata filter များအတွက် jsonb pattern

Table မှာ `metadata JSONB` column ထည့်ပါ (ဥပမာ `{"source": "handbook", "lang": "en", "year": 2024}`)။ Hybrid query ကို filter ထည့်ပြီး ပြန်ရေးပါ။

```sql
-- GIN index for fast jsonb containment queries
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);

-- Filtered vector search
SELECT id, content FROM documents
WHERE metadata @> '{"source": "handbook"}'::jsonb
ORDER BY embedding <=> :query_embedding
LIMIT 5;
```

**Hints:** `@>` containment operator သည် GIN index နှင့် တွဲမှ မြန်မည်။ Array filter အတွက် `metadata->>'lang' = ANY(:langs)` ကို စမ်းကြည့်ပါ။
**Expected behavior:** Filter ကို `ORDER BY` မတိုင်ခင် အသုံးပြုသဖြင့် distance တွက်ချက် လျှော့စေပြီး၊ ရလဒ်များအားလုံးက သတ်မှတ် metadata နှင့် ကိုက်ညီရမည်။

## လေ့ကျင့်ခန်း ၅ — Reranker pass ထည့်သွင်းခြင်း

Hybrid search မှ top-20 candidates ကို ယူပြီး reranker API (cross-encoder သို့မဟုတ် platform မှ rerank endpoint) ဖြင့် ပြန်စီပါ။ Candidate များကို တစ်ခါတည်း batch ဖြင့် ပို့ပါ။

```python
def rerank(query: str, docs: list[str], top_k: int = 5) -> list[str]:
    # scores come from an external reranker service
    results = reranker_api.score(query, docs)
    # sort by descending relevance score
    ranked = sorted(zip(docs, results), key=lambda x: -x[1])
    return [doc for doc, _ in ranked[:top_k]]
```

**Hints:** Reranker က embedding model ထက် ပိုစရိတ်ကြီးလေ့ရှိသဖြင့် candidate အရေအတွက်ကို ကန့်သတ်ပါ။ Reranker ကို ခေါ်ခြင်း မလုပ်ခင် hybrid top-k ကို အရင် တိကျစေပါ။
**Expected behavior:** Rerank ပြီးနောက် ထိပ်ဆုံးရလဒ်များက မေးခွန်းနှင့် ပိုသက်ဆိုင်လာပြီး၊ ကောင်းသော document တစ်ခု ထိပ်ဆုံးမှတစ်နေရာသို့ ရောက်လာသည်ကို မြင်ရမည်။

## လေ့ကျင့်ခန်း ၆ — Top-k tuning pipeline တည်ဆောင်ခြင်း

Pipeline တစ်ခု တည်ဆောက်ပါ — hybrid recall (`n1`) → rerank (`n2`) → final answer ထဲ ထည့်မည့် context။ `n1` (ဥပမာ 20/50/100) နှင့် `n2` (ဥပမာ 3/5/10) ကို ပြောင်းလွှဲ စမ်းသပ်ပြီး မိမိ data ပေါ်မှာ မည်သည့် setting က အကောင်းဆုံးဆိုသည်ကို ကြည့်ပါ။

```python
def run_pipeline(query: str, n1: int, n2: int) -> str:
    # stage 1: hybrid recall of n1 candidates
    candidates = hybrid_search(query, top_k=n1)
    # stage 2: rerank down to n2 documents
    final_docs = rerank(query, candidates, top_k=n2)
    # stage 3: assemble context for the LLM
    context = "\n".join(final_docs)
    return ask_llm(query, context)
```

**Hints:** `n1` ကြီးလျှင် rerank ကုန်ကျစရိတ် တိုးမည်။ တိကျသော ကြည့်ရှုရန်အတွက် မိမိ test queries အနည်းငယ် ချထားပြီး ရလဒ်များကို ကိုယ်တိုင် အမှတ်ပေး နှိုင်းယှဉ်ပါ။ မိမိ data ပေါ်မှာ တိုင်းတာထားသော ကိန်းဂဏန်းများကိုသာ ကိုးကားပါ၊ ထုတ်ဝေထားသော benchmark များ မမှီတမ်း မဆွဲပါ။
**Expected behavior:** `n1`/`n2` ပြောင်းလိုက်တိုင်း အဖြေအရည်အသွေ ကွာခြားမှုကို မြင်ရပြီး၊ cost နှင့် quality အကြား မိမိ data အတွက် သင့်တော်သော ချိန်ညှိချက် ရွေးနိုင်ရမည်။
