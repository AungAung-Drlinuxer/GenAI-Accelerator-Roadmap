# Cost, Latency & Model Routing

ဒီ module မှာ LLM application တစ်ခုရဲ့ ကုန်ကျစရိတ် (cost)၊ တုံ့ပြန်ချိန် (latency) နှင့် model ရွေးချယ်မှု (routing) ကို တိုင်းတာခြင်း၊ ထိန်းချုပ်ခြင်း နည်းလမ်းများကို လေ့လာပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Feature အလိုက် token နှင့် ကုန်ကျစရိတ် စာရင်းခြေရှင်းနည်း
- Latency budget သတ်မှတ်ပြီး တိုင်းတာနည်း
- Caching layer ထည့်သွင်းပြီး cost လျှော့ချနည်း
- သေးငယ်/ကြီးမား model routing ဆုံးဖြတ်ပုံ
- On-prem fallback model ထားရှိနည်း
- Alert threshold သတ်မှတ်ကြည့်ရှုနည်း

## သင်ခန်းစာများ

1. Token/cost accounting per feature
2. Latency budget သတ်မှတ်ခြင်းနှင့် တိုင်းတာခြင်း
3. Caching layer များ
4. Small-vs-large model routing
5. On-prem fallback models
6. Alert thresholds

## လိုအပ်ချက်များ (Prerequisites)

- Week 1–4 lessons များကို ပြီးမြောက်ထားရမယ်
- Python basics (function, dictionary, class)
- OpenAI-compatible API ခေါ်ယူမှု အခြေခံ
- Langfuse observation အခြေခံ (Week 5 ရှေ့ module)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Production မှာ LLM app စရိတ်က ကြီးလာတဲ့အချိန်
- User experience အတွက် response နှောင့်နှေးမှုကို ဖြေရှင်းချင်တဲ့အချိန်
- High-traffic feature တစ်ခုကို ကုန်ကျစရိတ်သက်သက်သာသာ လုပ်ဆောင်ချင်တဲ့အချိန်

## ကိုးကား

- Langfuse Docs — Model Usage & Cost Analytics: https://langfuse.com/docs/model-usage
- Langfuse.com/docs (documentation portal)
- OpenAI API Pricing Docs: https://platform.openai.com/docs/pricing
- Hashicorp/Redis caching အခြေခံနှင့် Langfuse prompt caching examples
