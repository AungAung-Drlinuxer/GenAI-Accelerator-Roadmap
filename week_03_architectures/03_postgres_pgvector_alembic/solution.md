# Week 3 — AI Architectures & Containerization
## Module: PostgreSQL + pgvector + Alembic — ဖြေဆိုများ (solution.md)

## လေ့ကျင့်ခန်း ၁ — Documents/Chunks အတွက် Schema ဒီဇိုင်း

Documents ဇယားနှင့် chunks ဇယားကို foreign key ဖြင့် ချိတ်ဆက်ပြီး စာသားရှည်များအတွက် `TEXT`၊ ရင်းမြစ်အချက်အလက်များအတွက် `JSONB` ကို အသုံးပြုသည်။

```python
from sqlalchemy import create_engine, Column, String, Text, DateTime, ForeignKey, JSON
from sqlalchemy.orm import declarative_base, relationship
import datetime

Base = declarative_base()

class Document(Base):
    # One row per ingested source document
    __tablename__ = "documents"
    id = Column(String, primary_key=True)
    source_url = Column(String, nullable=True)
    metadata_ = Column("metadata", JSON, default=dict)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    chunks = relationship("Chunk", back_populates="document", cascade="all, delete-orphan")

class Chunk(Base):
    # One row per text chunk split from a document
    __tablename__ = "chunks"
    id = Column(String, primary_key=True)
    document_id = Column(String, ForeignKey("documents.id", ondelete="CASCADE"))
    chunk_index = Column(String)  # order of the chunk inside the document
    content = Column(Text, nullable=False)
    embedding = Column(None)  # pgvector Vector column, added in exercise 2
    document = relationship("Document", back_populates="chunks")

engine = create_engine("postgresql+psycopg2://user:pass@localhost:5432/vectordb")
Base.metadata.create_all(engine)
```

**အဓိကအယူအဆ** — Document တစ်ခုလျှင် chunks များစွာရှိသော one-to-many ဆက်ဆံရေးကို `document_id` foreign key ဖြင့် သတ်မှတ်ပြီး မိခင် document ကို ဖျက်လျှင် chunks လည်း `CASCADE` ဖြင့် ပြိုကွဲသွားစေရမည်။

## လေ့ကျင့်ခန်း ၂ — Embedding Column အမျိုးအစားများ

`pgvector` extension ထည့်သွင်းပြီး `Vector(n)` column တစ်ခုကို chunks ဇယားသို့ ထည့်ရေးသည်။

```python
from sqlalchemy import text
from pgvector.sqlalchemy import Vector

# Run once after connecting to the database
with engine.begin() as conn:
    # Enable the pgvector extension in the current database
    conn.execute(text("CREATE EXTENSION IF NOT EXISTS vector"))

from sqlalchemy import Table
# Add the embedding column (dimension must match your model, e.g. 1536)
with engine.begin() as conn:
    conn.execute(text(
        "ALTER TABLE chunks ADD COLUMN IF NOT EXISTS embedding vector(1536)"
    ))

# Insert a chunk with a fake embedding of matching dimension
import numpy as np
fake_embedding = np.zeros(1536, dtype=float).tolist()
with engine.begin() as conn:
    conn.execute(text(
        "INSERT INTO chunks (id, document_id, chunk_index, content, embedding) "
        "VALUES (:id, :doc, :idx, :content, :emb)"
    ), {"id": "c1", "doc": "d1", "idx": "0",
        "content": "hello world", "emb": str(fake_embedding)})
```

**အဓိကအယူအဆ** — `vector(1536)` ကဲ့သို့ dimension ကို တိတိကျကျ သတ်မှတ်ရခြင်းကြောင့် embedding model တစ်ခုနှင့် တစ်ခု ပြောင်းလဲလျှင် column အမျိုးအစားလည်း အသစ်ပြန်ပြင်ရမည်။

## လေ့ကျင့်ခန်း ၃ — HNSW နှင့် IVFFlat Index နှိုင်းယှဉ်ခြင်း

HNSW index တည်ဆောက်ခြင်းနှင့် IVFFlat index တည်ဆောက်ခြင်းကို SQL ဖြင့် လက်တွေ့စမ်းသပ်သည်။

```python
with engine.begin() as conn:
    # HNSW: graph-based index, no training step needed, works on an empty table
    conn.execute(text(
        "CREATE INDEX IF NOT EXISTS chunks_hnsw_idx "
        "ON chunks USING hnsw (embedding vector_cosine_ops)"
    ))

# IVFFlat requires existing data to build cluster centroids (lists)
with engine.begin() as conn:
    # lists=100 is a common starting point; requires rows to already exist
    conn.execute(text(
        "CREATE INDEX IF NOT EXISTS chunks_ivf_idx "
        "ON chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100)"
    ))

# Compare sizes of both indexes
with engine.begin() as conn:
    result = conn.execute(text(
        "SELECT indexrelid::regclass AS idx, pg_relation_size(indexrelid) AS size "
        "FROM pg_indexes WHERE tablename = 'chunks'"
    ))
    for row in result:
        print(row.idx, row.size)
```

**အဓိကအယူအဆ** — HNSW သည် data မရှိခင် တည်ဆောက်နိုင်ပြီး query မြန်သော်လည်း index ဖိုင်ကြီးသည်၊ IVFFlat သည် data ရှိမှ တည်ဆောက်နိုင်ပြီး index သေးသော်လည်း `lists` တန်ဖိုးကို သင့်တင့်ရွေးရမည်။

## လေ့ကျင့်ခန်း ၄ — Distance Operators ( <->, <=>, <#> )

pgvector ၏ distance operator သုံးမျိုးကို အသုံးပြု၍ nearest neighbor ရှာဖွေမှု စမ်းသပ်သည်။

```python
query_vec = np.zeros(1536, dtype=float).tolist()

with engine.begin() as conn:
    # <=> : cosine distance (lower is more similar)
    rows = conn.execute(text(
        "SELECT id, content, embedding <=> :qv AS distance "
        "FROM chunks ORDER BY embedding <=> :qv LIMIT 5"
    ), {"qv": str(query_vec)}).fetchall()
    for r in rows:
        print("cosine:", r.id, r.distance)

    # <-> : L2 (Euclidean) distance
    rows = conn.execute(text(
        "SELECT id, embedding <-> :qv AS distance "
        "FROM chunks ORDER BY embedding <-> :qv LIMIT 5"
    ), {"qv": str(query_vec)}).fetchall()

    # <#> : negative inner product (use only with ivfflat/ip index type)
    rows = conn.execute(text(
        "SELECT id, embedding <#> :qv AS dist "
        "FROM chunks ORDER BY embedding <#> :qv LIMIT 5"
    ), {"qv": str(query_vec)}).fetchall()
```

**အဓိကအယူအဆ** — Cosine distance `<=>` သည် text embedding များအတွက် အသုံးအများဆုံးဖြစ်ပြီး၊ index တည်ဆောက်စဉ် ရွေးချယ်သော operator class (ဥပမာ `vector_cosine_ops`) နှင့် query operator တို့ တစ်ထပ်တည်းဖြစ်ရမည်။

## လေ့ကျင့်ခန်း ၅ — Alembic Migration ဖြင့် Schema ပြောင်းလဲမှုများ

Alembic setup ပြုလုပ်ပြီး embedding column နှင့် HNSW index ထည့်သွင်းသော migration တစ်ခု ရေးသည်။

```python
# Terminal commands (run in your project root):
#   pip install alembic
#   alembic init migrations
# Then edit migrations/env.py to set target_metadata and database URL.

# migrations/versions/0001_add_embedding.py
"""add embedding column and hnsw index

Revision ID: 0001
Revises:
"""
from alembic import op
import sqlalchemy as sa

revision = "0001"
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Add the pgvector extension if it is not installed yet
    op.execute("CREATE EXTENSION IF NOT EXISTS vector")
    # Add a 1536-dimension embedding column to chunks
    op.execute("ALTER TABLE chunks ADD COLUMN embedding vector(1536)")
    # Build an HNSW index for cosine similarity searches
    op.execute(
        "CREATE INDEX chunks_hnsw_idx ON chunks "
        "USING hnsw (embedding vector_cosine_ops)"
    )

def downgrade():
    # Reverse the migration: drop index, column, then extension
    op.execute("DROP INDEX IF EXISTS chunks_hnsw_idx")
    op.execute("ALTER TABLE chunks DROP COLUMN IF EXISTS embedding")
    op.execute("DROP EXTENSION IF EXISTS vector")
```

**အဓိကအယူအဆ** — Schema ပြောင်းလဲမှုတိုင်းကို Alembic migration file တစ်ခုစီဖြင့် မှတ်တမ်းတင်ပြီး `upgrade()` နှင့် `downgrade()` နှစ်မျိုးလုံး ရေးသားထားမှသာ တစ်ဆင့်ပြန်ရုံ့နိုင်မည်။

## လေ့ကျင့်ခန်း ၆ — Index တည်ဆောက်ရန်ကုန်ကျစရိတ် တိုင်းတာခြင်း

Index တည်ဆောက်ချိန်ကို Python ဖြင့် တိုင်းတာပြီး HNSW build parameters ၏သက်ရောက်မှုကို လေ့လာသည်။

```python
import time

def timed_build(sql: str) -> float:
    # Execute one CREATE INDEX statement and return elapsed seconds
    start = time.perf_counter()
    with engine.begin() as conn:
        conn.execute(text(sql))
    return time.perf_counter() - start

# HNSW with more links per node (m) and wider candidate list (ef_construction)
elapsed = timed_build(
    "CREATE INDEX chunks_hnsw_tuned_idx ON chunks "
    "USING hnsw (embedding vector_cosine_ops) "
    "WITH (m = 16, ef_construction = 200)"
)
print(f"HNSW (m=16, ef_construction=200) build took {elapsed:.2f}s")

# Drop the experimental index so the exercise can be re-run
with engine.begin() as conn:
    conn.execute(text("DROP INDEX IF EXISTS chunks_hnsw_tuned_idx"))
```

**အဓိကအယူအဆ** — `m` နှင့် `ef_construction` တန်ဖိုးများ ကြီးလာလေ index အရည်အသွေး (query တိကျမှု) တိုးလေဖြစ်သော်လည်း တည်ဆောက်ချိန်နှင့် memory အသုံးအစား ပိုမိုကြီးမားလေဖြစ်သည်။
