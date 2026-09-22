# Week 3 — PostgreSQL + pgvector + Alembic လေ့ကျင့်ခန်းများ

## လေ့ကျင့်ခန်း ၁ — pgvector extension ထည့်သွင်းပြီး documents table ဖန်တီးခြင်း

PostgreSQL တွင် pgvector extension ကို install လုပ်ပြီး `CREATE EXTENSION vector;` ဖြင့် enable လုပ်ပါ။ ထို့နောက် documents table (id, title, source_url, created_at) ကို ဖန်တီးပါ။ psycopg2 သို့မဟုတ် psql ဖြင့် လုပ်ဆောင်နိုင်သည်။

```python
import psycopg2

conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/aidb")
cur = conn.cursor()
cur.execute("CREATE EXTENSION IF NOT EXISTS vector;")
cur.execute("""
    CREATE TABLE IF NOT EXISTS documents (
        id BIGSERIAL PRIMARY KEY,
        title TEXT NOT NULL,
        source_url TEXT,
        created_at TIMESTAMPTZ DEFAULT now()
    );
""")
conn.commit()
```

**Hints:** extension ထည့်ပြီးမှ vector type ကို သုံးနိုင်မည် ဖြစ်သည်။ superuser permission လိုအပ်နိုင်သည်။

**Expected behavior:** query တွေအားလုံး error မရှိဘဲ runပြီး `\dx` တွင် vector extension ကို တွေ့ရမည်။

## လေ့ကျင့်ခန်း ၂ — chunks table နှင့် embedding column design ခြင်း

documents တစ်ခုကို chunk များစွာအဖြစ် ခွဲသိမ်းသည့် `document_chunks` table ကို ဖန်တီးပါ။ embedding column ကို `vector(1536)` (OpenAI text-embedding-3-small အတွက်) အဖြစ် သတ်မှတ်ပြီး foreign key ဖြင့် documents နှင့် ချိတ်ဆက်ပါ။ position column ဖြင့် chunk အစဉ် မှတ်ပါ။

```sql
CREATE TABLE document_chunks (
    id BIGSERIAL PRIMARY KEY,
    document_id BIGINT REFERENCES documents(id) ON DELETE CASCADE,
    position INT NOT NULL,
    content TEXT NOT NULL,
    embedding vector(1536)
);
```

**Hints:** dimension ကို သတ်မှတ်ထားပါက မတူသည့်အရွယ် vector ထည့်လျှင် error ရမည်။ ON DELETE CASCADE က document ပျက်လျှင် chunk များ အလိုအလျောက် ပျက်စေသည်။

**Expected behavior:** မတူညီသည့် dimension vector ထည့်သည့်အခါ PostgreSQL က error ပြမည်။

## လေ့ကျင့်ခန်း ၃ — Distance operators ဖြင့် similarity search

embedding များကို sample data အဖြစ် ထည့်သွင်းပြီး `<->` (L2), `<=>` (cosine), `<#>` (inner product) operator များကို စမ်းကြည့်ပါ။ cosine distance ဖြင့် query vector နှင့် အနီးစပ်ဆုံး chunk ၃ခု ရွေးထုတ်ပါ။

```sql
-- cosine distance: 0 = identical, 2 = opposite
SELECT id, content, embedding <=> '[0.1, 0.2, ...]'::vector AS distance
FROM document_chunks
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 3;
```

**Hints:** text embedding များအတွက် cosine distance ကို အများအားဖြင့် သုံးသည်။ ORDER BY တွင် operator ကို တိုက်ရိုက် ရေးနိုင်သည်။

**Expected behavior:** distance အနည်းဆုံး chunk များ အစဉ်လိုက် ပြန်ရမည်၊ တူညီသော vector သည် distance 0 နီးပါး ရမည်။

## လေ့ကျင့်ခန်း ၄ — HNSW vs IVFFlat index နှိုင်းယှဉ်ခြင်း

vector(4) column ပါသည့် စမ်းသပ် table တွင် HNSW index နှင့် IVFFlat index နှစ်မျိုးစလုံး ဖန်တီးပြီး `EXPLAIN ANALYZE` ဖြင့် စမ်းကြည့်ပါ။ IVFFlat အတွက် lists ကို row count ၏ square root နီးပါးနှင့် HNSW အတွက် m နှင့် ef_construction parameter များ စမ်းပါ။

```sql
CREATE INDEX ON document_chunks USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON document_chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

**Hints:** IVFFlat သည် index ဆောက်ရန် မြန်သော်လည်း data အရေအတွက် နည်းသည့်အခါ recall ကျနိုင်သည်။ HNSW သည် memory ပိုသုံးသော်လည်း query ပိုမြန်သည်။ index မဖန်တီးခင် data ထည့်ပြီးမှ IVFFlat ဆောက်သင့်သည်။

**Expected behavior:** EXPLAIN ANALYZE တွင် index scan ကို တွေ့ရပြီး index မရှိချင်း (Seq Scan) ထက် ကွာခြားမှုကို plan ထဲတွင် မြင်နိုင်မည်။

## လေ့ကျင့်ခန်း ၅ — Alembic project စတင်ပြီး initial migration ရေးခြင်း

`alembic init migrations` ဖြင့် project စတင်ပြီး `alembic/env.py` တွင် SQLAlchemy `target_metadata` ကို ချိတ်ပါ။ Exercise ၁-၂ ၏ schema ကို Python model များအဖြစ် ရေးပြီး `alembic revision --autogenerate` ဖြင့် migration ထုတ်ပါ။

```python
# migrations/env.py - connect metadata for autogenerate
from myapp.models import Base
target_metadata = Base.metadata
```

**Hints:** database URL ကို `alembic.ini` ရဲ့ `sqlalchemy.url` တွင် သို့မဟုတ် env variable ဖြင့် သတ်မှတ်ပါ။ autogenerate က vector column type ကို မသိလျှင် `postgresql` dialect မှ `Vector` type ကို custom ထည့်ပေးရန် လိုနိုင်သည်။

**Expected behavior:** `alembic upgrade head` run ပြီးနောက် database ထဲတွင် documents နှင့် document_chunks table များ ပေါ်ရမည်။

## လေ့ကျင့်ခန်း ၆ — HNSW index ထည့်သည့် migration နှင့် build cost တွက်ချက်ခြင်း

Exercise ၅ ၏ schema အပေါ် HNSW index ထည့်သည့် migration အသစ် တစ်ခု ရေးပါ။ ထို့နောက် row အရေအတွက် ကွာခြားသည့် dataset ၃ မျိုး (ဥပမာ ၁၀၀၊ ၁၀၀၀၀၊ ၁၀၀၀၀၀) ဖြင့် `CREATE INDEX` ကြာချိန်ကို Python `time` module ဖြင့် တိုင်းတာပြီး မှတ်တမ်းတင်ပါ။

```python
# migrations/versions/xxxx_add_hnsw_index.py
def upgrade():
    op.execute(
        "CREATE INDEX idx_chunks_embedding_hnsw "
        "ON document_chunks USING hnsw (embedding vector_cosine_ops)"
    )
```

**Hints:** index ဆောက်ချိန်သည် row count၊ vector dimension နှင့် m/ef_construction setting ပေါ် မူတည်၍ တိုးလာသည်။ migration ကို production တွင် run ချိန်ကို ကြိုတွက်ဖို့ အရေးကြီးသည် — `CREATE INDEX CONCURRENTLY` က write lock ကို လျှော့ပေးသည် (Alembic တွင် autocommit block လိုအပ်မည်)။

**Expected behavior:** dataset ကြီးလာသည့်အတိုင်း index build ကြာချိန် တိုးလာသည်ကို ကြေ့ရပြီး migration ကြောင့် table lock မဖြစ်ပဲ index ပေါ်လာမည်။
