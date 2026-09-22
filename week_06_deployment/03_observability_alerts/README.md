# Observability, Errors & Alerts

AI application တစ်ခုကို production မှာ ချောချောမောမော လည်ပတ်နေစေဖို့အတွက် structured logging, error tracking, metrics, SLOs နှင့် on-call playbook တို့ကို ဘယ်လိုတည်ဆောက်မလဲ ဆိုတာကို ဒီ module မှာ လက်တွေ့ကျကျ လေ့လာမှာ ဖြစ်ပါတယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Structured logging (JSON format) နှင့် LLM trace logging ကို ဘယ်လိုဖန်တီးမလဲ
- Sentry ကို အသုံးပြုပြီး exception များကို အလိုအလျောက် ဖမ်းဆီးတင်ပြမှု
- Request latency နှင့် error rate တို့အတွက် metrics စုဆောင်းခြင်းနှင့် dashboard ဆွဲခြင်း
- p95 latency နှင့် error rate အတွက် SLO သတ်မှတ်ခြင်းနှင့် စစ်ဆေးခြင်း
- Alert တက်လာတဲ့အခါ လိုက်နရမယ့် on-call playbook ရေးဆွဲခြင်း

## သင်ခန်းစာများ

1. **Structured Logging အခြေခံများ** — log တစ်ခုကို JSON structure နဲ့ ရေးပြီး search လုပ်ရလွယ်ကူစေဖို့ pattern တွေကို လေ့လာပါမယ်။
2. **Sentry နှင့် Error Tracking** — Python SDK ကို install လုပ်ပြီး unhandled exception တွေကို Sentry dashboard ပေါ် တင်ပြနိုင်အောင် စနစ်တကျ တည်ဆောက်ပါမယ်။
3. **Metrics နှင့် Dashboards** — request count, latency, token usage စသည့် metrics တွေကို စုဆောင်းပြီး dashboard ပေါ်မှာ မြင်နိုင်စေဖို့ ပြုလုပ်ပါမယ်။
4. **SLOs: p95 Latency & Error Rate** — service quality ကို တိုင်းတာဖို့ SLO သတ်မှတ်ပြီး p95 latency နဲ့ error rate တို့ကို monitor လုပ်ပါမယ်။
5. **On-call Playbook ရေးဆွဲခြင်း** — alert တက်ရင် ဘယ်လို တုံ့ပြန်ရမလဲဆိုတဲ့ အဆင့်ဆင့် လမ်းညွှန်ချက် document တစ်ခုကို ရေးသင့်တဲ့ပုံစံ လေ့လာပါမယ်။

## လိုအပ်ချက်များ (Prerequisites)

- Week 1–5 မှ FastAPI application တည်ဆောက်ခြင်းနှင့် LLM integration အသိအမြင်များ
- Python ဗasic syntax နှင့် exception handling (`try/except`) နားလည်မှု
- `pip` နှင့် virtual environment အသုံးပြုနိုင်စွမ်း
- Docker container basics (metrics collector များ စမ်းသပ်ရာတွင် အသုံးဝင်ပါတယ်)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- AI application တစ်ခုကို production environment ထဲ တင်ပြီးသည့်အခါ
- User တွေက error တွေကို ဖောက်သည်ဆီကနေ မကြားရခင် အရင်သိရှိနိုင်ချင်တဲ့အခါ
- Response နှောင့်နှေးမှု သို့မဟုတ် API timeout ပြဿနာတွေကို စစ်ဆေးချင်တဲ့အခါ
- Team တစ်ခုအနေနဲ့ on-call rotation စနစ်တည်ဆောက်ချင်တဲ့အခါ

## ကိုးကား

- Sentry Python SDK documentation: https://docs.sentry.io/platforms/python/
- Sentry error monitoring concepts: https://docs.sentry.io/product/sentry-basics/concepts/
- Langfuse tracing and observability docs: https://langfuse.com/docs
- Python `logging` module (JSON structured logging အတွက် အခြေခံ): https://docs.python.org/3/library/logging.html
