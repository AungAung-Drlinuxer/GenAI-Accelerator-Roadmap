# Tracing with Langfuse

ဒီ module မှာ LLM application တွေရဲ့ အလုပ်လုပ်ပုံကို Langfuse tracing နဲ့ မြင်သာအောင် ဖမ်းယူ၊ စစ်ဆေး၊ debug လုပ်နည်းတွေကို လေ့လာပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Trace, span, generation ဆိုတဲ့ concept တွေနဲ့ Langfuse data model
- Session နဲ့ user အလိုက် ခွဲခြား စုစည်းနည်း
- Prompt, completion, token usage တွေကို capture လုပ်နည်း
- Feature အလိုက် tag လုပ်ပြီး filter လုပ်နည်း
- Trace data ကို အသုံးချပြီး ဆိုးတဲ့ အဖြေတစ်ခုကို debug လုပ်နည်း

## သင်ခန်းစာများ

1. Trace, Span, Generation ဆိုတာ ဘာတွေလဲ
2. Session နဲ့ User ခွဲခြားနည်း
3. Prompt, Completion, Usage capture လုပ်နည်း
4. Feature အလိုက် Tagging လုပ်နည်း
5. Trace-based Debugging နည်း

## လိုအပ်ချက်များ (Prerequisites)

- Week 1–4 အထိရှိတဲ့ Python အ基础 နဲ့ LLM API ခေါ်သုံးနည်း နားလည်မှု
- `pip install langfuse` နဲ့ Langfuse account (cloud သို့မဟုတ် self-hosted)
- Langfuse project ရဲ့ `LANGFUSE_PUBLIC_KEY` နဲ့ `LANGFUSE_SECRET_KEY`

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Application ထဲမှာ LLM ခေါ်နေပြီးသည်မှစပြီး trace မလုပ်ထားရင် — debug ခက်ပြီး cost တွေကို အရင်မြင်ရတော့မှာ မဟုတ်ပါဘူး။
- Team ထဲမှာ quality ပြဿနာတွေကို data နဲ့ ပြောရမယ့် အချိန်မှာ အရေးကြီးဆုံး အခြေခံ ဖြစ်ပါတယ်။
- Week 5 ရဲ့ evaluation နဲ့ monitoring အပိုင်းတွေအတွက် ဒီ tracing က အခြေခံ data ပဲ ဖြစ်ပါတယ်။

## ကိုးကား

- Langfuse Documentation: https://langfuse.com/docs
- Tracing concept: https://langfuse.com/docs/tracing
- Python SDK: https://langfuse.com/docs/sdk/python/low-level-sdk
- FAQ / troubleshooting: https://langfuse.com/docs/opentelemetry/get-started
