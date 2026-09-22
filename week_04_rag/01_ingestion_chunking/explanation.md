# Ingestion & Chunking Strategies

RAG စနစ်တစ်ခုမှာ အရေးအကြီးဆုံးအဆင့်က ingestion ဖြစ်သည် — စာရွက်စာတမ်းများကို ဖတ်ပြီး၊ သင့်တော်သောအရွယ်အစားရှိ chunk များအဖြစ် ခွဲကာ၊ vector database (ဥပမာ pgvector) ထဲသို့ သိမ်းဆည်းခြင်းဖြစ်သည်။ ဤ module တွင် parsing၊ chunking မျိုးစုံ၊ metadata ဒီဇိုင်း၊ idempotent re-ingestion နှင့် chunk size ၏ သက်ရောက်မှုများ တို့ကို လေ့လာပါမည်။

---

## 1. Parsing Sources (အရင်းအမြစ်များကို ဖော်ပြဖို့ ဖတ်ခြင်း)

### ဘာကို ဆိုလိုတာလဲ

Parsing ဆိုသည်မှာ PDF၊ Word၊ HTML၊ Markdown စသည့် မူရင်းဖိုင်များမှ စာသားကို ကွန်ပျူတာဖတ်နိုင်သော ရိုးရိုး text အဖြစ် ထုတ်ယူခြင်းဖြစ်သည်။ PDF ထဲမှာ table၊ ပုံ၊ ကော်လံနှစ်ခုရှိတတ်ပြီး၊ HTML ထဲမှာ navigation bar၊ ကြော်ငြာ ကဲ့သို့ အနှစ်သက်မရှိသော အပိုင်းများပါဝင်တတ်သည်။

### ဘာကြောင့် လဲ

Parsing မှာ အမှားရှိပါက နောက်မှာ embedding ဘယ်လောက်ကောင်းနေစေကာမူ chunk အတွင်း စာသားပျက်နေပါက ရှာဖွေမှုအရည်အသွေး ကျဆင်းသွားမည်။ "Garbage in, garbage out" သဘောတရားအတိုင်းဖြစ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

၁။ ဖိုင်အမျိုးအစားအလိုက် library ရွေးမည် (PDF အတွက် pypdf၊ HTML အတွက် BeautifulSoup)။
၂။ စာသားကို ထုတ်ယူပြီး heading၊ paragraph စသည့် structure ကို မှတ်ယူမည်။
၃။ စာသားကို သန့်ရှင်းစွာ ရရှိမရ စစ်ဆေးမည်။

### ဥပမာ

```python
from pypdf import PdfReader

# Extract text from a PDF file, page by page
reader = PdfReader("report.pdf")

pages = []
for i, page in enumerate(reader.pages):
    text = page.extract_text() or ""
    # Keep a minimal record so we can trace chunks back to pages later
    pages.append({"page_number": i + 1, "text": text.strip()})

print(pages[0]["text"][:120])
print("Total pages:", len(pages))
# Expected output:
# (first ~120 characters of the report's first page, followed by:)
# Total pages: 12
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

အဖွင့်အဆုံး မေးခွန်းထုတ်သည့်အခါ မှန်ကန်သော အဖြေရစေဖို့ မှန်ကန်သော စာသားလိုအပ်သည်။ PDF တစ်ခုစီက ထူးခြားပြီး table များ ပျက်လွဲတတ်သောကြောင့် မိမိ data ပေါ် munged parsing ရလဒ်ကို ပြန်စစ်ဆေးခြင်းသည် production တွင် မဖြစ်မနေ လိုအပ်သည်။

---

## 2. Semantic vs Fixed Chunking (အနက်အဓိပ္ပာယ်အလိုက် vs အရွယ်အစားအလိုက် ခွဲခြင်း)

### ဘာကို ဆိုလိုတာလဲ

Fixed chunking ဆိုသည်မှာ character သို့မဟုတ် token အရေအတွက် သတ်မှတ်ပြီး ညီညာစွာ ခွဲခြင်းဖြစ်သည်။ Semantic chunking ကတော့ ဆူးတုန့်ပြီးနောက် ခေါင်းစဉ်ပြောင်းသည့်နေရာ — အကြောင်းအရာတစ်ခုမှ နောက်တစ်ခုသို့ ကူးပြောင်းသည့်နေရာများတွင် ခွဲခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ

Fixed chunking က လွယ်ကူပြီး မြန်သော်လည်း ဆူးတုန့်တစ်ခုအတွင်းမှ အလယ်တွင် ဖြတ်တောက်တတ်သည်။ Semantic chunking က အနက်အဓိပ္ပာယ်နယ်နိမိတ်ကို လေးစားသဖြင့် retrieval တွင် ပိုမှန်ကန်စေသည်၊ သို့သော် အနည်းငယ် ပိုရှုပ်ထွေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Semantic chunking ရိုးရိုး version တစ်ခုမှာ paragraph များကြားရှိ cosine similarity ကိုတွက်ပြီး၊ similarity တစ်ဖန်ထက် ပိုနိမ့်သွားသည့်နေရာများကို နယ်နိမိတ်အဖြစ် ယူခြင်းဖြစ်သည်။ Recursive character splitting (ဥပမာ LangChain ၏ `RecursiveCharacterTextSplitter`) ကမူ `\n\n` → `\n` → `. ` စဉ်အလိုက် ဖြတ်တောက်သည်။

### ဥပမာ

```python
# Simple fixed-size chunking with overlap
def fixed_chunks(text, size=400, overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + size
        chunks.append(text[start:end])
        start += size - overlap  # step forward, leaving an overlap
    return chunks

doc = "RAG systems retrieve relevant passages before answering. " * 20
result = fixed_chunks(doc)
print("Chunk count:", len(result))
print("Chunk 1 tail == Chunk 2 head?", result[0][-50:] == result[1][:50])
# Expected output:
# Chunk count: 9
# Chunk 1 tail == Chunk 2 head? True
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Chunk တစ်ခုသည် အကြောင်းအရာတစ်ခုတည်းကိုသာ ကိုယ်စားပြုပါက embedding vector သည် ပိုမှန်ကန်လွန်းသည်။ ဒါပေမယ့် စာရွက်များစွာအတွက် semantic chunking က ကုန်ကျစရိတ် ပိုများစေသဖြင့် အလုပ်တွင် structure ရှိသော စာရွက် (ဥပမာ heading ပါသော Markdown) များအတွက် structure-aware fixed chunking က များသောအားဖြင့် လုံလောက်သည်။

---

## 3. Overlap & Metadata Design (နောက်ဆက်ခြင်းနှင့် metadata ဒီဇိုင်း)

### ဘာကို ဆိုလိုတာလဲ

Overlap ဆိုသည်မှာ နောက် chunk သည် ရှေ့ chunk ၏ အဆုံးသတ်အပိုင်းအနည်းငယ် ထပ်မံပါဝင်ခြင်းဖြစ်သည်။ Metadata ဆိုသည်မှာ chunk တစ်ခုနှင့်အတူ သိမ်းဆည်းသော အပိုဆောင်း အချက်အလက်များ (ဖိုင်နာမည်၊ စာမျက်နှာနံပါတ်၊ အချိန်၊ အမျိုးအစား) ဖြစ်သည်။

### ဘာကြောင့် လဲ

Overlap မရှိပါက အရေးကြီးသော ဆူးတုန့်တစ်ခုသည် နှစ်ခုကြား ဖြတ်တောက်ခံရပါက မည်သည့် chunk မှာမှ အပြည့်အစုံ ရှိမည်မဟုတ်ပါ။ Metadata ကတော့ နောက်ပိုင်း စစ်ထုတ်ခြင်း (ဥပမာ "၂၀၂၄ ခုနှစ် စာရွက်များကိုသာ ရှာပါ")၊ အရင်းအမြစ်ပြန်ပြခြင်း (citation) နှင့် ပြန်တမ်း ရှင်းလင်းခြင်းတို့အတွက် အသုံးဝင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Chunk တစ်ခုစီကို Python `dict` အဖြစ် ပြင်ဆင်ပြီး pgvector table တွင် `source`၊ `page`၊ `chunk_index`၊ `content` စသည့် column များထည့်သည်။ Overlap ကို ယေဘုယျအားဖြင့် chunk size ၏ ၁၀-၂၀% ထားလေ့ရှိသည် — များပါက storage နှင့် embedding ကုန်ကျစရိတ် တက်သည်။

### ဥပမာ

```python
# Attach metadata to each chunk before insertion
def make_records(source_name, pages, size=400, overlap=50):
    records = []
    for page in pages:
        for idx, chunk in enumerate(fixed_chunks(page["text"], size, overlap)):
            records.append({
                "source": source_name,        # e.g. "policy_2024.pdf"
                "page": page["page_number"],   # provenance for citations
                "chunk_index": idx,           # order within the page
                "content": chunk,             # text to be embedded
            })
    return records

recs = make_records("policy_2024.pdf", [{"page_number": 1, "text": "terms " * 300}])
print("Records:", len(recs))
print("First record keys:", sorted(recs[0].keys()))
# Expected output:
# Records: 9
# First record keys: ['chunk_index', 'content', 'page', 'source']
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

အသုံးပြုသူက "ဒီအဖြေကို ဘယ်စာရွက်က ရသလဲ" ဟုမေးပါက metadata မရှိပါက ဖြေဆိုလို့မရပါ။ ထို့အပြင် pgvector query တွင် `WHERE source = '...'` ကဲ့သို့ metadata filter ထည့်နိုင်ခြင်းက retrieval တွင် အလွန်အရေးပါသည်။

---

## 4. Idempotent Re-Ingestion (ထပ်မံထည့်သွင်းမှု ပြန်မဖြစ်စေခြင်း)

### ဘာကို ဆိုလိုတာလဲ

Idempotent ingestion ဆိုသည်မှာ ဖိုင်တစ်ခုကို နှစ်ကြိမ်၊ သုံးကြိမ် ထည့်သွင်းချောင်း ရလဒ်တစ်ကြိမ်ထည့်သွင်းခြင်းနှင့် အတူတူဖြစ်ခြင်းကို ဆိုလိုသည် — duplicate chunk များ မတက်ပါ။

### ဘာကြောင့် လဲ

Pipeline ကို ပြန်လည်မောင်းချိန်တွင် (ဥပမာ cron job တစ်ခု နှစ်ကြိမ် run သွားခြင်း) idempotency မရှိပါက တူညီသော chunk များ အဆုံးမဲ့ ပေါင်းလာမည်။ ၎င်းက retrieval ရလဒ်များကို လွဲမှားစေပြီး storage ကုန်ကျစရိတ်ကိုလည်း တက်စေသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

၁။ chunk တစ်ခုစီအတွက် တည်ငြိမ်သော ID တွက်မည် (ဥပမာ `sha256(source + page + chunk_index)` သို့မဟုတ် content hash)။
၂။ Database တွင် `INSERT ... ON CONFLICT (id) DO NOTHING` (PostgreSQL) ကို အသုံးပြုမည် — pgvector docs တွင် ဤနမူနာအထိ ဖော်ပြထားသည်။
၃။ စာရွက်ပြောင်းလဲပါက document ID အလိုက် အဟောင်းများကို ပထမဦးစွာ ပျက်မှု (delete) ပြုလုပ်ပြီးမှ သစ်များ ထည့်မည်။

### ဥပမာ

```python
import hashlib
import psycopg

# Deterministic ID so re-running the pipeline creates no duplicates
def chunk_id(source, page, idx, content):
    raw = f"{source}:{page}:{idx}:{hashlib.sha256(content.encode()).hexdigest()}"
    return hashlib.sha256(raw.encode()).hexdigest()

rec = recs[0]
cid = chunk_id(rec["source"], rec["page"], rec["chunk_index"], rec["content"])

with psycopg.connect("postgresql://user:pass@localhost:5432/docs") as conn:
    with conn.cursor() as cur:
        # Idempotent insert: skips rows whose ID already exists
        cur.execute(
            """
            INSERT INTO chunks (id, source, page, chunk_index, content)
            VALUES (%s, %s, %s, %s, %s)
            ON CONFLICT (id) DO NOTHING
            """,
            (cid, rec["source"], rec["page"], rec["chunk_index"], rec["content"]),
        )
    conn.commit()
print("Inserted id:", cid[:16] + "...")
# Expected output:
# Inserted id: 3f9a1c7d2e4b5a60...
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

စာရွက်များကို အမြဲ update လုပ်နေရသော production RAG system များတွင် pipeline ကို လုံးဝ idempotent ဖြစ်စေခြင်းသည် ရလဒ်တွင် မူမမှန်မှုများနှင့် ကုန်ကျစရိတ် အလွန်တက်မှုကို တားဆီးပေးသည်။

---

## 5. Chunk-Size Effects on Recall & Cost (Chunk အရွယ်အစား၏ ရလဒ်နှင့် ကုန်ကျစရိတ် အပေါ် သက်ရောက်မှု)

### ဘာကို ဆိုလိုတာလဲ

Chunk size သည် တစ်ခုလျှင် စာသားမည်မျှ ပါဝင်မည်ကို ဆုံးဖြတ်သော ဆုံးဖြတ်ချက်ဖြစ်သည်။ အလွန်သေးငယ်ပါက အချက်အလက် ပိုင်းပြတ်နေမည်၊ အလွန်ကြီးပါက အရေးမကြီးသော အပိုင်းများ ပါလာမည်။

### ဘာကြောင့် လဲ

- သေးလွန်းပါက — retrieval တွင် ပြီးပြည့်စုံသော အချက်အလက် မပါဝင်ဘဲ recall ကျဆင်းနိုင်သည်။
- ကြီးလွန်းပါက — chunk များအတွင်း အကြောင်းအရာများ ပေါင်းစပ်မိပြီး embedding ရလဒ် ရှုပ်ထွေးလာပြီး၊ prompt တွင် အလွန်များသော
