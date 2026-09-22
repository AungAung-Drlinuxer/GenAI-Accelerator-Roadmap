## လေ့ကျင်းခန်း ၁ — Embedding model ရွေးချယ်မှုနှင့် dimensionality

Embedding model တစ်ခုက text တစ်ကွက်ကို vector (နံပါတ်စာရင်း) တစ်ခုအဖြစ်ပြောင်းပေးပါတယ်။ Model တစ်ခုစီမှာ ထုတ်ပေးတဲ့ vector ရဲ့ အတိုအရှည် (dimension) က ကွဲပြားပြီး dimension များရင် သတင်းအချက်အလက် အသေးစိတ် ပိုပါပေမယ့် storage နှင့် တွက်ချက်မှုကုန်ကျစရိတ် ပိုများပါတယ်။ Hugging Face docs မှာ model card တိုင်းရော ရေးထားတဲ့ `sentence-transformers` စာမျက်ကမ်းမှာ output dimension ကို ဖတ်နိုင်ပါတယ်။

```python
# pip install sentence-transformers
from sentence_transformers import SentenceTransformer

# A small, CPU-friendly model; check its model card for the output size
model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

text = "pgvector lets PostgreSQL store and search vectors."

# One call returns a numpy array of shape (1, dim)
vec = model.encode(text)

print("Dimension:", vec.shape)        # e.g. (384,) for this model
print("First 5 values:", vec[0][:5])

# Different models produce different dimensions; you cannot mix them
# in one collection without a conversion step.
```

**အဓိကအယူအဆ** — Embedding model တစ်ခုချင်းစီရဲ့ vector dimension က ကွဲပြားပြီး မတူတဲ့ model တွေထဲက vector တွေကို collection တစ်ခုတည်းထဲ တိုက်ရိုက်ရောစပ်ထားလို့ မရပါ။

## လေ့ကျင့်ခန်း ၂ — Cosine, L2 နှင့် inner product ခွဲခြားရွေးချယ်ခြင်း

Vector နှစ်ခုကို နှိုင်းယှဉ်တဲ့ နည်းသုံးနည်းရှိပါတယ် — cosine similarity (ထောင့်ကိုပဲ ကြည့်တာ), L2 distance (အကွာအဝေးကို ကြည့်တာ), နှင့် inner product (အတိုင်းအတာနှင့် ထောင့် နှစ်ခုစလုံး ပေါင်းစပ်ကြည့်တာ)။ Vector တွေကို unit length အထိ normalize လုပ်ထားရင် နည်းသုံးနည်းလုံးရဲ့ အစီအစဉ်ပေးမှု (ranking) ဟာ တူညီသွားပါတယ်၊ ဒါကြောင့် RAG အတွက် များတောင်းများသလောက် cosine ကို ရွေးကြပါတယ်။

```python
import numpy as np

a = np.array([1.0, 2.0, 2.0])
b = np.array([2.0, 4.0, 4.0])   # same direction, longer
c = np.array([2.0, 2.0, 1.0])   # different direction, same-ish scale

def cosine_similarity(x, y):
    return float(np.dot(x, y) / (np.linalg.norm(x) * np.linalg.norm(y)))

def l2_distance(x, y):
    return float(np.linalg.norm(x - y))

def inner_product(x, y):
    return float(np.dot(x, y))

print("a vs b cosine :", cosine_similarity(a, b))  # 1.0, same direction
print("a vs c cosine :", cosine_similarity(a, c))  # less than 1.0
print("a vs b L2     :", l2_distance(a, b))
print("a vs c L2     :", l2_distance(a, c))
print("a vs b inner  :", inner_product(a, b))      # scale affects the result
```

**အဓိကအယူအဆ** — Vector တွေကို normalize လုပ်ထားပါက cosine, L2 နှင့် inner product သုံးနည်းလုံးရဲ့ ranking ဟာ အတူတူဖြစ်သဖြင့် RAG တွင် အသုံးအများဆုံး cosine က စိတ်ချရရွေးချယ်စရာဖြစ်ပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Vector normalization လုပ်ခြင်း

Normalization ဆိုတာ vector တစ်ခုရဲ့ အရှည်ကို 1 ဖြစ်အောင် လျှော့ပေးတဲ့ လုပ်ငန်းဖြစ်ပါတယ်။ Normalized vector တွေနဲ့ cosine similarity ကို တွက်ရင် ရိုးရိုး dot product တစ်ခုတည်းနဲ့ ရပါတယ်၊ query လုပ်တဲ့အခါမှာလည်း stored vectors နဲ့ query vector နှစ်ခုလုံးကို တူညီတဲ့ ပုံစံနဲ့ သိမ်းဆည်းထားဖို့ အရေးကြီးပါတယ်။

```python
import numpy as np

def normalize(v):
    norm = np.linalg.norm(v)
    return v / norm if norm > 0 else v

a = np.array([3.0, 4.0])
b = np.array([6.0, 8.0])  # same direction as a, different length

a_n = normalize(a)
b_n = normalize(b)

print("Norm of a after normalize:", np.linalg.norm(a_n))  # 1.0
print("Norm of b after normalize:", np.linalg.norm(b_n))  # 1.0

# For unit vectors, cosine similarity equals the dot product
print("Dot of normalized:", float(np.dot(a_n, b_n)))     # 1.0
```

**အဓိကအယူအဆ** — Vector တိုင်းကို unit length ဖြစ်အောင် normalize လုပ်ထားပါက cosine similarity ကို dot product ဖြင့် အလွယ်တွက်နိုင်ပြီး stored vector များနှင့် query vector အကြား ပုံစံတညီတည်း ဖြစ်စေပါသည်။

## လေ့ကျင့်ခန်း ၄ — Batch embedding ဖြင့် အမြန်နှုန်းတိုးစေခြင်း

Text တွေတစ်ခုချင်း encode လုပ်တာထက် batch အဖြစ် အတူတူ encode လုပ်တာက ပိုမြန်ပါတယ်၊ ဘာကြောင့်လဲဆိုတော့ model က padding နှင့် GPU/CPU အသုံးပြုမှုကို ပိုကောင်းအောင် စီမံနိုင်လို့ပါ။ Batch size က hardware ပေါ်မူတည်ပြီး ကြီးရင် memory ပြတ်တတ်ပါတယ် — model card နှင့် ကိုယ်ပိုင် machine အပေါ် မူတည်၍ စမ်းသပ်ပါ။

```python
# pip install sentence-transformers
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

docs = [
    "PostgreSQL is a relational database.",
    "pgvector adds vector columns to PostgreSQL.",
    "RAG combines retrieval with generation.",
    "Chunking splits long documents into pieces.",
    "Embeddings capture semantic meaning.",
]

# Encode all documents at once; result shape is (n_docs, dim)
emb = model.encode(docs, batch_size=16, show_progress_bar=False)

print("Batch shape:", emb.shape)  # (5, dim)
print("Row 0 shape:", emb[0].shape)

# Many models can normalize during encoding, check the model card
emb_norm = model.encode(docs, normalize_embeddings=True)
print("Norm of row 0:", float((emb_norm[0] ** 2).sum() ** 0.5))  # ~1.0
```

**အဓိကအယူအဆ** — Text များစွာကို batch တစ်ခုတည်းဖြင့် encode လုပ်ခြင်းက တစ်ခုချင်း လုပ်ခြင်းထက် ပိုမြန်ပြီး `normalize_embeddings=True` ဖြင့် normalize လုပ်ခြင်းကိုပါ encode အတွင်းမှာပဲ ပြုလုပ်နိုင်ပါသည်။

## လေ့ကျင့်ခန်း ၅ — pgvector ဖြင့် vector သိမ်းဆည်းခြင်းနှင့် versioning စီမံခြင်း

pgvector extension က PostgreSQL table ထဲ vector column တွေ ထည့်ပေးပြီး similarity search လုပ်ခွင့်ပေးပါတယ် (github.com/pgvector/pgvector)။ Vector တွေကို သိမ်းဆည်းတဲ့အခါ ဘယ် embedding model၊ ဘယ် version၊ ဘယ် dimension နဲ့ ထုတ်ခဲ့လဲဆိုတာကို metadata အဖြစ် မှတ်တမ်းတင်ထားဖို့ အရေးကြီးပါတယ် — model ပြောင်းတဲ့အခါ vector အသစ် ပြန်ထုတ်ရပါမယ်၊ ဒါမှမဟုတ် မတူတဲ့ vector တွေ ရောစပ်သွားပါလိမ့်မယ်။

```python
# pip install psycopg pgvector
import psycopg
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

DB = "postgresql://postgres:postgres@localhost:5432/testdb"
model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
MODEL_ID = "all-MiniLM-L6-v2"  # record which model made these vectors

docs = [
    "PostgreSQL stores relational data.",
    "pgvector adds vector search to PostgreSQL.",
    "Embeddings represent text as numbers.",
]
embs = model.encode(docs, normalize_embeddings=True)

with psycopg.connect(DB) as conn:
    register_vector(conn)
    with conn.cursor() as cur:
        cur.execute("CREATE EXTENSION IF NOT EXISTS vector;")
        # Record model id, version and dimension beside the vectors
        cur.execute("""
            CREATE TABLE IF NOT EXISTS chunks (
                id SERIAL PRIMARY KEY,
                content TEXT,
                embedding VECTOR(384),
                model_id TEXT,
                model_version TEXT
            );
        """)
        for content, emb in zip(docs, embs):
            cur.execute(
                "INSERT INTO chunks (content, embedding, model_id, model_version) "
                "VALUES (%s, %s, %s, %s)",
                (content, emb, MODEL_ID, "v1"),
            )
        # Only search among vectors from the same model version
        cur.execute(
            "SELECT content FROM chunks "
            "WHERE model_id = %s AND model_version = 'v1' "
            "ORDER BY embedding <=> %s LIMIT 2",
            (MODEL_ID, embs[0]),
        )
        for row in cur.fetchall():
            print("Match:", row[0])
    conn.commit()
```

**အဓိကအယူအဆ** — Vector တိုင်းနဲ့အတူ model id, version နှင့် dimension ကို metadata အဖြစ်သိမ်းဆည်းပြီး search လုပ်တဲ့အခါ model version တူတဲ့ vector များကိုသာ အသုံးပြုရမ်းသည်၊ model ပြောင်းလျှင် vector အားလုံးကို ပြန်လည်ထုတ်ယူရပါမည်။

## လေ့ကျင့်ခန်း ၆ — Model Versioning နှင့် Vector ပြန်လည်စိစစ်ခြင်း

Model တစ်ခုချင်းစီ၏ embedding ကို `model_name` column ဖြင့် version သတ်မှတ်ခြင်းသည် vector database ကို တာဝန်ခံနိုင်စွာ အသုံးပြုရန် အရေးကြီးဆုံးအလေ့အကျင့်တစ်ခုဖြစ်သည်။ Model တစ်ခုစီသည် ကွဲပြားသော vector dimension နှင့် semantic space ကို ထုတ်ပေးသဖြင့် မတူညီသော model များမှ vector များကို တိုက်ရိုက်နှိုင်းယှဉ်ပါက ရလဒ်များသည် အဓိပ္ပာယ်မရှိတော့ပါ။ ထို့ကြောင့် query တွင် `WHERE model_name = ...` filter ကို မဖြစ်မနေ ထည့်သွင်းပြီး dimension မတူညီမှုကြောင့်ဖြစ်သော runtime error များကိုလည်း ရှောင်ရှားရမည်။

အောက်ပါ Python code သည် model version ပြောင်းလဲသည့်အခါ re-embed လုပ်ခြင်းနှင့် version-specific query လုပ်ခြင်းကို ပြသည် —

```python
import psycopg2

# Each model produces vectors in its own dimensional space,
# so every stored row must record which model made its embedding.
old_version = "paraphrase-multilingual-MiniLM-L12-v2"
new_version = "all-MiniLM-L6-v2"

conn = psycopg2.connect(
    dbname="mydb", user="postgres", password="secret", host="localhost"
)
cur = conn.cursor()

def search_documents(query_embedding, model_name, top_k=5):
    # Filter by model_name so we only compare vectors
    # that live in the same embedding space.
    cur.execute(
        """
        SELECT content
        FROM documents
        WHERE model_name = %s
        ORDER BY embedding <=> %s::vector
        LIMIT %s
        """,
        (model_name, str(query_embedding), top_k),
    )
    return cur.fetchall()

def reembed_and_store(new_model, texts, embed_fn):
    # When switching models, re-embed every document from scratch
    # and store the vectors under the new version tag.
    vectors = embed_fn(new_model, texts)
    for text, vec in zip(texts, vectors):
        cur.execute(
            """
            INSERT INTO documents (content, embedding, model_name)
            VALUES (%s, %s::vector, %s)
            """,
            (text, str(vec), new_model),
        )
    conn.commit()

# Example usage:
# results = search_documents([0.1, 0.2, ...], old_version)
# reembed_and_store(new_version, ["doc 1", "doc 2"], my_embed_function)

cur.close()
conn.close()
```

Query တစ်ခုချင်းစီတွင် `model_name` filter ပါဝင်သည်နှင့် တစ်ပြိုင်နက် ရလဒ်များသည် တစ်ခုတည်းသော embedding space အတွင်းမှသာ ဆွဲထုတ်သဖြင့် အနီးစပ်ဆုံးရလဒ်များသည် ယုံကြည်စိတ်ချရပြီး dimension mismatch error များကိုပါ တုံ့ပြန်နိုင်သည်။

**အဓိကအယူအဆ** — မတူညီသော model များမှ ထွက်ပေါလာသော vector များကို ရောနှောမမိစေရန် embedding တစ်ခုစီအတွက် `model_name` version tag ကို မှတ်သားထားပြီး query တိုင်းတွင် ထို version ဖြင့် filter လုပ်ရမည်။
