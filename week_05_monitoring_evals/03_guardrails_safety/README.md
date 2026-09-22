# Guardrails, PII & Prompt Injection

LLM application များအတွက် input/output validation, PII redaction နှင့် prompt injection ကာကွယ်မှုများကို လက်တွေ့ကျကျ လေ့လာသည့် module ဖြစ်သည်။

## ဒီ module မှာ ဘာသင်မလဲ

- Pydantic ဖြင့် input/output validation စနစ်တည်ဆောက်နည်း
- Allow-list / deny-list အခြေခံကျသော ထိန်းချုပ်မှုပုံစံများ
- Prompt injection တိုက်ရိုက် / သွယ်ဝိုက် အမျိုးအစားများနှင့် ကာကွယ်နည်း
- Log ထဲမှ PII နှင့် secret များကို redact လုပ်နည်း
- Refusal pattern များရေးသားနည်း
- Tool-permission boundary သတ်မှတ်နည်း

## သင်ခန်းစာများ

1. Pydantic ဖြင့် structured input validation
2. Output validation နှင့် allow-list စိစစ်ခြင်း
3. Prompt injection ကာကွယ်မှု (direct / indirect)
4. PII နှင့် secret redaction
5. Refusal patterns နှင့် safe fallback
6. Tool permission boundaries

## လိုအပ်ချက်များ (Prerequisites)

- Python အ基础 (function, class, exception) ရေးတတ်ရမည်
- Pydantic basic usage ကို သိရှိရမည် (`pip install pydantic`)
- LLM API ခေါ်သည့် code ဖတ်နိုင်စွမ်းရှိရမည်
- Regular expression အခြေခံကို နားလည်ရမည်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- User input ကို LLM ဆီပို့မည့်အခါ စိစစ်ရန်
- LLM output ကို downstream system ဆီမသွားခင် စစ်ဆေးရန်
- Production log မှ ကိုယ်ရေးအချက်အလက် မှန်မှားမှုမှ ကာကွယ်ရန်
- Tool/agent စနစ်တွင် လုပ်ပိုင်ခွင့် ကန့်သတ်ရန်

## ကိုးကား

- Pydantic Docs: https://docs.pydantic.dev/
- OWASP LLM Top 10: https://genai.owasp.org/llm-top-10/
- Model Context Protocol — Security: https://modelcontextprotocol.io/specification/security-and-privacy
- NeMo Guardrails (NVIDIA) Docs: https://docs.nvidia.com/nemo/guardrails/
