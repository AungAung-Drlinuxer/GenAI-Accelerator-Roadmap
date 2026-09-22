# LLM APIs & Providers (hosted + on-prem)

AI Engineering ရဲ့ ပထမဆုံး လက်တွေ့အသုံးချမှု module — hosted LLM APIs နဲ့ on-prem inference နှစ်မျိုးလုံးကို တစ်ပြိုင်တည်း လေ့လာမယ့် သင်ခန်းစာ။

## ဒီ module မှာ ဘာသင်မလဲ

- OpenAI-compatible Chat Completions request/response ပုံစံကို နားလည်အောင် လေ့လာခြင်း
- Streaming နဲ့ non-streaming response များရဲ့ ကွာခြားချက်နဲ့ အသုံးပြုပုံ
- API call များအတွက် retry, backoff, timeout စနစ်တည်ဆောက်ပုံ
- Token တွက်ချက်မှုနဲ့ cost ခန့်မှန်းတွက်ချက်ပုံ
- Ollama ကိုသုံးပြီး local machine ပေါ်မှာ LLM run ပုံ

## သင်ခန်းစာများ

1. **Chat Completions အခြေခံပုံစံ** — messages, roles, temperature စတဲ့ core parameters များနဲ့ Python မှာ ပထမဆုံး request ပို့ပုံ။
2. **Streaming vs Non-streaming** — `stream=True` သုံးပြီး token-by-token response လက်ခံပုံနဲ့ ဘယ်အခြေအနေမှာ ဘယ်လို method သင့်တော်လဲ။
3. **Reliability: Retries & Timeouts** — network error, rate limit တွေကို ကိုင်တွယ်ဖို့ exponential backoff နဲ့ timeout သတ်မှတ်ပုံ။
4. **Token Accounting & Cost** — `usage` field ဖတ်ပုံ၊ context window နဲ့ ကုန်ကျစရိတ် ခန့်မှန်းတွက်ချက်ပုံ။
5. **Ollama for On-Prem Inference** — local machine ပေါ်မှာ Ollama install လုပ်ပြီး OpenAI-compatible endpoint အဖြစ် ချိတ်ဆက်ပုံ။

## လိုအပ်ချက်များ (Prerequisites)

- Python 3.10 နဲ့အထက်၊ `pip` သုံးတတ်ရမယ်
- `requests` သို့မဟုတ် `openai` Python package install ထားရမယ်
- Hosted API အတွက် OpenAI API key တစ်ခု (သို့) Ollama အတွက် local installation
- HTTP request/response အခြေခံ ဗဟုသုတ (method, headers, JSON body)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- သင့် application ထဲမှာ LLM ကို production level နဲ့ ချိတ်ဆက်ချင်တဲ့အခါ
- API call တွေကို တည်ငြိမ်အောင်၊ ကုန်ကျစရိတ် ထိန်းသိမ်းအောင် စနစ်တကျ စီမံချင်တဲ့အခါ
- Data privacy ကြောင့် cloud API မသုံးနိုင်ဘဲ on-prem/local model လိုအပ်တဲ့အခါ
- OpenAI-compatible server တွေ (Ollama, vLLM စသည်) ကို တစ်ခုတည်းသော client code နဲ့ ချိတ်ဆက်ချင်တဲ့အခါ

## ကိုးကား

- OpenAI API Reference — Chat Completions: <https://platform.openai.com/docs/api-reference/chat>
- OpenAI API Reference — Streaming: <https://platform.openai.com/docs/api-reference/streaming>
- OpenAI API Reference — Errors & Rate Limits: <https://platform.openai.com/docs/guides/rate-limits>
- OpenAI Usage & Token Documentation: <https://platform.openai.com/docs/api-reference/usage>
- Ollama GitHub Repository: <https://github.com/ollama/ollama>
- Ollama OpenAI Compatibility Docs: <https://github.com/ollama/ollama/blob/main/docs/openai.md>
