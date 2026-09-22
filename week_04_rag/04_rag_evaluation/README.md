# RAG Evaluation & Grounding

RAG စနစ်တစ်ခုရဲ့ အဖြေမှန်ကန်မှု (grounding) နဲ့ retrieval အရည်အသွေးကို တိုင်းတာနိုင်ရန် Golden question sets, recall@k / MRR / nDCG, faithfulness checks နဲ့ CI regression gates တို့ကို လေ့လာမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- RAG စနစ်ကို စနစ်တကျ အကဲဖြတ်ဖို့ Golden question set တစ်ခု တည်ဆောက်နည်း
- Retrieval quality တိုင်းတာမှု metrics ဖြစ်တဲ့ recall@k, MRR နဲ့ nDCG တို့ရဲ့ အဓိပ္ပာယ်နဲ့ တွက်နည်း
- LLM အဖြေထဲက အချက်အလက်တွေက context နဲ့ ကိုက်ညီမှုရှိမရှိ စစ်ဆေးတဲ့ faithfulness နဲ့ citation checks
- Code ပြင်တိုင်း evaluation ပြေးတဲ့ CI regression gates ထည့်သွင်းနည်း
- Hallucination လျှော့ချရန် လက်တွေ့ကျတဲ့ နည်းလမ်းတွေ

## သင်ခန်းစာများ

1. **Golden Question Sets** — သင်ရဲ့ dataset အတွက် မေးခွန်း-အဖြေမှန် စုံတွဲများ စုဆောင်းပြီး evaluation အခြေခံပြုနည်း။
2. **Retrieval Metrics: recall@k** — Top-k retrieved documents ထဲမှာ မှန်ကန်တဲ့ document ပါဝင်မှုရှိမရှိ တိုင်းတာနည်း။
3. **Retrieval Metrics: MRR & nDCG** — Ranking quality ကို တိုင်းတာတဲ့ Mean Reciprocal Rank နဲ့ normalized Discounted Cumulative Gain တွက်နည်း။
4. **Faithfulness & Citation Checks** — ထုတ်ပေးတဲ့ အဖြေက context ထဲက အချက်အလက်တွေအပေါ် အခြေခံထားမှုရှိမရှိ အလိုအလျောက် စစ်ဆေးနည်း။
5. **Regression Gates in CI** — GitHub Actions စတဲ့ CI pipeline ထဲမှာ score threshold တွေကို gate အဖြစ် သတ်မှတ်ပြီး အရည်အသွေးကျဆင်းမှုကို အလိုအလျောက် ဖမ်းနည်း။
6. **Hallucination Mitigation** — Prompt ဒီဇိုင်း, retrieval တိုးတက်အောင်လုပ်ခြင်းနဲ့ refusal pattern တို့နဲ့ hallucination လျှော့ချနည်း။

## လိုအပ်ချက်များ (Prerequisites)

- Week 4 ရဲ့ အခြေခံ RAG pipeline (embedding, vector store, retrieval) အကြောင်း နားလည်ထားရမယ်
- Python အခြေခံနဲ့ pytest သို့မဟုတ် unit test ရေးနည်း အခြေခံကောင်းရမယ်
- Git နဲ့ CI/CD အခြေခံသဘောတရား သိထားရမယ်
- OpenAI သို့မဟုတ် တခြား LLM API key တစ်ခုခု လိုအပ်မယ်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

RAG prototype တစ်ခုကို အသုံးပြုသူထံ မတင်ခင် ဒါမှမဟုတ် production မှာ ရှိပြီးသား RAG စနစ်ကို ပြင်ဆင်တိုးတက်စေလိုတဲ့ အချိန်မှာ ဒီ module ရဲ့ နည်းလမ်းတွေက မရှိမဖြစ် လိုအပ်ပါတယ်။ Chunking strategy ပြောင်းတဲ့အခါ, model ပြောင်းတဲ့အခါ ဒါမှမဟုတ် prompt ပြင်တဲ့အခါ — ဘယ်အရာမဆို ပြောင်းလိုက်ရင် အရည်အသွေး ထပ်ပြောင်းသွားမှန်း အာမခံချက်မရှိပါဘူး။ Evaluation metrics တွေနဲ့ CI gates တွေက ဒီပြောင်းလွဲမှုတွေကို ချက်ချင်း ဖမ်းဆီးပေးပြီး ယုံကြည်စိတ်ချစွာ iterate လုပ်နိုင်စေပါတယ်။

## ကိုးကား

- Langfuse RAG Evaluation Guide — https://langfuse.com/docs
- Langfuse LLM Evaluation Docs — https://langfuse.com/docs/scores/overview
- Langfuse Python SDK — https://python.reference.langfuse.com
- scikit-learn: Metrics for Ranking Evaluation — https://scikit-learn.org/stable/modules/model_evaluation.html
- pytest Documentation — https://docs.pytest.org/en/stable/
- GitHub Actions Documentation — https://docs.github.com/en/actions
