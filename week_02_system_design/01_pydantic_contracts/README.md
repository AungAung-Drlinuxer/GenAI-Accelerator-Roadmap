# Pydantic v2 Contracts for LLM I/O

LLM input/output များကို Pydantic v2 ဖြင့် type-safe contract များအဖြစ် သတ်မှတ်ပြီး structured prompting၊ validation နှင့် serialization ကို လက်တွေ့ကျကျ လေ့လာသည်။

## ဒီ module မှာ ဘာသင်မလဲ

- `BaseModel` ဖြင့် LLM output အတွက် strict data contract များ ရေးဆွဲနည်း
- `Field` constraints (ဥပမာ `min_length`, `ge`, `le`, `pattern`) များဖြင့် input/output ကို ကန့်သတ်နည်း
- `@field_validator` နှင့် `@model_validator` ဖြင့် business logic validation ထည့်သွင်းနည်း
- `model_json_schema()` ကို အသုံးပြုပြီး tool calling / JSON-schema driven prompting အတွက် schema ထုတ်ပေးနည်း
- Nested models ဖြင့် ရှုပ်ထွေးသော JSON structures များကို model ပြုလုပ်နည်း
- `model_dump()` / `model_validate()` ဖြင့် serialization round-trip များ စစ်ဆေးနည်း

## သင်ခန်းစာများ

1. `BaseModel` အခြေခံ — LLM response တစ်ခုအတွက် ပထမဆုံး contract model ရေးသားခြင်း
2. `Field` constraints — type အပြင် တန်ဖိုး boundary များနှင့် description များ ထည့်သွင်းခြင်း
3. Validators — `@field_validator` နှင့် `@model_validator` ဖြင့် custom validation rules ရေးခြင်း
4. JSON Schema output — `model_json_schema()` ဖြင့် tool definitions နှင့s prompt engineering schema များ ထုတ်ခြင်း
5. Nested models — parent-child JSON structures များကို nested `BaseModel` များဖြင့် ကိုယ်စားပြုခြင်း
6. Serialization round-trips — Python object → JSON → Python object ပြန်ရောက်လာစဉ် data integrity စစ်ဆေးခြင်း

## လိုအပ်ချက်များ (Prerequisites)

- Python 3.10+ ၏ basic syntax (dataclasses, type hints, dicts, lists)
- Week 1 ၏ LLM API calling အခြေခံ သင်ခန်းစာများ ပြီးမြောက်ထားရမည်
- JSON format ဖတ်ရှုနိုင်စွမ်း (objects, arrays, nesting)
- Pydantic v2 installed — `pip install pydantic` (version 2.x ဖြစ်ရမည်)

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- LLM ထံမှ လက်ခံရရှိသော JSON output များကို မှန်ကန်ကြောင်း အလိုအလျောက် စစ်ဆေးလိုသောအခါ
- Tool calling / function calling API များအတွက် parameter schema များ ထုတ်ယူလိုသောအခါ
- Prompt တစ်ခုတွင် "ဒီ format အတိအကျ ပြန်ပေး" ဟု တောင်းဆိုလိုသောအခါ — schema ကို prompt ထဲ ထည့်ခြင်းဖြင့် hallucinated structure များ လျှော့ချနိုင်သည်
- Production pipeline များတွင် untrusted LLM output ကို downstream system ထဲ မဝင်မီ ခိုင်မာသော type check လုပ်လိုသောအခါ

## ကိုးကား

- Pydantic v2 Models: https://docs.pydantic.dev/latest/concepts/models/
- Fields & Constraints: https://docs.pydantic.dev/latest/concepts/fields/
- Validators: https://docs.pydantic.dev/latest/concepts/validators/
- JSON Schema: https://docs.pydantic.dev/latest/concepts/json_schema/
- Serialization: https://docs.pydantic.dev/latest/concepts/serialization/
