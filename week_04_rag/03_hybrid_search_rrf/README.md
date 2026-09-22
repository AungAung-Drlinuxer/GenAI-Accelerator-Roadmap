# Hybrid Search, RRF & Reranking

Vector search တစ်ခုတည်းထက် ပိုမြန်စွာနားလည်နိုင်စေဖို့ keyword search နဲ့ vector search ကို ပေါင်းစပ်ပြီး RRF ဖြင့် rank ပြန်စီကာ reranker နဲ့ အဆင့်မြှင့်တဲ့ hybrid search pipeline ကို တည်ဆောက်သင်ရမယ့် module ဖြစ်ပါတယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Keyword search နဲ့ vector search ရဲ့ အားသားချက်၊ နှိမ့်ချက်များနဲ့ ဘာကြောင့် နှစ်ခုလုံး လိုအပ်လဲဆိုတာ
- Reciprocal Rank Fusion (RRF) ဆိုတာဘာလဲ၊ score အစား rank ကို အခြေခံပြီး ဘယ်လို ပေါင်းလဲ
- PostgreSQL နဲ့ pgvector ပေါ်မှာ hybrid search SQL patterns ရေးနည်း
- Cross-encoder reranker သုံးပြီး retrieved candidates တွေကို ထပ်မံ အဆင့်ခွဲနည်း
- top-k parameter တွေကို ဘယ်လို tune လုပ်မလဲ
- `jsonb` metadata filters နဲ့ search results တွေကို ဘယ်လို ကျဉ်းမလဲ

## သင်ခန်းစာများ

1. **Keyword vs Vector Recall** — lexical match ရဲ့ တိကျမှုနဲ့ semantic match ရဲ့ နားလည်မှုကို နှိုင်းယှဉ်
2. **RRF အခြေခံများ** — reciprocal rank fusion formula နဲ့ သူ့ ဘာကြောင့် robust ဖြစ်လဲဆိုတာ
3. **SQL Hybrid Search Patterns** — PostgreSQL full-text search နဲ့ pgvector similarity search ကို CTE နဲ့ ပေါင်းစပ်ခြင်း
4. **Reranker Pass** — LLM ရဲ့ score ဖြင့် မဟုတ်ဘဲ reranking model သုံးပြီး precision တိုးစေခြင်း
5. **Top-k Tuning နှင့် Evaluation** — retrieval နဲ့ rerank အဆင့်တွေရဲ့ k တန်ဖိုးတွေကို ချိန်ညှိခြင်း
6. **Metadata Filters with JSONB** — document metadata တွေကို filter လုပ်ပြီး result တွေကို ကန့်သတ်ခြင်း

## လိုအပ်ချက်များ (Prerequisites)

- Week 3 ရဲ့ embeddings အခြေခံနဲ့ vector database သုံးပုံကို နားလည်ထားဖို့
- Python အခြေခံနဲ့ `psycopg2` သို့မဟုတ် `SQLAlchemy` နဲ့ PostgreSQL ချိတ်ဆက်နိုင်စွမ်း
- PostgreSQL 16 နဲ့အထက်၊ `pgvector` extension install ထားဖို့
- SQL basics — `SELECT`, `JOIN`, CTE နဲ့ `jsonb` operators အကြမ်းသဘော

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- အသုံးအနှုန်း တိကျတဲ့ product codes၊ error codes သို့မဟုတ် နာမည်တွေ ပါဝင်တဲ့ query တွေကို vector search ချော်နေတဲ့အခါ
- Retrieval quality ကို accuracy နဲ့ latency အကြား ညှိနှိုင်းရတဲ့ production RAG system တွေမှာ
- Document တွေမှာ department၊ date သို့မဟုတ် language စတဲ့ structured metadata တွေ ပါရှိပြီး access control လိုအပ်တဲ့အခါ

## ကိုးကား

- pgvector documentation — https://github.com/pgvector/pgvector
- PostgreSQL FTS (full text search) documentation — https://www.postgresql.org/docs/current/textsearch.html
- PostgreSQL JSON functions and operators — https://www.postgresql.org/docs/current/functions-json.html
- OpenAI embeddings guide — https://platform.openai.com/docs/guides/embeddings
- Cohere rerank API documentation — https://docs.cohere.com/reference/rerank
