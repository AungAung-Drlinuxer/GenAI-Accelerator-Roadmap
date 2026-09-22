## လေ့ကျင့်ခန်း ၁ — Sentence Transformer ဖြင့် Embedding ထုတ်ခြင်း

Hugging Face ၏ `sentence-transformers` library ကို အသုံးပြု၍ စာသားနှစ်ကြောင်းအတွက် embedding vector ထုတ်ပြီး vector ၏ dimension ကို စစ်ကြည့်ပါ။

```python
from sentence_transformers import SentenceTransformer

# Load a small multilingual embedding model
model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

sentences = [
    "Database stores vectors efficiently.",
    "Yangon is the largest city in Myanmar.",
]

# Generate embeddings
embeddings = model.encode(sentences)
print(embeddings.shape)
print(embeddings[0][:5])
```

**Hints:** `model.encode()` သည် NumPy array တစ်ခု return ပြန်သည်။ `.shape` ဖြင့် `(number_of_sentences, dimension)` ကို ကြည့်နိုင်သည်။

**Expected behavior:** Output shape သည် `(2, 384)` ကဲ့သို့ ဖြစ်ပြီး ကိန်းများသည် `-1` နှင့် `1` ကြားရှိ float များဖြစ်သည်။

## လေ့ကျင့်ခန်း ၂ — Cosine, L2, Inner Product နှိုင်းယှဉ်ခြင်း

စာသားသုံးကြောင်းကို embedding ပြောင်းပြီး cosine similarity, Euclidean distance (L2), inner product သုံးမျိုးဖြင့် အနီးစပ်ဆုံးစာသားကို ရှာကြည့်ပါ။

```python
import numpy as np
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

query = "How do I store vectors in a database?"
docs = [
    "Pgvector stores embeddings in PostgreSQL.",
    "The weather in Yangon is warm today.",
    "Vector databases support similarity search.",
]

q = model.encode(query)
d = model.encode(docs)

# Cosine similarity
cos = [np.dot(q, x) / (np.linalg.norm(q) * np.linalg.norm(x)) for x in d]
print("Cosine ranking:", np.argsort(cos)[::-1])
```

**Hints:** L2 အတွက် `np.linalg.norm(q - x)` ကိုသုံးပါ။ Inner product အတွက် `np.dot(q, x)` ကိုသုံးပါ။ ranking အားလုံးတူညီခြင်းရှိမရှိ နှိုင်းယှဉ်ကြည့်ပါ။

**Expected behavior:** Cosine နှင့် L2 သည် ဆင်တူသော ranking ပေးပြီး စာသားအလယ်ကြောင်း (weather) သည် အနိမ့်ဆုံး score ရသည်။

## လေ့ကျင့်ခန်း ၃ — Normalization လုပ်ခြင်းနှင့် သက်ရောက်မှု

Embedding vector များကို unit length ဖြစ်အောင် normalize လုပ်ပြီး normalization ပြုလုပ်ပြီးနောက် inner product နှင့် cosine similarity တူညီကြောင်း သက်သေပြပါ။

```python
import numpy as np

v = np.array([3.0, 4.0, 0.0])
w = np.array([1.0, 0.0, 2.0])

# Normalize each vector to unit length
v_norm = v / np.linalg.norm(v)
w_norm = w / np.linalg.norm(w)

print("Inner product before normalization:", np.dot(v, w))
print("Inner product after normalization:", np.dot(v_norm, w_norm))
```

**Hints:** L2 norm သည် `sqrt(3^2 + 4^2) = 5` ဖြစ်သည်။ Normalize ပြီးနောက် inner product သည် cosine similarity ဖြစ်သွားသည်။

**Expected behavior:** Normalization ပြုလုပ်ပြီးနောက် inner product တန်ဖိုးသည် cosine similarity တန်ဖိုးနှင့် တစ်ပြားချင်း တူညီသည်။

## လေ့ကျင့်ခန်း ၄ — Batch Embedding ဖြင့် အရေအတွက်များစွာ ထုတ်ခြင်း

စာသား ၂၀ ကို batch ခွဲ၍ embedding ထုတ်ပြီး တစ်ခုချင်းထုတ်ခြင်းနှင့် batch ထုတ်ခြင်း၏ vector တန်ဖိုးများ တူညီကြောင်း စစ်ဆေးပါ။

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")
texts = [f"Document number {i} about retrieval." for i in range(20)]

# Embed all texts at once with a batch size
batch_embeddings = model.encode(texts, batch_size=8)

# Embed one text at a time
single = np.stack([model.encode(t) for t in texts])
```

**Hints:** `np.stack()` ဖြင့် list of arrays ကို 2D array ဖြစ်စေပါ။ `np.allclose(batch_embeddings, single)` ဖြင့် နှိုင်းယှဉ်ပါ။

**Expected behavior:** Batch ခွဲထုတ်သည့်ရလဒ်နှင့် တစ်ခုချင်းထုတ်သည့်ရလဒ်သည် tolerance အတွင်း တူညီသည်။

## လေ့ကျင့်ခန်း ၅ — pgvector ဖြင့် Table ဖန်တီးပြီး Vector သိမ်းခြင်း

PostgreSQL တွင် pgvector extension ကို enable လုပ်ပြီး `vector(384)` column ပါသော table ဆောက်ကာ embedding သိမ်းပါ။

```sql
-- Enable the pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create a table for documents and their embeddings
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(384),
    model_name TEXT NOT NULL
);

-- Insert a row with an embedding (replace with real values)
INSERT INTO documents (content, embedding, model_name)
VALUES ('Pgvector stores vectors in PostgreSQL.',
        '[0.1, 0.2, 0.3, ...]'::vector,
        'paraphrase-multilingual-MiniLM-L12-v2');
```

**Hints:** `vector(384)` သည် dimension ၃၈၄ ဖြစ်ကို သတ်မှတ်သည်။ Python မှ insert လုပ်ရန် `psycopg2` သို့မဟုတ် `pgvector` Python package ကို သုံးနိုင်သည်။

**Expected behavior:** Table ဖန်တီးမှုနှင့် insert လုပ်ခြင်းအောင်မြင်ပြီး `SELECT content FROM documents;` ဖြင့် စာသားကို ပြန်ရနိုင်သည်။

## လေ့ကျင့်ခန်း ၆ — Model Versioning နှင့် Vector ပြန်လည်စိစစ်ခြင်း

Model တစ်ခုချင်းစီ၏ embedding ကို `model_name` column ဖြင့် version သတ်မှတ်ပြီး မတူညီသော model များ၏ vector များကို ရောနှောမမိစေရန် query တွင် filter လုပ်ပါ။

```sql
-- Find nearest neighbors only among vectors from a specific model version
SELECT content
FROM documents
WHERE model_name = 'paraphrase-multilingual-MiniLM-L12-v2'
ORDER BY embedding <-> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

```python
# When switching models, re-embed and store under a new version tag
new_model = "all-MiniLM-L6-v2"
# old_version = "paraphrase-multilingual-MiniLM-L12-v2"
```

**Hints:** `<->` operator သည် pgvector ၏ L2 distance operator ဖြစ်သည်။ Cosine အတွက် `<=>` ကို သုံးပါ။ Model ပြောင်းလျှင် vector dimension ပြောင်းနိုင်သဖြင့် `model_name` ကို မဖြစ်မနေ မှတ်သားရမည်။

**Expected behavior:** Query သည် version တစ်ခုတည်းသော vector များထဲမှသာ အနီးစပ်ဆုံးရလဒ်များကို ပြနိုင်ပြီး dimension မတူညီမှုကြောင့်ဖြစ်သော error များကို ရှောင်နိုင်သည်။
