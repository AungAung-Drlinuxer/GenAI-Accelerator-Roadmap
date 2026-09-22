# Python & uv Project Workflow for AI Engineering

uv ကို အသုံးပြု၍ Python project တစ်ခုကို သတ်မှတ်ချက်ရှိရှိ (deterministic) စနစ်တကျ စီမံခန့်ခွဲနည်းကို သင်ကြားမည့် module ဖြစ်သည်။

## ဒီ module မှာ ဘာသင်မလဲ

- `pyproject.toml` ဖိုင်ရဲ့ ဖွဲ့စည်းပုံနဲ့ သူက project dependency တွေကို ဘယ်လို သတ်မှတ်လဲ
- Lockfile (`uv.lock`) က ဘာကြောင့် environment တွေကို တူညီအောင် ထိန်းပေးနိုင်လဲ
- `uv run` command နဲ့ ဘယ်လို script တွေကို လွယ်ကူစွာ run လို့ရလဲ
- LLM application တစ်ခုမှာ library version တွေ တူညီမှု (reproducibility) က ဘာကြောင့် အရေးကြီးလဲ

## သင်ခန်းစာများ

1. **pyproject.toml အခြေခံ** — project metadata နှင့် dependency သတ်မှတ်ခြင်း။
2. **Lockfile နှင့် deterministic environment** — `uv.lock` ဖိုင်က ဘယ်လို အာမခံလဲ။
3. **uv run နှင့် နေ့စဉ်အသုံး** — virtual environment မဝင်ဘဲ script run ခြင်း။
4. **Reproducibility နှင့် LLM application** — version မတူခြင်းက ဘာကြောင့် ပြဿနာဖြစ်စေလဲ။

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံ (function, import, pip အကြောင်း အနည်းငယ်) သိရှိရမည်။
- Terminal/command line အခြေခံ command တွေ (`cd`, `ls`) သုံးနိုင်ရမည်။
- `uv` ကို install ပြုလုပ်ထားရမည် — official docs မှာ နည်းပါ ရှင်းပြထားသည်။

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- LLM API ချိတ်တဲ့ Python project တစ်ခုကို စတင်တည်ဆောက်ချိန်။
- Team တစ်ခုလုံး တူညီတဲ့ dependency version တွေနဲ့ အလုပ်လုပ်ချင်ချိန်။
- CI/CD pipeline မှာ test run တွေကို ယုံကြည်စိတ်ချရအောင် လုပ်ချင်ချိန်။

## ကိုးကား

- uv official documentation — https://docs.astral.sh/uv/
- uv project guide — https://docs.astral.sh/uv/guides/projects/
- uv lockfile concept — https://docs.astral.sh/uv/concepts/projects/
- Python `pyproject.toml` standard (PEP 621) — https://packaging.python.org/en/latest/specifications/pyproject-toml/
