# PostgreSQL + pgvector + Alembic

## Documents နှင့် Chunks အတွက် Schema ဒီဇိုင်း

### ဘာကို ဆိုလိုတာလဲ

RAG (Retrieval-Augmented Generation) စနစ်တစ်ခုမှာ မှတ်တမ်းတွေ (documents) ကို အချော့အချော့ ဖြတ်ပြီး chunks လို့ခေါ်တဲ့ အပိုင်းငယ်လေးများအဖြစ် သိမ်းဆည်းလေ့ရှိသည်။ Schema ဒီဇိုင်းဆိုသည်မှာ ဒီ document နှင့် chunk အချက်အလက်များကို PostgreSQL tables များအဖြစ် စနစ်တကျ စီစဉ်ဖန်တီးပေးခြင်းကို ဆိုလိုသည်။

### ဘာကြောင့် လဲ

Document တစ်ခုလုံးကို embedding တစ်ခုတည်းနှင့် ကိုယ်စားပြုပါက အနက်အဓိပ္ပာယ် အသေးစိတ် ပျောက်ဆုံးတတ်သည်။ အပိုင်းငယ်များအဖြစ် ဖြတ်တောက်ပြီး တစ်ခုချင်းစီကို embedding ပြုလုပ်မှသာ ရှာဖွေမှု အရည်အသွေး မြင့်မားသည်။ ထို့ကြောင့် documents နှင့် chunks ဟူ၍ ဇယားနှစ်ခု ခွဲခြားဖန်တီးရန် လိုအပ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`documents` ဇယားမှာ မူရင်း စာသား၊ အရင်းအမြစ်၊ ဖန်တီးချိန် စသည့် metadata များကို သိမ်းသည်။ `chunks` ဇယားမှာ document ၏ id (foreign key)၊ အပိုင်းစာသား၊ အပိုင်းအမှတ်စဉ်၊ embedding vector တို့ကို သိမ်းသည်။ ဇယားနှစ်ခုကြားမှာ `FOREIGN KEY` ဆက်ဆံရေးဖြင့် document တစ်ခုပျက်စီးလျှင် သူ့အပိုင်းများပါ အလိုအလျောက် ပျက်စီးစေရန် `ON DELETE CASCADE` ကို သုံးနိုင်သည်။

### ဥပမာ

```python
# example_schema.py - show the SQL schema for documents and chunks
schema_sql = """
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    source TEXT NOT NULL,          -- where the document came from
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE chunks (
    id BIGSERIAL PRIMARY KEY,
    document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    chunk_index INT NOT NULL,      -- position of this chunk inside the document
    content TEXT NOT NULL,        -- the raw text of this chunk
    embedding vector(1536)        -- pgvector column, fixed dimension
);
"""
print(schema_sql)
# Expected output:
# CREATE TABLE documents (
#     id BIGSERIAL PRIMARY KEY,
#     source TEXT NOT NULL,
#     created_at TIMESTAMPTZ DEFAULT now()
# );
#
# CREATE TABLE chunks (
#     id BIGSERIAL PRIMARY KEY,
#     document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
#     chunk_index INT NOT NULL,
#     content TEXT NOT NULL,
#     embedding vector(1536)
# );
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production စနစ်တွေမှာ document များကို ပြန်ဖျက်ရန်၊ ပြန်စုရန်၊ metadata အရ စစ်ထုတ်ရန် လိုအပ်ချိန် ရောက်လာသည်။ Schema ကောင်းမှန်လျှင် ဒီလို စီမံခန့်ခွဲမှုများ လွယ်ကူပြီး၊ schema မှားလျှင် data အရမ်းများလာသောအခါ ပြင်ရခြင်း အလွန်ခက်ခဲသည်။

## Embedding Column Types (vector type)

### ဘာကို ဆိုလိုတာလဲ

pgvector extension က PostgreSQL ထဲမှာ `vector` ဟူသော ဒေတာအမျိုးအစား (data type) အသစ် ထည့်ပေးသည်။ ၎င်းကို embedding များသိမ်းရန် အသုံးပြုပြီး dimension ကို အတိအက သတ်မှတ်နိုင်သည် — ဥပမာ `vector(1536)` ဆိုလျှင် အရွယ် 1536 ရှိတဲ့ vector များသာ လက်ခံသည်။

### ဘာကြောင့် လဲ

ပုံမှန် PostgreSQL မှာ vector များကို `TEXT` ဒါမျမ `JSONB` အဖြစ် သိမ်းနိုင်သော်လည်း ရှာဖွေမှု တွက်ချက်နိုင်စွမ်း မရှိပါ။ `vector` type ကို သုံးမှသာ distance operator များ၊ ANN index များနှင့် တိုက်ရိုက် အလုပ်လုပ်နိုင်သည်။ Dimension ကို သတ်မှတ်ခြင်းဖြင့် မတူညီတဲ့ embedding model များက vector များ ရောနှောထည့်သွင်းမိခြင်းကို ကာကွယ်ပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

အသုံးပြုမနေုင်မီ `CREATE EXTENSION vector;` ဖြင့် extension ကို enable လုပ်ရသည်။ Vector ထည့်သွင်းရာမှာ `[0.1, 0.2, ...]` ပုံစံဖြင့် string literal အဖြစ် ရေးရသည်။ Python ဘက်မှ `psycopg` သို့မဟုတ် `pgvector` တဲ့ Python library က tuple ဒါမျ list များကို အလိုအလျောက် ပြောင်းပေးသည်။

### ဥပမာ

```python
# example_insert.py - insert an embedding row using the pgvector python helper
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("dbname=test", autocommit=True)
register_vector(conn)  # teaches psycopg how to encode/decode vector columns

embedding = [0.1, 0.2, 0.3]  # normally produced by an embedding model
conn.execute(
    "INSERT INTO chunks (document_id, chunk_index, content, embedding) "
    "VALUES (%s, %s, %s, %s)",
    (1, 0, "hello world", embedding),
)
row = conn.execute(
    "SELECT content, embedding FROM chunks WHERE document_id = %s", (1,)
).fetchone()
print(row[0], row[1])
# Expected output:
# hello world [0.1 0.2 0.3]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Model ပြောင်းလျှင် embedding dimension ပြောင်းတတ်သည် (ဥပမာ 1536 → 768)။ Dimension သတ်မှတ်ထားခြင်းက ဒီလို မတည်ငြာမှုများကို စနစ်အစောပိုင်းမှာပဲ ဖမ်းဆုပ်နိုင်စေပြီး production ထဲမှာ တိတ်တဆိတ် အချက်အလက်ပျက်စီးမှုများ ကာကွယ်ပေးသည်။

## HNSW vs IVFFlat Index

### ဘာကို ဆိုလိုတာလဲ

Vector သန်းပေါင်းများကြားမှ အနီးစပ်ဆုံးနေအိမ်များ (nearest neighbors) ကို တစ်ခုချင်း စစ်ဆေးခြင်း (exact scan) က နှေးသည်။ ထို့ကြားမှာ pgvector က approximate nearest neighbor (ANN) index နှစ်မျိုး ပေးသည် — HNSW (Hierarchical Navigable Small World) နှင့် IVFFlat (Inverted File with Flat scanning)။

### ဘာကြောင့် လဲ

ANN index များက အချို့ accuracy စွန့်လွှတ်ပြီး ရှာဖွေမှု speed ကို ဆုံးရှုံးများစွာ မြှင့်တင်ပေးသည်။ Data အရွယ်အစား၊ ရှာဖွေမှုလုံးရဲရဲ (recall) နှင့် build ချိန်တို့ အပေါ် မူတည်ပြီး ဒီ index နှစ်မျိုးက အားသာချက် ကွဲပြားသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

**IVFFlat** က vector များကို cluster များ (lists) အဖြစ် အုပ်စုခွဲပြီး ရှာဖွေသောအခါ အနီးစပ်ဆုံး cluster အချို့သာ စစ်ဆေးသည်။ Build မြန်သော်လည်း data အသစ် ထည့်လိုက်လျှင် cluster ဗလုံဗလုံ မှန်ကန်မှု လျော့နည်းတတ်သည်။ **HNSW** က multi-layer graph တည်ဆောက်ပြီး ရှာဖွေမှု ပိုမြန်ပြီး recall ပိုမြင့်သော်လည်း build ချိန် ပိုကြာပြီး memory ပိုသုံးသည်။ IVFFlat ကို build ရန် data အခြေအနေ (training data) ရှိနေရပြီး၊ HNSW က data မရှိချင် ရှိသည့်အတိုင်း ဆောက်နိုင်သည်။

### ဥပမာ

```python
# example_index.py - compare the two index creation statements
ivfflat_sql = (
    "CREATE INDEX ON chunks USING ivfflat (embedding "
    "vector_cosine_ops) WITH (lists = 100);"
)
hnsw_sql = (
    "CREATE INDEX ON chunks USING hnsw (embedding "
    "vector_cosine_ops) WITH (m = 16, ef_construction = 64);"
)
print(ivfflat_sql)
print(hnsw_sql)
# Expected output:
# CREATE INDEX ON chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
# CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Data နည်းစဉ်တုန်းက IVFFlat နဲ့ စမ်းကြည့်ပြီး data ကြီးလာရင် HNSW ကို ပြောင်းတာက အများအားဖြင့် သင့်တော်သည်။ Index ရွေးချယ်မှုဟာ ရှာဖွေမှုလုံးရဲရဲ (recall) နှင့် resource အသုံးအစွဲ ကြားထဲက ဆုံးဖြတ်ချက်တစ်ခုဖြစ်၍ production ဒီဇိုင်းမှာ ဗဟိုအခန်းကပါသည်။

## Distance Operators (distance operators)

### ဘာကို ဆိုလိုတာလဲ

pgvector က vector နှစ်ခုကြား အကွာအဝေး (သို့မဟုတ် similarity) တွက်ရန် SQL operators သုံးမျိုး ပေးသည် — `<->` (L2 / Euclidean distance)၊ `<#>` (negative inner product)၊ `<=>` (cosine distance နှင့် inner product အတွက်)။

### ဘာကြောင့် လဲ

ရှာဖွေမှုအရည်အသွေးဟာ embedding model သုံးတဲ့ metric နဲ့ ကိုက်ညီမှုအပေါ် မူတည်သည်။ Model ထွက် embedding များကို cosine similarity နှင့် တွက်ရန် ရည်ရွယ်ထားလျှင် `<->` (L2) ဖြင့် စစ်လျှင် အကွာအဝေး အစဉ်အလိုက် မတူတော့ဘဲ ရလဒ် အနည်းငယ် ယိုယွင်းနိုင်သည်။ ထို့ကြားမှာ index ကိုလည်း တွဲဖက် operator နှင့်အညီ ဆောက်ရမည် — ဥပမာ `vector_cosine_ops`။

### ဘယ်လို အလုပ်လုပ်လဲ

`ORDER BY embedding <=> query_vector LIMIT 5` ဆိုလျှင် query vector နှင့် အနီးစပ်ဆုံး အပိုင်း ငါးခုကို ပြန်ပေးသည်။ `<=>` က cosine distance ဖြစ်၍ တန်ဖိုး နည်းလေ ဆင်တူလေဖြစ်သည်။ Query vector ကို parameter အဖြစ် တူရှယ်ပြီး SQL injection ကာကွယ်ရန် placeholder သုံးသင့်သည်။

### ဥပမာ

```python
# example_search.py - nearest-neighbor search with the cosine operator
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("dbname=test", autocommit=True)
register_vector(conn)

query_embedding = [0.1, 0.2, 0.3]  # embedding of the user question
rows = conn.execute(
    "SELECT content, embedding <=> %s AS distance "
    "FROM chunks ORDER BY distance LIMIT 3",
    (query_embedding,),
).fetchall()
for content, distance in rows:
    print(f"{content}  ->  distance={distance:.4f}")
# Expected output:
# hello world  ->  distance=0.0000
# (plus other nearest chunks with their cosine distances)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

ရှာဖွေမှုရလဒ် အစဉ်အလိုက် မှန်ကန်မှုဟာ RAG စနစ်ရဲ့ အသွင်အပြင်ပေါ် တိုက်ရိုက်သက်ရောက်သည်။ Operator မှားသုံးခြင်း (ဒါမျမ index နှင့် မကိုက်တဲ့ operator) က index ကို လုံးဝမသုံးနိုင်စေပြီး query များ တဖြည်းဖြည်း ဖြစ်သွားစေသည်။

## Alembic ဖြင့် Migrations

### ဘာကို ဆိုလိုတာလဲ

Alembic ဆိုသည်မှာ SQLAlchemy အသုံးပြုသည့် စနစ်များအတွက် database schema ပြောင်းလဲမှုများ (migrations) ကို ဗားရှင်းတစ်ခုချင်း မှတ်တမ်းတင်ပြီး ထိန်းသိမ်းစီမံပေးတဲ့ tool ဖြစ်သည်။

### ဘာကြောင့် လဲ

Table များကို လက်ဖြင့် တိုက်ရိုက် `CREATE TABLE` လုပ်ခြင်းက development မှာ ရပါသော်လည်း production နှင့် teammate များရဲ့ database များနှင့် ကွဲပြားသွားစေသည်။ Migration files များက schema ပြောင်းလဲမှု မှတ်တမ်းအပြည့်အစုံ ပေးပြီး ရှေ့တိုး / နောက်ပြန် (upgrade/downgrade) ဆွဲနိုင်စေသည်။

### ဘယ်လို အလုပ
