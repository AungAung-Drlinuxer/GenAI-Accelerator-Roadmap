# PostgreSQL + pgvector + Alembic

PostgreSQL အတွင်း vector embeddings သိမ်းဆည်းရန် pgvector extension ကို အသုံးပြုပြီး schema ဒီဇိုင်း၊ index နှင့် Alembic migrations ကို လက်တွေ့လေ့လာမည်။

## ဒီ module မှာ ဘာသင်မလဲ

- Documents နှင့် chunks အတွက် relational schema ဒီဇိုင်းရေးဆွဲနည်း
- pgvector ၏ embedding column types (`vector`, `halfvec`, `sparsevec`) များအကြောင်း
- HNSW နှင့် IVFFlat index နှစ်မျိုးကွာခြားချက်နှင့် ရွေးချယ်ပုံ
- Distance operators (`<->`, `<=>`, `<#>`) များအသုံးပြုပြီး similarity search ပြုလုပ်နည်း
- Alembic ဖြင့် pgvector schema များကို migration စီမံခန့်ခွဲနည်း
- Vector index တည်ဆောက်မှုကုန်ကျစရိတ်နှင့် build အချိန်အကြောင်း

## သင်ခန်းစာများ

1. **Lesson 3.5.1** — Document/Chunk schema ဒီဇိုင်း — metadata, content နှင့် embedding columns စီမံခန့်ခွဲနည်း
2. **Lesson 3.5.2** — Embedding column types — `vector(n)`, `halfvec(n)` နှင့် `sparsevec` တို့၏ သင့်တော်မှု
3. **Lesson 3.5.3** — HNSW vs IVFFlat — အသုံးပြုမည့် scenario အလိုက် index ရွေးချယ်နည်း
4. **Lesson 3.5.4** — Distance operators — L2, cosine နှင့် inner product ဖြင့် query ရေးနည်း
5. **Lesson 3.5.5** — Alembic migrations — pgvector column များနှင့် index များကို version control ပြုလုပ်နည်း
6. **Lesson 3.5.6** — Index build costs — index တည်ဆောက်ချိန်၊ memory နှင့် parameter tuning အကြောင်း

## လိုအပ်ချက်များ (Prerequisites)

- Week 1–2 မှ Python နှင့် embedding basics အသိပညာ
- PostgreSQL 15 နှင့်အထက် ထည့်သွင်းထားပြီး shell/command အသုံးပြုနိုင်စွမ်း
- Docker basics (Week 3 ရှေ့ပိုင်းမှ) — database container လည့်ရန်
- SQLAlchemy အခြေခံ နားလည်မှု
- `pip`, Python virtual environment တည်ဆောက်နိုင်စွမ်း

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- RAG pipeline တစ်ခုတွင် chunk embeddings များကို database ထဲ တာဝန်ယူစွာသိမ်းဆည်းလိုသည့်အခါ
- Vector search ကို application logic ကနေ ခွဲထုတ်ပြီး database layer တွင် ပြုလုပ်လိုသည့်အခါ
- Schema ပြောင်းလဲမှုများကို team လိုက် မျှဝေပြီး reproducible ဖြစ်စေလိုသည့်အခါ
- Production တွင် index build ကြာမည့်အချိန်နှင့် memory ကို ကြိုတင်ခန့်မှန်းရန်

## ကိုးကား

- pgvector — https://github.com/pgvector/pgvector
- Alembic — https://alembic.sqlalchemy.org
- PostgreSQL Docs — https://www.postgresql.org/docs/
- SQLAlchemy — https://www.sqlalchemy.org
