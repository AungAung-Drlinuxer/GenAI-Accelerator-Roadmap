# Week 5 — LLM Monitoring & Evaluations

ဒီပတ်က production ပတ်ဝန်းကျင်မှာ LLM application တွေကို တိုင်းတာခြင်း၊ အမှားရှာဖွေခြင်း၊ အရည်အသွေးစစ်ဆေးခြင်းနဲ့ ဘေးအန္တရာယ်ကာကွယ်ခြင်းတွေအကြောင်း အဓိကထား သင်ကြားပါမယ်။ Model ကို deploy လုပ်ပြီးတာနဲ့ အလုပ်ပြီးသွားတာ မဟုတ်ပါဘူး — အသုံးပြုသူတွေဆီက ဘယ်လိုအဖြေတွေ ထွက်နေလဲ၊ ကုန်ကျစရိတ်နဲ့ latency ဘယ်လောက်ရှိလဲ၊ အသစ်ပြောင်းတဲ့ prompt က ရလဒ်ကို ပိုဆိုးစေသလား ဆိုတာတွေကို မြင်နိုင်ရပါမယ်။

## ဒီပတ်မှာ ဘာသင်မလဲ

- Langfuse သုံးပြီး prompt, completion, token usage နဲ့ latency တွေကို trace လုပ်ခြင်း
- Session နဲ့ user အလိုက် ခွဲခြားကြည့်ခြင်း၊ feature tag တပ်ခြင်း
- Bad answer တစ်ခုကို trace data သုံးပြီး debug လုပ်ခြင်း
- Eval dataset ဒီဇိုင်းချခြင်းနဲ့ deterministic vs LLM-judge scoring
- CI pipeline ထဲမှာ eval လုပ်ကြည့်ပြီး threshold ကျရင် deploy ရပ်တန့်စေခြင်း (regression gate)
- Prompt version အလိုက် score ဖြစ်ပြောင်းမှုကို စောင့်ကြည့်ခြင်း
- Pydantic နဲ့ input/output validation၊ prompt injection ကာကွယ်ခြင်း
- Log ထဲက PII နဲ့ secret တွေကို redact လုပ်ခြင်း
- Feature အလိုက် cost နဲ့ latency budget သတ်မှတ်ခြင်း၊ caching နဲ့ model routing

## Modules

- `01_tracing_langfuse/` — **Tracing with Langfuse** — LLM call တိုင်းရဲ့ prompt, completion, usage, latency တွေကို span/generation အနေနဲ့ ဖမ်းယူပြီး session/user/tag အလိုက် စုစည်းကြည့်နည်းနဲ့ trace သုံး debug လုပ်နည်း သင်ပါတယ်။
- `02_eval_pipelines/` — **Evaluation Pipelines & Regression Gates** — Eval dataset တည်ဆောက်နည်း၊ deterministic နဲ့ LLM-judge score ထုတ်နည်း၊ CI ထဲမှာ eval ပြေးပြီး threshold ကျရင် deploy တားဆီးနည်း သင်ပါတယ်။
- `03_guardrails_safety/` — **Guardrails, PII & Prompt Injection** — Input/output ကို validate လုပ်နည်း၊ injection တုံ့ပြန်နည်း၊ log ထဲ PII redact လုပ်နည်းနဲ့ tool သုံးခွင့်နယ်နိမိတ်သတ်နည်း သင်ပါတယ်။
- `04_cost_latency_slo/` — **Cost, Latency & Model Routing** — Feature အလိုက် token/cost စာရင်းမှန်းနည်း၊ latency budget သတ်မှတ်နည်း၊ caching နဲ့ small/large model routing လုပ်နည်း သင်ပါတယ်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ် ပြီးဆုံးရင် သင်ဟာ — LLM application တစ်ခုကို observability ရှိစေရင်းနဲ့ production မှာ ထားနိုင်မယ်။ Bad answer တစ်ခုကို trace ပြန်ကြည့်ပြီး ဘာကြောင့်ဖြစ်တယ်ဆိုတာ ရှာနိုင်မယ်၊ prompt အသစ်တိုင်းကို eval ပြေးပြီး ရလဒ်ကျလာရင် deploy မလုပ်ရဘူးဆိုတဲ့ gate ထားနိုင်မယ်၊ အသုံးပြုသူ input ကနေလာတဲ့ အန္တရာယ်တွေကို guardrail နဲ့ ကာနိုင်မယ်၊ ကုန်ကျစရိတ်နဲ့ latency ကို feature အလိုက် ကြီးကြပ်နိုင်မယ်။

## လေ့လာရန် အစီအစင်

| နေ့ | Module |
|-----|--------|
| Day 1–2 | `01_tracing_langfuse/` — Langfuse နဲ့ tracing အခြေခံ၊ trace-based debugging |
| Day 3 | `02_eval_pipelines/` — Dataset design၊ scoring၊ CI regression gate |
| Day 4 | `03_guardrails_safety/` — Validation၊ PII redaction၊ injection defence |
| Day 5 | `04_cost_latency_slo/` — Cost accounting၊ latency budget၊ model routing |
| Day 6–7 | Mini-project — ဒီပတ်က techniques အားလုံးကို သင့် app တစ်ခုမှာ တွဲစပ်သုံးကြည့်ခြင်း |

## Checkpoint

1. Langfuse ထဲမှာ generation span တစ်ခုက ပုံမှန်အားဖြင့် prompt, completion, usage နဲ့ ဘာတွေကို သိမ်းဆည်းသလဲ။ Bad answer တစ်ခုကို debug လုပ်ရာမှာ trace data က ဘယ်လိုကူညီနိုင်လဲ။
2. Deterministic scoring နဲ့ LLM-judge scoring ရဲ့ ကွာခြားချက်က ဘာလဲ။ ဘယ်နေရာမှာ ဘယ်ဟာကို သုံးသင့်လဲ။
3. Prompt version အသစ်တစ်ခုက eval score ကို ကျစေခဲ့ရင် regression gate က deploy ကို ဘယ်လို တားဆီးသလဲ။
4. Prompt injection attack တစ်ခုကို ကာကွယ်ဖို့ input validation ရော၊ tool permission boundary ရော ဘယ်လို သတ်မှတ်သင့်လဲ။
