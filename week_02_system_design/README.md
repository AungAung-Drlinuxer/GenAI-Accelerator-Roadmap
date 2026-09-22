# Week 2 — AI System Design Principles

## ဒီပတ်မှာ ဘာသင်မလဲ

ဒီပတ်သည် LLM ကို "အလွတ်သဘော" ဘက်က ခေါ်သုံးသည့်အဆင့်မှ Production စနစ်တစ်ခုအဖြစ် စနစ်တကျ ဒီဇိုင်းရေးဆွဲနိုင်ရန် အခြေခံသဘောတရားများကို သင်ကြားပေးပါသည်။ အဓိက အကြောင်းအရာများမှာ—

- **Pydantic contracts** — LLM ၏ input/output ကို typed schema ဖြင့် တိကျစွာ သတ်မှတ်ခြင်း
- **Structured output & repair loops** — validation မအောင်မြင်လျှင် ပြန်လည်ပြင်ဆင်သည့် loop ဖန်တီးခြင်း
- **Context engineering** — context window ထဲ ဘာထည့်သင့်၊ ဘာမထည့်သင့်နှင့် token budget စီမံခြင်း
- **Modular architecture** — orchestration နှင့် business logic ခွဲခြားခြင်း၊ dependency injection ဖြင့် စမ်းသပ်လွယ်သည့် structure ဆောက်ခြင်း

## Modules

- `01_pydantic_contracts/` — **Pydantic v2 Contracts for LLM I/O**: `BaseModel`, `Field` constraints, custom validators, `model_json_schema()` ဖြင့် tool/JSON-schema driven prompting နှင့် serialization round-trip များ လက်တွေ့သုံး သင်ကြားသည်။
- `02_structured_output/` — **Structured Output & Repair Loops**: Schema-first ဒီဇိုင်း၊ JSON mode / function calling၊ validation errors ကို feedback အဖြစ် ပြန်ပို့ခြင်း၊ bounded retry loop နှင့် deterministic fallback များ သင်ကြားသည်။
- `03_context_engineering/` — **Context Engineering & Token Budgets**: context window ထဲ ထည့်သွင်းရမည့် အချက်အလက်များ၊ retrieval vs stuffing၊ truncation ၏ တာဝန်ခံမှု၊ summarisation၊ prompt caching နှင့် context size ၏ cost/latency သက်ရောက်မှုများ သင်ကြားသည်။
- `04_modular_architecture/` — **Modular Architecture: Workflows, Nodes, DI**: orchestration နှင့် steps ခွဲခြားခြင်း၊ typed node inputs/outputs၊ dependency injection၊ failure isolation နှင့် observability hooks များ သင်ကြားသည်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ်အဆုံးမှာ သင်သည်—

- LLM application တစ်ခု၏ input/output ကို Pydantic schema ဖြင့် တိကျစွာ သတ်မှတ်ပြီး schema မှ prompt ထုတ်နိုင်ခြင်း
- Structured output မှ ရလာသည့် output ကို စစ်ဆေးပြီး မမှန်လျှင် bounded repair loop ဖြင့် ပြန်လည်ပြင်ဆင်စေနိုင်ခြင်း
- Context window ကို တာဝန်ခံစွာ စီမံပြီး token cost နှင့် latency ကို ထိန်းနိုင်ခြင်း
- Logic များကို ပြန်လည်အသုံးပြုနိုင်သော၊ စမ်းသပ်နိုင်သော modular workflow အဖြစ် ခွဲခြားတည်ဆောက်နိုင်ခြင်း

## လေ့လာရန် အစီအစဉ်

အောက်ပါအစဉ်အလိုက် လေ့လာရန် အကြံပြုပါသည် (ပုံမှန် ၅ ရက် အလုပ်ရက်များအတွက်)—

1. **Day 1–2** — `01_pydantic_contracts/`: Pydantic v2 အခြေခံနှင့် contract ဒီဇိုင်း
2. **Day 2–3** — `02_structured_output/`: Schema-first ရေးသားမှုနှင့် repair loop တည်ဆောက်ခြင်း
3. **Day 4** — `03_context_engineering/`: Context စီမံခန့်ခွဲမှုနှင့် token budget
4. **Day 5** — `04_modular_architecture/`: Modular workflow ဒီဇိုင်း၊ ထပ်တူ ပြန်လည်ဆန်းစစ်ခြင်း

## Checkpoint

- Pydantic `Field` constraints များ (ဥပမာ `ge`, `max_length`) က LLM output validation မှာ ဘယ်လို အလုပ်လုပ်ပါသလဲ။
- Repair loop တစ်ခုကို ဘာကြောင့် **bounded** (အကန့်အသတ်ရှိရမ်) အဖြစ် ဒီဇိုင်းရမှာလဲ၊ unbounded loop ရဲ့ အန္တရာယ်က ဘာလဲ။
- Context window ထဲသို့ document တစ်ခုလုံး stuff လုပ်ခြင်းနှင့် retrieval ဖြင့် အပိုင်းလိုက် ထည့်ခြင်းရဲ့ ခြားနားချက်က ဘာလဲ။
- Workflow orchestration နှင့် individual node logic ကို ခွဲခြားခြင်းက စမ်းသပ်မှု (testing) အတွက် ဘယ်လို အထောက်အကူ ဖြစ်စေပါသလဲ။
