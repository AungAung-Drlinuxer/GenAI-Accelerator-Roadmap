# Background Jobs with Celery + Redis

LLM application များထဲမှာ request path ကနေ ခွဲထုတ်ထားတဲ့ background job တွေကို Celery နဲ့ Redis ကို သုံးပြီး ဘယ်လို စီမံခန့်ခွဲမလဲဆိုတာ သင်ကြားမယ့် module ဖြစ်ပါတယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Task queue ဆိုတာ ဘာလဲ၊ Celery broker အဖြစ် Redis ကို ဘယ်လို ချိတ်ဆက်မလဲ
- Worker pool တွေကို configure လုပ်ပြီး task တွေကိu parallel မှာ ဆက်တိုက် လည်ပတ်စေနည်း
- Retry နဲ့ backoff policy တွေကို သင့်တော်စွာ ချမှတ်နည်း
- Idempotency key သုံးပြီး task တွေ ထပ်မံ run သွားတာကနေ ကာကွယ်နည်း
- Periodic task တွေကိu scheduler နဲ့ ချိန်းဆိုနည်း
- Dead-letter handling နဲ့ ကျရှုံးတဲ့ task တွေကို နောက်ဆုံးမှာ ဘယ်လို စုစည်းနည်း
- ရှည်ကြာတဲ့ LLM job တွေကိu request path ကနေ ခွဲပြီး async အဖြစ် ဆက်တိုက် run စေနည်း

## သင်ခန်းစာများ

1. Task Queue အခြေခံ — Celery ရဲ့ architecture (broker, worker, result backend) ကို နားလည်ပါတယ်။
2. Redis ကို Broker အဖြစ် သုံးခြင်း — Redis connection နဲ့ broker settings တွေကိu configure လုပ်ပါတယ်။
3. Task ရေးဆွဲခြင်း — `@app.task` decorator, serialization နဲ့ argument passing ကို လက်တွေ့ စမ်းသပ်ပါတယ်။
4. Worker Pool စီမံခန့်ခွဲခြင်း — concurrency model တွေ (`prefork`, `threads`) ကို နှိုင်းယှဉ်ပြီး ရွေးချယ်ပါတယ်။
5. Retry နဲ့ Backoff — `autoretry_for`, `retry_backoff` တို့ကို သုံးပြီး LLM API ရဲ့ transient failure တွေကို ကိုင်တွယ်ပါတယ်။
6. Idempotency Key — task တစ်ခု ထပ်ခါထပ်ခါ ဆိုင်းပရား run သွားတာကိu ကာကွယ်နည်းကို ဆက်တိုက် အသုံးပြုပါတယ်။
7. Periodic Task နဲ့ Scheduling — `celery beat` ကို သုံးပြီး အချိန်စဉ်အတိုင်း run ရမယ့် job တွေကို ချိန်းပါတယ်။
8. Dead-Letter နဲ့ Error Handling — ကျရှုံးနေတဲ့ task တွေကိu log မှတ်ပြီး နောက်မှာ ပြန်လည် စစ်ဆေးနိုင်စေပါတယ်။
9. Long-running LLM Job တွေ Offloading လုပ်ခြင်း — HTTP request ကနေ ခွဲထုတ်ပြီး job ID နဲ့ status polling လုပ်နည်းကို တည်ဆောက်ပါတယ်။

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံ (function, decorator, virtual environment) ကို နားလည်ထားရပါမယ်။
- Week 2 က API service တည်ဆောက်နည်း သင်ခန်းစာတွေကို ပြီးမြောက်ထားရပါမယ်။
- Docker နဲ့ container အခြေခံ သိထားပါက ပိုအဆင်ပြေပါတယ်။
- Local machine မှာ `docker` နဲ့ `docker compose` run နိုင်ရပါမယ်။

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- LLM inference တစ်ခုက စက္ကန့်အချို့အထိ ကြာတဲ့အခါ၊ user request ကို မစောင့်စေပဲ job အဖြစ် ခွဲထားချင်ရင် အသုံးဝင်ပါတယ်။
- Batch processing (document အများအပြားကို summarize လုပ်တာမျိုး) မှာ တစ်ခါတည်း အားလုံးကို ဆက်တိုက် စီမံခန့်ခွဲချင်ရင် လိုအပ်ပါတယ်။
- Third-party API rate limit တွေကြောင့် retry နဲ့ backoff လုပ်ဖို့ လိုအပ်တဲ့ နေရာတွေမှာလည်း အသုံးဝင်ပါတယ်။

## ကိုးကား

- Celery တရားဝင် documentation — https://docs.celeryq.dev/
- Redis documentation — https://redis.io/docs/
- Celery task retries အပိုင်း — https://docs.celeryq.dev/en/stable/userguide/tasks.html#retrying
- Celery periodic tasks (beat) — https://docs.celeryq.dev/en/stable/userguide/periodic-tasks.html
