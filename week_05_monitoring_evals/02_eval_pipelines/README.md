# Evaluation Pipelines & Regression Gates

LLM application တစ်ခုရဲ့ quality ကို တိုင်းတာဖို့ dataset ဒီဇိုင်းဆွဲခြင်း၊ deterministic နဲ့ LLM-judge scoring ခွဲခြားသုံးခြင်း၊ CI pipeline ထဲမှာ eval လည်ပတ်စေပြီး deploy ကို block လုပ်တဲ့ regression gate တည်ဆောက်နည်းကို ဒီ module မှာ လေ့လာပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Prompt version တစ်ခုစီအတွက် evaluation dataset ကို စနစ်တကျ ဒီဇိုင်းဆွဲနည်း
- Rule-based (deterministic) scoring နဲ့ LLM-as-a-judge scoring ရဲ့ ကွာခြားချက်နဲ့ သင့်တော်ရာ ရွေးချယ်နည်း
- pytest သုံးပြီး evaluation run တွေကို CI pipeline ထဲထည့်跑တင်နည်း
- Threshold သတ်မှတ်ပြီး regression ဖြစ်ရင် deploy ကို ရပ်တန့်စေတဲ့ gate ဆောက်နည်း
- Prompt version အလိုက် score တွေကို Langfuse နဲ့ တိုက်ရိုက် မှတ်တမ်းတင်ပြီး နှိုင်းယှဉ်ကြည့်နည်း

## သင်ခန်းစာများ

1. **Eval Dataset ဒီဇိုင်း** — test case မျိုးစုံ (edge case, golden answer ပါတဲ့ case) နဲ့ dataset structure ကို JSON/CSV format နဲ့ စုစည်းနည်း သင်ယူမယ်။
2. **Deterministic Scoring** — exact match, keyword check, code test စတဲ့ တိတိကျကျ စစ်ဆေးနိုင်တဲ့ scorer တွေကို Python နဲ့ ရေးသားနည်း လေ့လာမယ်။
3. **LLM-as-a-Judge** — rubric prompt သုံးပြီး output quality ကို အမှတ်ပေးစေနည်းနဲ့ သတိထားရမည့် bias အချက်တွေကို ဖော်ပြပါမယ်။
4. **pytest နဲ့ Eval Run** — evaluation suite ကို pytest test အဖြစ် ရေးပြီး CI ထဲမှာ အလိုအလျောက် လည်ပတ်စေနည်း လက်တွေ့ လုပ်ကြည့်မယ်။
5. **Regression Gate ထည့်သွင်းခြင်း** — score threshold သတ်မှတ်ပြီး အောက်ကျရင် pipeline fail ဖြစ်စေတဲ့ exit-code logic ကို ဆောက်မယ်။
6. **Languse နဲ့ Score Tracking** — trace နဲ့ score တွေကို Languse မှာ မှတ်တမ်းတင်ပြီး prompt version အလိုက် နှိုင်းယှဉ်ကြည့်နည်း သင်မယ်။
7. **End-to-End Mini Project** — dataset → eval script → CI gate → Languse dashboard အထိ တစ်ကြောင်းတည်း ချိတ်ဆက်ထားတဲ့ pipeline အသေးစား တစ်ခု တည်ဆောက်မယ်။

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံနဲ့ pytest ရေးသားနိုင်စွမ်း (Week 1–4 များ ပြီးစေရန်)
- OpenAI-compatible API (သို့) local model တစ်ခုကို ခေါ်ဆိုနိုင်တဲ့ ကျွမ်းကျင်မှု
- Git နဲ့ GitHub Actions (သို့မဟုတ် အလားတူ CI system) အခြေခံ သိရှိမှု
- Languse account တစ်ခု (free tier အသုံးပြုနိုင်)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Prompt တစ်ခုကို ပြင်လိုက်တိုင်း quality ကျသွားမသွား စိတ်ချရအောင် စစ်ချင်တဲ့အခါ
- Production မှာ လည်ပတ်နေတဲ့ LLM feature တစ်ခုကို model upgrade မလုပ်ခင် safety-net လိုအပ်တဲ့အခါ
- Team အတွင်း prompt version တွေရဲ့ အရည်အသွေးကို data နဲ့ အခြေခံပြီး ဆွေးနွေးချင်တဲ့အခါ
- CI pipeline ထဲမှာ human review မလိုဘဲ အလိုအလျောက် deploy block လုပ်စေချင်တဲ့အခါ

## ကိုးကား

- Languse Documentation — Evaluation & Scores: https://langfuse.com/docs/scores/overview
- Languse Documentation — Prompt Management: https://langfuse.com/docs/prompts
- pytest Documentation — Get Started: https://docs.pytest.org/en/7.4.x/getting-started.html
- pytest Documentation — Continuous Integration: https://docs.pytest.org/en/7.4.x/explanation/goodpractices.html
- GitHub Actions Documentation — Workflow syntax: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions
