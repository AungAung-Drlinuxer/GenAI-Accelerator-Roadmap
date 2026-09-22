# Week 1 — Foundations of AI Engineering

## ဒီပတ်မှာ ဘာသင်မလဲ

Production AI engineering အတွက် အခြေခံအကျဆုံးဖြစ်တဲ့ အရာများကို ဒီပတ်မှာ စတင်သင်ကြားပါမယ် —

- **uv** ကို သုံးပြီး Python project တစ်ခုကို ပြန်လည်ထုတ်နိုင်တဲ့ (reproducible) ပုံစံနဲ့ စီမံခြင်း
- Prompt engineering ၏ အခြေခံသဘောတရားများ — system/user prompts၊ few-shot examples၊ output format ကန့်သတ်ချက်များ
- LLM APIs တွေကို ခေါ်ယူအသုံးပြုခြင်း — Chat Completions shape၊ streaming၊ retries၊ token နဲ့ cost စာရင်းစစ်ခြင်း
- Code quality toolchain — `ruff`, `pytest`, secrets hygiene နဲ့ pre-commit gates

## Modules

- `01_python_uv_workflow/` — **Python & uv Project Workflow for AI Engineering** — `pyproject.toml`၊ lockfile၊ `uv run` တွေကို သုံးပြီး library version များ အတိအကျသတ်မှတ်ထားတဲ့ environment တစ်ခု တည်ဆောက်နည်းကို သင်ပါမယ်။
- `02_prompt_engineering/` — **Prompt Engineering Fundamentals** — System prompt နဲ့ user prompt ကွာခြားချက်၊ instruction hierarchy၊ few-shot examples၊ output format ကန့်သတ်ခြင်း၊ temperature/top-p တို့ရဲ့ သက်ရောက်မှုကို လက်တွေ့စမ်းသပ်ပါမယ်။
- `03_llm_apis_providers/` — **LLM APIs & Providers (hosted + on-prem)** — OpenAI-compatible Chat Completions ရဲ့ ဖွဲ့စည်းပုံ၊ streaming၊ retries/backoff၊ timeouts၊ token နဲ့ cost တွက်ချက်ပုံ၊ on-prem အတွက် Ollama အသုံးပြုနည်းကို သင်ပါမယ်။
- `04_quality_toolchain/` — **Quality Toolchain** — `ruff` နဲ့ lint/format လုပ်ခြင်း၊ `pytest` structure နဲ့ fixtures၊ API keys တွေကို `.env` ထဲမှာ သိမ်းပြီး git မထဲ မဝင်စေဘဲ လုံခြုံအောင် ထိန်းသိမ်းနည်းကို လက်တွေ့လုပ်ပါမယ်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ် ပြီးတဲ့အခါ လေ့လာသူတစ်ယောက်ဟာ —

1. `uv` သုံးပြီး lock လုပ်ထားတဲ့ dependencies နဲ့ Python project တစ်ခုကို အခြား machine တစ်ခုပေါ်မှာ အတိအကျ ပြန် run နိုင်မယ်။
2. ရည်မှန်းထားတဲ့ output format ရအောင် prompt တစ်ခုကို system/user role ခွဲခြားပြီး၊ few-shot examples ထည့်သွင်းရေးဆွဲနိုင်မယ်။
3. LLM API တစ်ခုကို Python ကနေ streaming နဲ့ non-streaming နည်းနဲ့ ခေါ်ပြီး၊ errors တွေအတွက် retry/backoff logic ထည့်နိုင်မယ်။
4. API key တွေကို လုံခြုံစွာ ထိန်းပြီး၊ `ruff` နဲ့ `pytest` ပါတဲ့ အနိမ့်ဆုံး quality gate တစ်ခု တည်ဆောက်နိုင်မယ်။

## လေ့လာရန် အစီအစဉ်

| နေ့ | Module |
| --- | --- |
| Day 1–2 | `01_python_uv_workflow/` |
| Day 3–4 | `02_prompt_engineering/` |
| Day 4–5 | `03_llm_apis_providers/` |
| Day 6 | `04_quality_toolchain/` |
| Day 7 | Checkpoint ပြန်လည်စစ်ဆေးခြင်းနဲ့ ပြန်လေ့ကျင့်ခြင်း |

`03_llm_apis_providers/` ကို စမှ 02 ရဲ့ prompt တွေကို code နဲ့ တကွ စမ်းသပ်ရမလို့ 02 ကို အရင်ပြီးအောင် လေ့လာထားဖို့ အကြံပြုပါတယ်။

## Checkpoint

1. LLM တစ်ခုကို ခေါ်တဲ့ script တစ်ခုဟာ library version မတူရင် ဘာကြောင့် အမူအကျင့် ကွာခြားနိုင်တာလဲ — `uv.lock` က ဒီပြဿနာကို ဘယ်လို ဖြေရှင်ပေးလဲ။
2. System prompt တစ်ခုနဲ့ user prompt တစ်ခုရဲ့ ကွာခြားချက်က ဘာလဲ — JSON output တောင်းခံချင်ရင် prompt ကို ဘယ်လို ရေးသင့်လဲ။
3. API call တစ်ခုဟာ timeout ပေးပြီး မှားရင် retry လုပ်စေချင်ရင် code ကို ဘယ်လို ရေးမလဲ — backoff ဆိုတာ ဘာကို ဆိုလိုတာလဲ။
4. API key တစ်ခုကို source code ထဲ တိုက်ရိုက် မရေးသင့်တာက ဘာကြောင့်လဲ — key တွေကို ဘယ်နေရာမှာ သိမ်းသင့်လဲ။
