# Week 4 — Embeddings & Vector Store (Embedding နှင့် Vector သိုလှောင်မှု)

## Embedding Model နှင့် Dimensionality

### ဘာကို ဆိုလိုတာလဲ
Embedding model ဆိုသည်မှာ စာသား (text) ကို ဂဏန်းအမျိုးအစားများသာ ပါဝင်သော vector တစ်ခုအဖြစ်ပြောလှဲပေးသည့် model ဖြစ်သည်။ ဥပမာ — `sentence-transformers` မှ `all-MiniLM-L6-v2` ဆိုသည်မှာ စာကြောင်းတစ်ကြောင်းကို ဂဏန်း 384 ခုပါဝင်သော list တစ်ခုအဖြစ်ပြောလှဲပေးသည်။ ထိုဂဏန်းအရေအတွက်ကို "dimension" (dimensionality) ဟုခေါ်သည်။

### ဘာကြောင့် လဲ
Computer သည် စာသားကို တိုက်ရိုက်နားမလည်နိုင်သည်။ သို့သော် ဂဏန်းများကိုမူ မြန်ဆန်စွာ တွက်ချက်နိုင်သည်။ စာသားအဓိပ္ပာယ်များကို ဂဏန်း space တစ်ခုအတွင်း နေရာချထားပေးခြင်းအားဖြင့် "အဓိပ္ပာယ်အားဖြင့် နီးစောင်းသော စာကြောင်းများ" ကို ရှာဖွေနိုင်သည်။ Dimension ကြီးလေ စာကြောင်းတစ်ခုချင်းစီ၏ အသေးစိတ်အချက်အလက် ပိုမိုဖမ်းစားနိုင်လေ ဖြစ်သော်လည်း သိုလှောင်မှုနှင့် တွက်ချက်မှုစရိတ်လည်း တိုးလေသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Model အတွင်းရှိ neural network သည် input စာသားကို token များအဖြစ် ခွဲ၍ အလွှာများဖြင့် တွက်ချက်ပြီး နောက်ဆုံးတွင် ကိန်းရှည်များ (float များ) တစ်စု ထုတ်ပေးသည်။ ထိုကိန်းရှည်များကိ်ု NumPy array သို့မဟုတ် list အဖြစ် ယူ၍ အခြားစာကြောင်းများနှင့် နှိုင်းယှဉ်နိုင်သည်။

### ဥပမာ

```python
from sentence_transformers import SentenceTransformer

# Load a small, CPU-friendly embedding model
model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

# Embed a single sentence and inspect its shape
embedding = model.encode("RAG helps reduce hallucination in LLMs.")
print(type(embedding))
print(embedding.shape)
print(embedding[:5])
# Expected output:
# <class 'numpy.ndarray'>
# (384,)
# [ 0.0453 -0.0217  0.0831 -0.0562  0.0398]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Vector store တစ်ခုရွေးချယ်ရာတွင် သင်၏ embedding model ၏ dimension နှင့် ကိုက်ညီရမည်။ pgvector တွင် `vector(384)` ကဲ့သို့ column type သတ်မှတ်ရာတွင် ထို dimension အရေအတွက်ကို အတိအကျ ရေးရသည်။ မတိုက်ဆိုင်ပါက insert လုပ်ချင်း မအောင်မြင်ပါ။

## Cosine / L2 / Inner Product — ခြားနားချက် ရွေးချယ်ခြင်း

### ဘာကို ဆိုလိုတာလဲ
Vector နှစ်ခုက "အဓိပ္ပာယ်အားဖြင့် နီးစောင်းလေသလော" ကို တိုင်းတာရန် နည်းလမ်းသုံးမျိုး အဓိကရှိသည် —
- **Cosine similarity**: ထောင့် (angle) ကိုသာ ကြည့်သည်။ 1.0 ဆိုတာ အာရုံတူညီခြင်း။
- **L2 (Euclidean) distance**: အမှတ်နှစ်ခုကြား အကွာအဝေး။ 0 ဆိုတာ လုံးဝတူညီခြင်း။
- **Inner product (dot product)**: ဂဏန်းအလွှာပြင်း မပါဘဲ တိုက်ရိုက်မြှောက်ခြင်း။

### ဘာကြောင့် လဲ
စာသားရှာဖွေမှုတွင် စာကြောင်းတစ်ခုချင်းစီသည် ရှည်လား တိုလား (vector ၏ magnitude) အပေါ် မူတည်၍ ပမာဏ ကွဲပြားနိုင်သည်။ အဓိပ္ပာယ်နှိုင်းယှဉ်မှုအတွက် ထောင့်သည် ပို၍ အရေးကြီးသောကြောင့် cosine ကို အသုံးများသည်။ သို့သော် vector များကို normalize လုပ်ထားပါက နည်းလမ်းသုံးမျိုးလုံး အဓိပ္ပာယ်အားဖြင့် တူညီသွားသည် — ထိုအခါ တွက်ချက်မှု အမြန်ဆုံးနည်းကို ရွေးလို့ရသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Cosine သည် ထောင့်ကို တိုင်းတာ၍ magnitude ကို ဖျောက်ဖျက်သည်။ L2 သည် ရင်းနှီးမှုနှင့် ပမာဏ နှစ်ခုစလုံးပေါင်းစပ်သည်။ Inner product သည် normalize မလုပ်ထားပါက magnitude ကြီးသော vector က ရလဒ်ကို ဆွဲငင်နိုင်သည်။

### ဥပမာ

```python
import numpy as np

a = np.array([1.0, 2.0, 2.0])
b = np.array([2.0, 4.0, 4.0])   # same direction, larger magnitude
c = np.array([-1.0, -2.0, -2.0])  # opposite direction

def cosine_similarity(x, y):
    # Angle-based comparison, magnitude ignored
    return np.dot(x, y) / (np.linalg.norm(x) * np.linalg.norm(y))

def l2_distance(x, y):
    # Straight-line distance between two points
    return np.linalg.norm(x - y)

print("cos(a, b) =", cosine_similarity(a, b))   # 1.0 -> same meaning
print("cos(a, c) =", cosine_similarity(a, c))    # -1.0 -> opposite meaning
print("L2(a, b)  =", l2_distance(a, b))
print("L2(a, c)  =", l2_distance(a, c))
# Expected output:
# cos(a, b) = 1.0
# cos(a, c) = -1.0
# L2(a, b)  = 3.0
# L2(a, c)  = 6.0
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
pgvector တွင် `vector_cosine_ops`, `vector_l2_ops`, `vector_ip_ops` ဟူ၍ index operator များရှိပြီး ရွေးချယ်မှုသည် query ရလဒ်အပေါ် သက်ရောက်သည်။ စာသား semantic search အတွက် cosine ကို စံအဖြစ် အသုံးများသည်။

## Normalization

### ဘာကို ဆိုလိုတာလဲ
Normalization ဆိုသည်မှာ vector ၏ အရှည် (norm) ကို 1 ဖြစ်စေရန် ဂဏန်းအားလုံးကို တူညီသော တန်ဖိုးဖြင့် စားခြင်းဖြစ်သည်။ ရလဒ်မှာ "unit vector" ဖြစ်ပြီး လမ်းညွှန် (direction) ချည်းသာ ကျန်ရှိသည်။

### ဘာကြောင့် လဲ
Normalize လုပ်ထားသော vector များတွင် cosine, L2 နှင့် inner product တို့၏ အဆင့်အတန်း (ranking) သည် တူညီသွားသည်။ ထို့ကြောင့် inner product (တွက်ချက်မှု ပိုမြန်သည်) ဖြင့် cosine ရလဒ် ရယူလိုက်နိုင်သည်။ `sentence-transformers` model အများစုသည် output vector များကို မူလကပင် normalize လုပ်ပေးထားသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Vector ၏ အချိုးများကို မပြောင်းဘဲ အရှည်ကိုသာ 1 အဖြစ် ပြောင်းလိုက်ခြင်းဖြစ်သည် — ထောင့် မပြောင်းပါ။

### ဥပမာ

```python
import numpy as np

v = np.array([3.0, 4.0])

# Normalize: divide by the vector's length so the length becomes 1
norm = np.linalg.norm(v)
v_normalized = v / norm

print("before:", v, "length =", np.linalg.norm(v))
print("after :", v_normalized, "length =", np.linalg.norm(v_normalized))
# Expected output:
# before: [3. 4.] length = 5.0
# after  : [0.6 0.8] length = 1.0
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Model အမျိုးမျိုးမှ vector များကို ရောနှောသိမ်းဆည်းခဲ့ပါက (ဥပမာ — model တင်ပြီး version ပြောင်းခဲ့ပါက) normalization မတူညီမှုက search ranking ကို ပျက်စေနိုင်သည်။ သိုလှောင်မည့် vector အားလုံးကို တစ်သတ်မှတ်တည်း normalize လုပ်ရန် သတိပြုရသည်။

## Batch Embedding

### ဘာကို ဆိုလိုတာလဲ
စာကြောင်းတစ်ခုချင်းစီ encode လုပ်ခြင်းအစား စာကြောင်းများစွာကို တစ်ပြိုင်တည်း (batch တစ်ခုအဖြစ်) model ထဲသို့ တိုက်ရိုက် ပို့ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Batch processing သည် GPU/CPU ကို ပိုမိုအပြည့်အဝ အသုံးချနိုင်၍ စာကြောင်း ထောင်ပေါင်းများကို embedding လုပ်ရာတွင် အရေးကြီးသည့် အချိန်ချွေရှင်းမှု ရရှိစေသည်။ Loop တစ်ခုချင်း encode လုပ်ခြင်းထက် များစွာ ပိုမြန်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
`SentenceTransformer.encode()` သည် Python list တစ်ခုကို တန်းခံနိုင်ပြီး `(num_sentences, dim)` shape ပါသော matrix တစ်ခု ထုတ်ပေးသည်။

### ဥပမာ

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

sentences = [
    "How do I reset my password?",
    "I forgot my login credentials.",
    "What is the weather today?",
    "Please help me regain account access.",
]

# Embed all sentences in one call (batch)
matrix = model.encode(sentences)
print(matrix.shape)

# Find the most similar sentence to the first one using cosine
similarities = model.similarity(matrix[0], matrix[1:])
print(similarities)
# Expected output:
# (4, 384)
# tensor([[0.8021, 0.1476, 0.6638]])
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
RAG စနစ်တစ်ခုကို စတင်တည်ဆောက်ချင်သည့်အခါ document ရာပေါင်းများစွာကို တစ်ခါတည်း vector ပြောလှဲ၍ database ထဲသို့ တင်ရသည်။ Batch embedding မရှိပါက ထိုအဆင့်မှာ အချိန်အများကြီး ကုန်ဆုံးမည်။

## Storing နှင့် Versioning Vectors (pgvector ဖြင့်)

### ဘာကို ဆိုလိုတာလဲ
Vector များကို PostgreSQL database အတွင်း `vector` column type ဖြင့် သိုလှောင်ခြင်း၊ ထို့ပြင် vector များကို မည်သည့် model ၏ မည်သည့် version မှ လာသည်ကို မှတ်တမ်းတင်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Embedding model ကို ပြောင်းလဲပါက vector dimension သာမက အဓိပ္ပာယ် space ပါ ပြောင်းသွားသည်။ ထို့ကြောင့် တစ်စုံတစ်ခုကို ရှာတော့မိသည့်အခါ မတူညီသော model ကလာသော vector များကို နှိုင်းယှဉ်မိပါက ရလဒ်များ အဓိပ္ပာယ်မရှိတော့ပါ။ Version column တစ်ခု ထားရှိခြင်းဖြင့် ရှောင်ရှောင်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
pgvector extension ကို enable လုပ်၍ `vector(384)` column ပါသော table တည်ဆောက်ပြီး embedding များကို insert လုပ်သည်။ Model အမည်နှင့် မည်သည့်အချိန်က ထုတ်ခဲ့သည့် vector များဖြစ်သည်ကို metadata အဖြစ် အတူတကွ သိမ်းဆည်းသည်။

### ဥပမာ

```python
# SQL setup (run once, e.g. via psql):
# CREATE EXTENSION IF NOT EXISTS vector;
# CREATE TABLE documents (
#     id BIGSERIAL PRIMARY KEY,
#     content TEXT,
#     embedding vector(384),
#     model_name TEXT NOT NULL,      -- e.g. 'all-MiniLM-L6-v2'
#     model_version TEXT NOT NULL    -- your own version tag, e.g. 'v1'
# );

# Python example inserting and querying with psycopg + pgvector
import psycopg
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

with psycopg.connect("postgresql://user:pass@localhost/mydb") as conn:
    register_vector(conn)
    text = "RAG combines retrieval with generation."
    emb = model.encode(text)

    with conn.cursor() as cur:
        # Store the vector together with model metadata
        cur.execute(
            "INSERT INTO documents (content, embedding, model_name, model_version) "
            "VALUES (%s, %s, %s, %s)",
            (text, emb, "all-MiniLM-L6
