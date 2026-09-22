# Structured Output & Repair Loops

AI system တစ်ခုကနေ ယုံကြည်စိတ်ချရတဲ့ structured data ထုတ်ယူဖို့ schema-first design၊ validation၊ နဲ့ repair loop တွေကို လေ့လာမယ့် module ပါ။

## ဒီ module မှာ ဘာသင်မလဲ

- Schema-first design နည်းပညာနဲ့ Pydantic model တွေ ဘယ်လို ရေးသင့်တာလဲ
- LLM ရဲ့ JSON mode နဲ့ function calling / structured outputs ကို ဘယ်လို သုံးမလဲ
- Validation error တွေကို feedback အဖြစ် ပြန်ပေးပြီး model ကို ဘယ်လို ဖြေရှင်းခိုင်းမလဲ
- Bounded repair retry — ကန့်သတ်ထားတဲ့ အကြိမ်အရေအတွက်နဲ့ ပြန်စမ်းခြင်း
- Model မအောင်မြင်ရင် deterministic fallback နဲ့ system ကို မညှိုးစေဘဲ ဆက်လုပ်နိုင်စေခြင်း

## သင်ခန်းစာများ

1. Schema-first design နဲ့ Pydantic
2. JSON mode / function calling / structured outputs
3. Validation errors ကို feedback အဖြစ် အသုံးချခြင်း
4. Bounded repair retries
5. Deterministic fallbacks

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံ (function, dict, class, type hints)
- Week 1 မှာ သင်ခဲ့တဲ့ LLM API ခေါ်တဲ့ အခြေခံ အသိများ
- pip နဲ့ package install လုပ်နိုင်စွမ်း (`pip install pydantic`)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

LLM ထုတ်ပေးတဲ့ output ကို database ထဲ သိမ်းရမယ်၊ downstream code က ဖျက်သွင်းရမယ်၊ ဒါမှမဟုတ် အခြား service တွေဆီ ပို့ရမယ်ဆိုရင် — output ဟာ အမြဲတမ်း မှန်ကန်တဲ့ format နဲ့ ရှိဖို့ သေချာစေရမှာ မလိုမရမီ ဖြစ်ပါတယ်။

## ကိုးကား

- Pydantic documentation: https://docs.pydantic.dev/
- OpenAI Structured Outputs guide: https://platform.openai.com/docs/guides/structured-outputs
- Pydantic data validation: https://docs.pydantic.dev/latest/concepts/validation/
