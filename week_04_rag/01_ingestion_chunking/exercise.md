## လေ့ကျင့်ခန်း ၁ — PDF မှ စာသား extract လုပ်ခြင်း

`pypdf` library ကို အသုံးပြုပြီး PDF file တစ်ခုမှ စာသားများကို page အလိုက် extract လုပ်ပါ။ ထွက်ရှိလာသော စာသားတစ်ခုစီကို `{"page": 1, "text": "..."}` ပုံစံ dictionary ဖြင့် list တစ်ခုအဖြစ် return ပေးသော function ရေးပါ။

```python
from pypdf import PdfReader

def extract_pdf_pages(pdf_path):
    reader = PdfReader(pdf_path)
    pages = []
    for i, page in enumerate(reader.pages, start=1):
        # TODO: extract text and store page number as metadata
        pass
    return pages
```

**Hints:** `page.extract_text()` ကိုသုံးပါ။ စာသားမရှိသော page များကို `if text:` ဖြင့် စစ်ပြီး ချန်လှပ်ပါ။

**Expected behavior:** function ကိုခေါ်လိုက်ရာတွင် PDF ရှိ page အရေအတွက်နှင့် ကိုက်ညီသော list တစ်ခုရပြီး တစ်ခုစီတွင် page number ပါဝင်သည်။

## လေ့ကျင့်ခန်း ၂ — Fixed-size chunking with overlap

စာသားရှည်တစ်ခုကို သတ်မှတ်အရွယ်အစား (ဥပမာ `chunk_size=500` characters) ဖြင့် အပိုင်းများခွဲပါ။ `chunk_overlap=50` ဖြင့် အပိုင်းတစ်ခုနှင့် တစ်ခု စာလုံး ၅၀ ထပ်နေစေရမည်။

```python
def fixed_chunk(text, chunk_size=500, chunk_overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        # TODO: slice text from start to start + chunk_size
        # TODO: advance start by chunk_size - chunk_overlap
        pass
    return chunks
```

**Hints:** `step = chunk_size - chunk_overlap` ကို သုံးပြီး `start += step` လုပ်ပါ။ `step` သည် 0 သို့မဟုတ် negative မဖြစ်စေရန် စစ်ဆေးပါ။

**Expected behavior:** character ရှည် 1200 ခန့် စာသားတစ်ခုအတွက် အပိုင်းများ ရရှိပြီး အပိုင်းဆက်များတွင် စာလုံး ၅၀ ခန့် ထပ်နေသည်။

## လေ့ကျင့်ခန်း ၃ — Metadata design

Extract လုပ်ထားသော page dictionary တစ်ခုချင်းစီအတွက် ရှာဖွေမှု ပိုမိုတိကျစေရန် metadata ပေါင်းထည့်ပါ — `source` (file name), `title`, `section`, `created_at`။ ထို့နောက် metadata အပေါ် အခြေခံ၍ filter လုပ်နိုင်သော function တစ်ခုရေးပါ။

```python
import os
from datetime import datetime

def add_metadata(pages, pdf_path):
    # TODO: add source, created_at to each page dict
    pass

def filter_by_source(pages, source_name):
    # TODO: return only pages whose source matches
    pass
```

**Hints:** `os.path.basename(pdf_path)` ဖြင့် file name ရယူပါ။ `datetime.now().isoformat()` ဖြင့် အချိန်မှတ်ပါ။

**Expected behavior:** `filter_by_source` ကို ခေါ်လိုက်သောအခါ ထို source မှ pages များသာ ပြန်ရရှိသည်။

## လေ့ကျင့်ခန်း ၄ — Idempotent re-ingestion

ထပ်မံ ingest လုပ်သောအခါ စာသားတူပါးဒေတာများ မပေါ်ပေါက်စေရန် chunk ID တစ်ခု တည်းရှိစေသော deterministic ID ဖန်တီးမှု ရေးပါ။ ID ကို source + page + chunk index တို့မှ hash လုပ်ပြီး ဖန်တီးပါ။

```python
import hashlib

def make_chunk_id(source, page, chunk_index, chunk_text):
    # TODO: build a stable string and hash it
    pass
```

**Hints:** `hashlib.sha256(...).hexdigest()[:16]` ကို သုံးပါ။ ID တွင် အချိန်ကာလ (timestamp) မပါဝင်စေရ — idempotency ပျက်သွားမည်။

**Expected behavior:** source, page, chunk index, text တူညီလျှင် function ကို အကြိမ်များစွာခေါ်သော်လည်း ID တူညီသည်။ Timestamp ပါသော ID ကို စမ်းကြည့်ပြီး ကွာခြားချက် နှိုင်းယှဉ်ပါ။

## လေ့ကျင့်ခန်း ၅ — Semantic vs fixed chunking နှိုင်းယှဉ်ခြင်း

`nltk` (သို့) ရိုးရိုး sentence splitting ဖြင့် ဝါကျ ခွဲခြားမှုများအတိုင်း ခွဲသော semantic chunking function တစ်ခုရေးပြီး fixed chunking (လေ့ကျင့်ခန်း ၂) နှင့် နှိုင်းယှဉ်ပါ။ ဝါကျဖြတ်ပါးမှု အရေအတွက်၊ ပျမ်းမျှ chunk အရွယ်အစားနှင့် ဝါကျတစ်ခု နှစ်ပိုင်းကျနေမှု ရှိမရှိကို တွက်ပြပါ။

```python
import re

def semantic_chunk(text, max_chars=800):
    # TODO: split into sentences, merge until max_chars
    pass

def compare(text, size=500, overlap=50):
    # TODO: call both chunkers and print statistics
    pass
```

**Hints:** `re.split(r'(?<။)', text)` ကဲ့သို့ ဗမာစာ punctuation သင်္ကျာ်အတွက် နားလည်သင့်သည်။ Sentences များကို `max_chars` မကျော်မီအထိ ပေါင်းစည်းပါ။

**Expected behavior:** နှစ်မျိုးစလုံး chunk များထုတ်ပေးပြီး ဗမာစာ နမူနာတစ်ခုအတွက် ဝါကျဖြတ်ပါးခံရမှု အရေအတွက် ကွာခြားချက်ကို console တွင် မြင်ရသည်။

## လေ့ကျင့်ခန်း ၆ — Chunk size ၏ recall နှင့် cost အပေါ် သက်ရောက်မှု

Chunk size သုံးမျိုး (ဥပမာ 250, 500, 1000) ဖြင့် တစ်ခုတည်းသော document ကို ingest လုပ်ပြီး keyword-based ရှာဖွေမှု (simple substring or TF scoring) ဖြင့် မေးခွန်း ၃-၅ ခု စမ်းသပ်ပါ။ Chunk အရေအတွက်၊ စမ်းသပ်ချက်တိုင်းရှ ရှာတွေ့မှု (hit or miss) နှင့် ပျမ်းမျှ chunk အရှည် တို့ကို ဇယားဖြင့် ပြပါ။

```python
def ingest_at_sizes(text, sizes=(250, 500, 1000)):
    # TODO: chunk text at each size, store results
    pass

def search(chunks, query):
    # TODO: return True if query appears in any chunk
    pass
```

**Hints:** မေးခွန်းစာလုံးများက document တွင် ပါဝင်ကြောင်း သေချာပါ။ Chunk သေးလွန်းလျှင် စာလုံးများ ဖြတ်ပါးခံရပြီး miss ဖြစ်နိုင်သည်။ မည်သည့် benchmark နံပါတ်မှ မကြေညာပါနှင့် — သင်၏ document အပေါ်သာ အခြေခံပါ။

**Expected behavior:** ဇယားတွင် chunk size တစ်ခုချင်းစီအတွက် chunk အရေအတွက် နှင့် မေးခွန်းအောင်မြင်မှု အရေအတွက် မြင်ရပြီး သေးသော chunk size များတွင် chunk အရေအတွက် ပိုများသော်လည်း အချို့မေးခွန်းများ miss ဖြစ်နိုင်ကြောင်း လေ့လာနိုင်သည်။
