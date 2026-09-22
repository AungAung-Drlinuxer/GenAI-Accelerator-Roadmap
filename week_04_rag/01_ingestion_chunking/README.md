# Ingestion & Chunking Strategies

RAG system တစ်ခုမှာ source data များကို parsing လုပ်ခြင်း၊ chunking strategy ရွေးချင်ခြင်း၊ metadata ဒီဇိုင်းဆွဲခြင်းနှင့် re-ingestion ကို idempotent ဖြစ်စေခြင်းတို့အကြောင်း သင်ကြားပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- PDF, HTML, Markdown စတဲ့ source file များကို parsing လုပ်ပြီး text ထုတ်ယူနည်း
- Fixed-size chunking နှင့် semantic chunking ကွာခြားချက်နှင့် ရွေးချယ်ဆုံးဖြတ်နည်း
- Chunk overlap သုံးခြင်းအားဖြင့် context ဆုံးရှုံးမှု လျှော့ချနည်း
- Chunk တစ်ခုချင်းစီအတွက် metadata schema ဒီဇိုင်းဆွဲနည်း
- Document ပြန် upload လုပ်တဲ့အခါ duplicate data မဖြစ်စေဘဲ idempotent re-ingestion လုပ်နည်း
- Chunk size ကြီးရင် သေးရင် recall နှင့် cost ပေါ်ဘယ်လိုသက်ရောက်မှုရှိမလဲ

## သင်ခန်းစာများ

1. **Source Parsing** — LangChain document loader တွေနဲ့ file format အမျိုးမျိုးကနေ text ထုတ်ယူခြင်း
2. **Fixed Chunking** — အရွယ်အစားသတ်မှတ်ထားတဲ့ chunk တွေအဖြစ် ခွဲခြမ်းခြင်း (RecursiveCharacterTextSplitter)
3. **Semantic Chunking** — အဓိပ္ပာယ်အရ ကွဲပြားတဲ့အပိုင်းတွေကို အခြေခံပြီး chunk ခွဲခြင်း
4. **Overlap & Context** — Chunk တွေကြားမှာ overlap ထည့်သွင်းပြီး ဆက်စပ်အချက်အလက် မပျောက်စေနည်း
5. **Metadata Design** — source, page, section, timestamp စတဲ့ metadata တွေကို schema တကျ သတ်မှတ်ခြင်း
6. **Idempotent Re-ingestion** — Content hash နှင့် stable ID သုံးပြီး ထပ်ခါထပ်ခါ ingest လုပ်တာကို ဘေးကင်းအောင် စီမံခြင်း
7. **Chunk Size Trade-offs** — Chunk size နဲ့ recall, embedding cost, retrieval cost ကြားရှိ ချိန်ညှိမှုတွေ

## လိုအပ်ချက်များ (Prerequisites)

- Python basics (function, dictionary, list) နှင့် pip နဲ့ package install လုပ်နိုင်စွမ်း
- Week 3 မှ vector embedding အယူအဆ (OpenAI embeddings) ကို နားလည်ထားခြင်း
- PostgreSQL အခြေခံ နှင့် pgvector extension ထည့်သွင်းထားခြင်း
- LangChain Python SDK ကို သိကျွမ်းထားခြင်း (သိမှုမရှိပါက install လုပ်နည်းကို lesson 1 မှာပြထားပါမယ်)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Company document, manual, FAQ စတဲ့ file တွေကနေ RAG pipeline တစ်ခု စတင်တည်ဆောက်ချင်တဲ့အခါ
- Ingest လုပ်ထားတဲ့ data တွေက ဟာသေးလို့ သို့မဟုတ် ဟာကြီးလို့ search result တွေ မှန်ကန်မှုနည်းနေတဲ့အခါ chunking strategy ပြန်တည်ဖို့
- Document တွေ update ဖြစ်ချင်မိချင်နေပြီး database ထဲမှာ duplicate chunk တွေ စုနေတဲ့အခါ
- Embedding နှင့် storage cost တွေကို ထိန်းသိမ်းချင်ပြီး retrieval quality ကိုပါ ထိန်းထားချင်တဲ့အခါ

## ကိုးကား

- pgvector official documentation — https://github.com/pgvector/pgvector
- LangChain Text Splitters docs — https://python.langchain.com/docs/concepts/text_splitters/
- LangChain Document Loaders docs — https://python.langchain.com/docs/concepts/document_loaders/
- OpenAI Embeddings docs — https://platform.openai.com/docs/guides/embeddings
- Datalumina RAG build (public roadmap reference) — https://learn.datalumina.com
