## လေ့ကျင့်ခန်း ၁ — ပထမဆုံး BaseModel တည်ဆောက်ခြင်း

LLM ထုတ်ပေးသော အဖြေတစ်ခုကိုကိုယ်စားပြုရန် `ArticleSummary` ဆိုသည့် Pydantic model တစ်ခုရေးပါ။ Field များမှာ — `title` (str, အနည်းဆုံး စာလုံး ၃ လုံး), `word_count` (int, ၁ ထက်မနည်း), `tags` (str များပါဝင်သော list) တို့ဖြစ်သည်။ Valid input တစ်ခုဖြင့် instance တည်ဆောက်ပြီး `.title` နှင့် `.tags` ကို print လုပ်ပါ။ ထို့နောက် `word_count=0` ဖြင့် ထပ်စမ်းကြည့်ပြီး `ValidationError` ဖမ်းယူ၍ error message ကို print လုပ်ပါ။

```python
from pydantic import BaseModel, Field

class ArticleSummary(BaseModel):
    title: str = Field(min_length=3)
    word_count: int = Field(gt=0)
    tags: list[str] = []
```

**Hints:** `Field(min_length=3)` သည် str အတွက်လည်း `Field(gt=0)` သည် int အတွက်လည်း ကန့်သတ်ချက် သတ်မှတ်ပေးသည်။ Error ကို `try/except ValidationError` ဖြင့်ဖမ်းပါ။
**Expected behavior:** Valid input တွင် တိကျသော output ရပြီး၊ `word_count=0` တွင် `ValidationError` ဖြင့် error message ပေါ်သည်။

## လေ့ကျင့်ခန်း ၂ — Field constraints နှင့် default values

`LLMResponse` model ရေးပါ — `model_name: str` (default `"gpt-4o-mini"`)၊ `latency_ms: float` (0 ထက်ကြီးရမည်၊ default `1.0`)၊ `tokens: int` (0 ထက်ကြီးရမည်)၊ `temperature: float` (0.0 မှ 2.0 အတွင်း၊ default `0.7`)။ Out-of-range တန်ဖိုးများကို စမ်းသပ်၍ ဘယ် field က ဘယ် error ပေးသည်ကို စစ်ဆေးပါ။

```python
class LLMResponse(BaseModel):
    model_name: str = "gpt-4o-mini"
    latency_ms: float = Field(default=1.0, gt=0)
    tokens: int = Field(gt=0)
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
```

**Hints:** `ge` (greater-or-equal) နှင့် `gt` (strictly greater) ခြားနားချက်ကို သတိပြုပါ။ `temperature=2.5` ကို စမ်းကြည့်ပါ။
**Expected behavior:** `temperature=2.5` ကို ငြင်းပယ်ပြီး `temperature=2.0` ကို လက်ခံသည်။

## လေ့ကျင့်ခန်း ၃ — field_validator ဖြင့် စင်ကြယ်စေခြင်း

`PromptRequest` model ရေးပါ — `prompt: str` field ပါဝင်သည်။ `@field_validator("prompt")` ဖြင့် — (a) prompt ၏ အစ/အဆုံး whitespace များကို ဖျက်ပါ (`strip()`)၊ (b) prompt အလွတ်ဖြစ်နေလျှင် `ValueError` တင်ပါ။ Empty prompt နှင့် space-only prompt နှစ်မျိုးစလုံး စမ်းပါ။

```python
from pydantic import field_validator

class PromptRequest(BaseModel):
    prompt: str

    @field_validator("prompt")
    @classmethod
    def check_prompt(cls, v: str) -> str:
        v = v.strip()
        if not v:
            raise ValueError("prompt must not be empty")
        return v
```

**Hints:** validator အတွင်း တန်ဖိုးကို `return` ပြန်မှသာ ပြင်ဆင်ထားသော တန်ဖိုး သိမ်းဆည်းမည်ဖြစ်သည်။ `@classmethod` ကိုမမေ့ပါနှင့်။
**Expected behavior:** `"  hello  "` သည် `"hello"` ဖြစ်သွားပြီး `"   "` သည် `ValidationError` ပေးသည်။

## လေ့ကျင့်ခန်း ၄ — Nested models ဖြင့် structured output

Tool call တစ်ခု၏ output ကို ကိုယ်စားပြုရန် nested model များရေးပါ — `Argument` (field: `name: str`, `value: str`) နှင့် `ToolCall` (field: `tool_name: str`, `arguments: list[Argument]`, `confidence: float` 0.0–1.0)။ Argument နှစ်ခုပါသော `ToolCall` dict တစ်ခုကို `ToolCall(**data)` ဖြင့် parse ပြီး `.arguments[0].value` ကို print လုပ်ပါ။

```python
class Argument(BaseModel):
    name: str
    value: str

class ToolCall(BaseModel):
    tool_name: str
    arguments: list[Argument]
    confidence: float = Field(ge=0.0, le=1.0)
```

**Hints:** Nested dict/list များကို Pydantic က အလိုအလျောက် `Argument` model များအဖြစ် ပြောင်းပေးသည် — လက်ဖြင့်ပြောင်းစရာမလိုပါ။
**Expected behavior:** `data["arguments"][0]` က plain dict ဖြစ်စေကာမှ အလိုအလျောက် `Argument` instance ဖြစ်သွားပြီး တန်ဖိုးများ မှန်ကန်စွာ ဖတ်ရှိနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — model_json_schema ဖြင့် schema-driven prompting

လေ့ကျင့်ခန်း ၄ ၏ `ToolCall` model အတွက် `ToolCall.model_json_schema()` ကိုခေါ်၍ `json.dumps(schema, indent=2)` ဖြင့် print လုပ်ပါ။ ထို့နောက် LLM ကို tool-call JSON ထုတ်စေရန် ညွှန်ကြားရေး prompt template တစ်ခု ရေးပါ — prompt ထဲတွင် schema JSON ကို တိုက်ရိုက်ထည့်ပြီး “ဤ schema နှင့်ကိုက်ညီသော JSON သာ ထုတ်ပေးပါ” ဟုပါဝင်စေပါ။

```python
import json

schema = ToolCall.model_json_schema()
prompt = (
    "You are a function-calling assistant. "
    "Respond ONLY with JSON matching this schema:\n"
    f"{json.dumps(schema, indent=2)}"
)
print(prompt)
```

**Hints:** `model_json_schema()` သည် JSON Schema draft လိုက်သော dict ပြန်ပေးသည်။ `$defs` အောက်တွင် nested `Argument` schema ကို တွေ့ရမည်။
**Expected behavior:** Prompt ထဲတွင် field name, type နှင့် constraints များ ပါဝင်သော schema JSON အပြည့်အစုံ ပေါ်လာသည်။

## လေ့ကျင့်ခန်း ၆ — Serialization round-trip

`ToolCall` instance တစ်ခုကို — (a) `model_dump()` ဖြင့် Python dict ပြောင်းပါ၊ (b) `model_dump_json()` ဖြင့s JSON string ပြောင်းပါ၊ (c) ထို JSON string မှ `ToolCall.model_validate_json()` ဖြင့် ပြန်တည်ဆောက်ပြီး မူလ instance နှင့် `==` ဖြင့် တူမတူစစ်ပါ။ ထို့နောက် JSON string ထဲမှ field တစ်ခုကို ပျက်စေ၍ (ဥပမာ `confidence` ကို ဖျက်ပါ) validate လုပ်ကြည့်ပါ။

```python
tc = ToolCall(
    tool_name="get_weather",
    arguments=[Argument(name="city", value="Yangon")],
    confidence=0.92,
)
as_dict = tc.model_dump()
as_json = tc.model_dump_json()
tc2 = ToolCall.model_validate_json(as_json)
print(tc == tc2)  # True
```

**Hints:** `model_dump()` သည် Python object များပေးပြီး `model_dump_json()` သည် JSON string ပေးသည်။ ပျက်စေသော JSON ကို validate လုပ်လျှင် missing-field error ရမည်။
**Expected behavior:** Round-trip ပြီးပါက `tc == tc2` သည် `True` ဖြစ်ပြီး၊ `confidence` ပျက်နေသော JSON သည် `ValidationError` ပေးသည်။
