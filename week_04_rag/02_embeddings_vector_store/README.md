# Embeddings & Vector Store

RAG pipeline ရဲ့အခြေခံဖြစ်တဲ့ embedding model ရွေးချယ်မှု၊ vector similarity metric များ၊ batch embedding နဲ့ pgvector သုံး vector သိမ်းဆည်းနည်းကို ဒီ module မှာ လေ့လာပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Embedding model တွေရဲ့ output dimension နဲ့ quality/size tradeoff အခြေခံများ
- Cosine similarity, L2 distance, inner product ကွာခြားချက်နဲ့ ဘယ်အချိန်မှာ ဘယ်ဟာကို သုံးသင့်တယ်ဆိုတာ
- Vector normalization လုပ်ရခြင်းအကြောင်းရင်းနဲ့ သက်ရောက်မှု
- API rate limit ကို သတိထားပြီး batch embedding လုပ်နည်း
- pgvector extension နဲ့ PostgreSQL ထဲမှာ vector သိမ်းခြင်း၊ schema versioning လုပ်နည်း

## သင်ခန်းစာများ

1. **Embedding Model နဲ့ Dimensionality** — sentence embedding model တွေရဲ့ dimension အရွယ်အစားနဲ့ storage cost ဆက်စပ်မှု
2. **Similarity Metric ရွေးချယ်ခြင်း** — cosine, L2, inner product တို့ရဲ့ သင်္ချာအခြေခံနဲ့ အသုံးချမှုအခြေအနေ
3. **Normalization လုပ်ခြင်း** — L2 normalize လုပ်တဲ့အခါ cosine နဲ့ inner product ညီမှုရှိပုံ
4. **Batch Embedding** — document အများအကြီးကို တစ်ခါတည်း embed လုပ်တဲ့ pattern နဲ့ error handling
5. **Vector သိမ်းခြင်းနဲ့ Versioning** — pgvector column type, index နဲ့ model version ကို tracking လုပ်နည်း

## လိုအပ်ချက်များ (Prerequisites)

- Week 3 (Document Loading & Chunking) အပြီးသရောက်ရှိနေရန်
- Python အခြေခံနဲ့ list/dict data structure နားလည်ရန်
- PostgreSQL install လုပ်ထားပြီး SQL basic query တွေရေးနိုင်ရန်
- pip နဲ့ package install လုပ်နိုင်ရန်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Chunk လုပ်ထားတဲ့ document တွေကို numeric vector အဖြစ်ပြောင်းပြီး database ထဲမှာ ရှာဖွေနိုင်စေချင်တဲ့အခါ
- RAG system တည်ဆောက်စဉ်မှာ user query နဲ့ အနီးစပ်ဆုံး chunk တွေကို match လုပ်ချင်တဲ့အခါ
- Embedding model ပြောင်းတဲ့အခါ သိမ်းထားတဲ့ vector တွေကို ဘယ်လို handle လုပ်ရမလဲ ဆုံးဖြတ်ရတဲ့အခါ

## ကိုးကား

- pgvector README & documentation — https://github.com/pgvector/pgvector
- Hugging Face — Text Embeddings (Inference) documentation — https://huggingface.co/docs/text-embeddings-inference
- Hugging Face Sentence Transformers documentation — https://www.sbert.net/docs/quickstart.html
- Hugging Face Models — embedding models overview — https://huggingface.co/models?pipeline_tag=sentence-embeddings
