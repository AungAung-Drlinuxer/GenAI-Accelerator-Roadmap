## လေ့ကျင့်ခန်း ၁ — BaseModel အခြေခံ

Pydantic v2 တွင် `BaseModel` ကိ� ဆက်စပ်ခြင်းဖြင့် data ဖွဲ့စည်းပုံကို class တစ်ခုအဖြစ် သတ်မှတ်နိုင်သည်။ Field type များကို Python type hints ဖြင့် ကြေညာပြီး validation နှင့် parsing ကို အလိုအလျောက် လုပ်ဆောင်ပေးသည်။

```python
from pydantic import BaseModel, ValidationError

# Define a simple model for an LLM response
class Answer(BaseModel):
    question_id: int
    answer_text: str
    confidence: float

# Valid input: strings are coerced to the declared types
data = {"question_id": "42", "answer_text": "Yangon", "confidence": "0.9"}
obj = Answer.model_validate(data)
print(obj)  # question_id=42 answer_text='Yangon' confidence=0.9

# Invalid input raises a clear error
try:
    Answer(question_id="not_a_number", answer_text="x", confidence=0.5)
except ValidationError as e:
    print(e.error_count(), "validation error(s)")
```

**အဓိကအယူအဆ** — Pydantic BaseModel သည် type hints အပေါ် အခြေခံ၍ input data ကို အလိုအလျောက် စစ်ဆေးပြီး မှားယွင်းပါက `ValidationError` ပေးသဖြင့် LLM output ကို စိတ်ချစွာ လက်ခံနိုင်သည်။

## လေ့ကျင့်ခန်း ၂ — Field constraints

`Field` ဖြင့် default value၊ length နှင့် numeric နယ်နိမိတ်များ သတ်မှတ်နိုင်ပြီး LLM ထံမှ မလိုလားအပ်သော တုံ့ပြန်ချက်များကို စစ်ထုတ်နိုင်သည်။

```python
from pydantic import BaseModel, Field, ValidationError

class ToolCall(BaseModel):
    # Name must be 1-40 characters, with a human-readable description
    name: str = Field(..., min_length=1, max_length=40,
                      description="Exact tool name to invoke")
    # Value must be between 0 and 1, with a default
    score: float = Field(default=0.0, ge=0.0, le=1.0)
    # Optional field with a default
    args: dict = Field(default_factory=dict)

ok = ToolCall(name="search", score=0.75)
print(ok.name, ok.score, ok.args)

try:
    ToolCall(name="", score=1.5)
except ValidationError as e:
    # Each error includes location, type, and message
    for err in e.errors():
        print(err["loc"], err["type"])
```

**အဓိကအယူအဆ** — Field constraints များသည် LLM output တွင် ခွင့်ပြုထားသော တန်ဖိုးများကို ကန့်သတ်ပေးခြင်းဖြင့် model ထုတ်ပေးသည့် data ကို ပိုမိုတိကျစွာ ထိန်းချုပ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — Validators

`@field_validator` ဖြင့် field တစ်ခုချင်းစီ၏ တန်ဖိုးကို စစ်ဆေးပြင်ဆင်နိုင်ပြီး `@model_validator` ဖြင့် field များစွာကြားရှိ ဆက်နွယ်မှုကို စစ်ဆေးနိုင်သည်။

```python
from pydantic import BaseModel, field_validator, model_validator

class ExtractedEntity(BaseModel):
    text: str
    label: str
    start: int
    end: int

    # Normalize text: strip and collapse whitespace
    @field_validator("text")
    @classmethod
    def clean_text(cls, v: str) -> str:
        return " ".join(v.split())

    # Check that the span is valid across fields
    @model_validator(mode="after")
    def check_span(self) -> "ExtractedEntity":
        if not (0 <= self.start < self.end <= len(self.text)):
            raise ValueError("invalid character span")
        return self

entity = ExtractedEntity(text="  Yangon  City ", label="LOC",
                        start=0, end=6)
print(repr(entity.text))  # 'Yangon City'
print(entity.text[entity.start:entity.end])  # 'Yangon'
```

**အဓိကအယူအဆ** — Validators များသည် type စစ်ဆေးမှုထက် ပိုမိုနက်နဲသော စည်းမျဉ်းများ (စာသားရှင်းလင်းခြင်း၊ span တည်မြဲမှုစသည်) ကို ကိုယ်တိုင်ရေးသားစစ်ဆေးနိုင်စေသည်။

## လေ့ကျင့်ခန်း ၄ — Nested models

Model တစ်ခုအတွင်း အခြား model များကို field အဖြစ် ထည့်သွင်းခြင်းဖြင့် ရှုပ်ထွေးသော JSON output ဖွဲ့စည်းပုံများကို ခြောက်သွေ့စွာ ဖော်ပြနိုင်သည်။

```python
from pydantic import BaseModel, Field
from typing import List

class Step(BaseModel):
    action: str
    detail: str

class Plan(BaseModel):
    goal: str
    steps: List[Step] = Field(default_factory=list)

raw = {
    "goal": "Deploy the model",
    "steps": [
        {"action": "test", "detail": "Run unit tests"},
        {"action": "build", "detail": "Build Docker image"},
    ],
}

plan = Plan.model_validate(raw)
# Access nested data with normal attribute access
for step in plan.steps:
    print(step.action, "->", step.detail)
```

**အဓိကအယူအဆ** — Nested models ကို အသုံးပြုခြင်းဖြင့် LLM ထုတ်ပေးသော အဆင့်များစွာရှိ JSON structure ကို Python object များအဖြစ် အလွယ်တကူ ပြောင်းလဲ အသုံးချနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Serialization round-trips

`model_dump_json()` ဖြင့် JSON string ထုတ်ပေးပြီး `model_validate_json()` ဖြင့် ပြန်လည် parse လုပ်နိုင်သည်။ ဤ round-trip သည် data မဆုံးရှုံးစေဘဲ ကျန်ရှိကြောင်း စစ်ဆေးရလွယ်ကူသည်။

```python
from pydantic import BaseModel
from typing import List

class Citation(BaseModel):
    source: str
    quote: str

class Report(BaseModel):
    title: str
    citations: List[Citation]

report = Report(title="Summary",
                citations=[Citation(source="doc1.md", quote="key fact")])

# Serialize to a JSON string
json_str = report.model_dump_json()
print(json_str)

# Parse back from the JSON string
round_tripped = Report.model_validate_json(json_str)
print(round_tripped == report)  # True

# model_dump() gives a plain Python dict
as_dict = report.model_dump()
print(type(as_dict), as_dict["title"])
```

**အဓိကအယူအဆ** — JSON ထုတ်ခြင်းနှင့် ပြန် parse ခြင်း round-trip သည် LLM system တစ်ခု၏ input/output pipeline တွင် data စိတ်ချမှုရှိကြောင်း အလိုအလျောက် စစ်ဆေးပေးသည်။

## လေ့ကျင့်ခန်း ၆ — model_json_schema ဖြင့် tool/JSON-schema prompting

Pydantic model တစ်ခုမှ JSON Schema ထုတ်ယူပြီး LLM ထံမှ တိကျသော format ဖြင့် တုံ့ပြန်စေရန် prompt တွင် ထည့်သွင်းနိုင်သည်။

```python
import json
from pydantic import BaseModel, Field
from typing import List

class WeatherQuery(BaseModel):
    city: str = Field(..., description="City name in English")
    unit: str = Field(default="celsius", description="celsius or fahrenheit")
    days: int = Field(default=1, ge=1, le=7)

# Generate the JSON Schema for this model
schema = WeatherQuery.model_json_schema()
print(json.dumps(schema, indent=2))

# Use the schema inside a prompt for structured output
prompt = (
    "Extract a weather query from the user message. "
    "Respond ONLY with JSON matching this schema:\n"
    + json.dumps(schema)
)
print(prompt)
```

**အဓိကအယူအဆ** — `model_json_schema()` ထုတ်ပေးသော schema ကို prompt တွင် တိုက်ရိုက်ထည့်သွင်းခြင်းဖြင့် LLM ထံမှ တိကျသော structured JSON တုံ့ပြန်ချက်များ ရယူနိုင်ပြီး တုံ့ပြန်ချက်ကို ထို model ဖြင့်ပင် ပြန်လည် validate လုပ်နိုင်သည်။
