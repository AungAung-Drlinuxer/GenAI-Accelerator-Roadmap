# Pydantic v2 Contracts for LLM I/O

AI system တစ်ခုမှာ LLM ကနေ ထွက်လာတဲ့ output ကို ယုံမှားစရာမရှိဘဲ လက်ခံဖို့၊ tool တွေကို ခေါ်ဖို့ schema တစ်ခု လိုအပ်ပါတယ်။ Pydantic v2 က ဒီ schema တွေကို Python class တွေနဲ့ ရေးပြီး JSON schema အဖြစ် အလိုအလျောက် ထုတ်ပေးနိုင်ပါတယ်။ ဒီ module မှာ BaseModel, Field constraints, validators, model_json_schema, nested models နဲ့ serialization round-trips တို့ကို လေ့လာပါမယ်။

---

## BaseModel နဲ့ Field Constraints

### ဘာကို ဆိုလိုတာလဲ
`BaseModel` က Pydantic ရဲ့ base class ဖြစ်ပြီး data class တစ်ခုကို type annotation တွေနဲ့ သတ်မှတ်ထားရင် input data ကို အလိုအလျောက် validate လုပ်ပေးပါတယ်။ `Field()` function ကတော့ field တစ်ခုချင်းစီမှာ extra rule (default value, length limit, numeric range, description) တွေ ထည့်ဖို့ အသုံးပြုပါတယ်။

### ဘာကြောင့် လဲ
LLM output က အမြဲစာသွားတာမဟုတ်ပါဘူး — JSON ပုံစံမှားနိုင်သလို field တစ်ခု လွတ်နေနိုင်ပါတယ်။ BaseModel ကို သုံးရင် မှားတဲ့ data ကို program run ဆက်မခင့် စစ်ပေးလို့ production system မှာ crash တွေကို ကြိုတင်ကာကွယ်နိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Class ထဲမှာ field တွေကို `name: str` စတဲ့ annotation နဲ့ ရေးရုံနဲ့ Pydantic က runtime မှာ type စစ်ပေးပါတယ်။ `Field(...)` ထဲမှာ `min_length`, `max_length`, `ge`, `le`, `default` စတဲ့ constraint တွေထည့်နိုင်ပြီး၊ `description` ထည့်ရင် နောက်ပိုင်း JSON schema ထဲ ပါဝင်သွားမှာ ဖြစ်ပါတယ်။

### ဥပမာ

```python
from pydantic import BaseModel, Field, ValidationError

class Answer(BaseModel):
    topic: str = Field(..., min_length=2, description="Subject of the answer")
    confidence: float = Field(..., ge=0.0, le=1.0, description="Model confidence score")
    source_count: int = Field(default=0, ge=0)

# Valid data: types and constraints are checked automatically
ok = Answer(topic="pydantic", confidence=0.9)
print(ok.topic, ok.confidence, ok.source_count)

# Invalid data: confidence exceeds the upper bound
try:
    bad = Answer(topic="x", confidence=1.5)
except ValidationError as e:
    print("Validation failed:", e.errors()[0]["type"])
# Expected output: pydantic 0.9 0
# Expected output: Validation failed: less_than_equal
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
LLM ရဲ့ output ကို `Answer(**json.loads(text))` ဆိုတဲ့ pattern နဲ့ တိုက်ရိုက် model ထဲသွင်းရင် format မှားတာ၊ value ပြင်ပအထိ ချက်ချင်း သိနိုင်ပါတယ်။ Error ဖမ်းပြီး retry လုပ်တာ၊ fallback သုံးတာတွေကို တစ်နေရာတည်းကနေ စနစ်တကျ လုပ်နိုင်လို့ LLM application တွေရဲ့ reliability ကို သိသိသာသာ တိုးစေပါတယ်။

---

## Validators

### ဘာကို ဆိုလိုတာလဲ
Validator က field တစ်ခု (သို့) တစ်ခုထက်ပိုတဲ့ value တွေပေါ်မှာ custom check (သို့) transform logic ထည့်ခွင့်ပေးတဲ့ function ဖြစ်ပါတယ်။ Pydantic v2 မှာ `@field_validator` နဲ့ `@model_validator` decorator တွေနဲ့ ရေးပါတယ်။

### ဘာကြောင့် လဲ
Type စစ်တာနဲ့ range စစ်တာက business logic အားလုံးကို မဖမ်းနိုင်ပါဘူး — ဥပမာ `role` field က "user" ဒါမှမဟုတ် "assistant" ဘဲဖြစ်ရမယ်၊ ဒါမှမဟုတ် `start_date` က `end_date` ထက် စောရမယ် စသဖြင့်။ ဒီလို rule တွေကို validator နဲ့ model ထဲမှာပဲ တစ်နေရာတည်း စုပေးနိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
`@field_validator("field_name")` က field တစ်ခုစီအတွက် ခေါ်ပြီး၊ input value ကို argument အနေနဲ့ ရပါတယ် — ပြင်ပြီး return လုပ်ရင် အဲဒါက အသုံးဝင်တဲ့ value ဖြစ်သွားပါတယ်။ `@model_validator(mode="after")` ကတော့ field အားလုံး validate ဖြစ်ပြီးနောက် ခေါ်ပြီး field တွေကြားမှာ ဆက်နွယ်တဲ့ rule စစ်ဖို့ အသုံးပြုပါတယ်။

### ဥပမာ

```python
from pydantic import BaseModel, field_validator, model_validator

class ChatTurn(BaseModel):
    role: str
    content: str

    @field_validator("role")
    @classmethod
    def check_role(cls, value: str) -> str:
        # Normalize to lowercase and enforce allowed values
        value = value.lower()
        if value not in ("user", "assistant", "system"):
            raise ValueError("role must be user, assistant or system")
        return value

    @model_validator(mode="after")
    def check_content(self) -> "ChatTurn":
        # Cross-field rule: system turns must not be empty
        if self.role == "system" and not self.content.strip():
            raise ValueError("system content cannot be empty")
        return self

turn = ChatTurn(role="ASSISTANT", content="Hello!")
print(turn.role)
# Expected output: assistant
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
LLM က role name ကို အမြဲ မှန်ကန်စွာ မထုတ်ပေးပါဘူး — "Assistant" လို့ ကြီးစာလုံးနဲ့ ရေးတာမျိုး ရှိနိုင်ပါတယ်။ Validator က normalize လုပ်ပေးလို့ နောက်ပိုင်း processing logic တွေက အခြေအနေရှင်းရှင်းလင်းလင်းနဲ့ အလုပ်လုပ်နိုင်ပြီး၊ prompt-driven output တွေကို contract တစ်ခုအဖြစ် ယုံကြည်စိတ်ချစွာ သုံးနိုင်ပါတယ်။

---

## model_json_schema နဲ့ JSON Schema-Driven Prompting

### ဘာကို ဆိုလိုတာလဲ
`model_json_schema()` method က Pydantic model တစ်ခုရဲ့ ဖွဲ့စည်းပုံကို JSON Schema အဖြစ် ထုတ်ပေးပါတယ်။ ဒီ schema ကို LLM ဆီ prompt ထဲ ထည့်ပေးရင် ဒါမှမဟုတ် tool calling API မှာ တိုက်ရိုက် ဖြတ်ပေးရင် model ရဲ့ output ကို ကျုံ့ပေးနိုင်ပါတယ်။

### ဘာကြောင့် လဲ
"JSON နဲ့ ပြန်ပေးပါ" လို့ prompt မှာ ရေးထားတာထက် schema တစ်ခု တိတ်တိတ်ထည့်ပေးတာက ပိုတိကျပါတယ် — field နာမည်တွေ၊ type တွေ၊ description တွေ၊ required ဖြစ်မဖြစ်တွေ အကုန်ပါလို့ model က မှန်ကန်တဲ့ structure ထုတ်ဖို့ အခွင့်အလမ်း ပိုများပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Field တွေရေးထားတဲ့ class ကနေ `.model_json_schema()` ကို ခေါ်ရင် dict တစ်ချောက် ရပါတယ် — `properties` ထဲမှာ field တွေ၊ `required` ထဲမှာ မဖြစ်မနေ လိုအပ်တဲ့ field တွေ၊ သောခါ constraint တွေပါ `json.dumps` နဲ့ string ပြောင်းပြီး prompt ထဲ သွားနိုင်ပါတယ်။ ထိုးချင်းပဲ OpenAI-style tool definitions မှာ `parameters` အဖြစ် တိုက်ရိုက် သုံးနိုင်ပါတယ်။

### ဥပမာ

```python
import json
from pydantic import BaseModel, Field

class ExtractedEntity(BaseModel):
    name: str = Field(..., description="Entity name exactly as it appears")
    category: str = Field(..., description="One of: person, org, location")
    mentions: int = Field(default=1, ge=1, description="Times it appears")

schema = ExtractedEntity.model_json_schema()
print(json.dumps(schema, indent=2)[:300])

# This schema can be embedded in a prompt or tool definition
prompt = f"Extract entities matching this JSON schema:\n{json.dumps(schema)}"
# Expected output: (JSON schema dict with properties: name, category, mentions)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Schema ကို code တစ်နေရာတည်းကနေ သိမ်းထားရင် input validation နဲ့ prompting နှစ်ခုစလုံး အတူတူ schema တစ်ခုပဲ အသုံးပြုပါတယ် — schema နဲ့ validator က လွဲပြောင်းမနေတော့ပါဘူး။ Model က field အသစ်ထပ်ထည့်တာ၊ နာမည်ပြောင်းတာမျိုး ဖြစ်လာရင် runtime validation က ချက်ချင်း ဖမ်းပေးလို့ schema-driven design က production LLM system တွေမှာ အဓိက safety net တစ်ခု ဖြစ်ပါတယ်။

---

## Nested Models နဲ့ Serialization Round-Trips

### ဘာကို ဆိုလိုတာလဲ
Nested model က model တစ်ခု ထဲမှာ model အခြားတစ်ခုကို field အဖြစ် ထည့်သွင်းတာပါ။ Serialization round-trip က Python object → JSON → Python object ပြန်ဆွဲတဲ့ ခရီးကို ဆိုလိုပြီး၊ ပြန်လာတဲ့ object က မူလ object နဲ့ တူညီရမှာ ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ
LLM ရဲ့ output တွေက များသောအားဖြင့် flat structure မဟုတ်ပါဘူး — message ထဲမှာ tool call list ရှိမယ်၊ tool call တစ်ခုမှာ arguments ရှိမယ် စသဖြင့်။ Nested models နဲ့ ဒီ hierarchy တွေကို type-safe ဖြစ်စွာ ကိုယ်စားပြုနိုင်ပြီး၊ round-trip အလုပ်လုပ်တာက storage/cache နဲ့ API boundary တွေမှာ ယုံကြည်နိုင်မှုပေးပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
`model_dump()` က Python dict ထုတ်ပေးပြီး `model_dump_json()` က JSON string ထုတ်ပေးပါတယ်။ ပြန်ဖွင့်ဖို့အတွက် `Model.model_validate(dict)` နဲ့ `Model.model_validate_json(json_string)` ကို သုံးပါတယ် — validate ချိန်မှာ nested constraint တွေအားလုံး ထပ်စစ်ပါတယ်။

### ဥပမာ

```python
import json
from pydantic import BaseModel, Field

class ToolCall(BaseModel):
    tool_name: str
    arguments: dict

class LLMResponse(BaseModel):
    text: str
    tool_calls: list[ToolCall] = Field(default_factory=list)

# Step 1: Python object -> JSON string
original = LLMResponse(
    text="Looking it up.",
    tool_calls=[ToolCall(tool_name="search", arguments={"query": "pydantic v2"})],
)
js = original.model_dump_json()
print(js)

# Step 2: JSON string -> Python object (full re-validation)
restored = LLMResponse.model_validate_json(js)
print(restored.tool_calls[0].tool_name)

# Step 3: verify the round-trip preserved the data
print(restored.model_dump() == original.model_dump())
# Expected output: {"text":"Looking it up.","tool_calls":[{"tool_name":"search","arguments":{"query":"pydantic v2"}}]}
# Expected output: search
# Expected output: True
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
LLM ကထုတ်တဲ့ tool call တွေကို nested model နဲ့ parse လုပ်တဲ့အခါ argument တွေထဲက required field တစ်ခု လွတ်နေရင် ချက်ချင်း error ရပါတယ်။ တကယ်လက်ခံလိုက်ရရင် tool execution layer မှာ undefined behavior ဖြစ်တတ်ပါတယ်။ Round-trip ကလည်း response တွေကို Redis/cache ထဲ သိမ်းပြီး ပြန်ယူတဲ့ အခါမှုနားမရှိကြောင်း သက်သေပြနိုင်လို့ production pipeline တစ်ခုစီမှာ စစ်သင့်တဲ့ invariant တစ်ခု ဖြစ်ပါတယ်။

---

## အနှစ်ချုပ်

- `BaseModel` က LLM output တွေကို runtime မှာ type-safe စွာ စစ်ပေးတဲ့ contract layer ဖြစ်ပြီး `Field()` နဲ့ length/range/default/description တွေ ထည့်နိုင်ပါတယ်။
- `@field_validator` က field တစ်ခုချင်းစီကို normalize/check လုပ်ပြီး `@model_validator` က field တွေကြားမှာ ဆက်နွယ်တဲ့ rule တွေကို စစ်ပေးပါတယ်။
- `model_json_schema()` ထုတ်တဲ့ JSON Schema ကို prompt ထဲဒါမှမဟုတ် tool definition ထဲ ထည့်ပြီး LLM output ကို တိကျစွာ ကျုံ့နိုင်ပါတယ်။
- Nested models က hierarchical output (message → tool_calls → arguments) တွေကို ကိုယ်စားပြုနိုင်ပြီး serialization round-trip (`model_dump_json()` / `model_validate_json()`) က အချက်အလက် မစွန့်စားဘဲ သိမ်း/ဖွင့်နိုင်ကြောင်း သက်သေပြပါတယ်။
- Schema နဲ့ validator ကို တစ်နေရာတည်းကနေ သုံးခြင်းက prompting နဲ့ validation ကို တစ်ပြားတည်းတည်ရှိစေပြီး — production LLM system ရဲ့ အခြေခံဆုံး ယုံကြည်နိုင်မှုပေးပါတယ်။
