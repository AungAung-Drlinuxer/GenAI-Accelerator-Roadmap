# အဖြေများ — Structured Output & Repair Loops

## လေ့ကျင့်ခန်း ၁ — Pydantic schema ဆောက်ခြင်း

```python
from pydantic import BaseModel, ValidationError

class Movie(BaseModel):
    title: str
    year: int
    rating: float

good = {"title": "Parasite", "year": 2019, "rating": 8.6}
bad = {"title": "Unknown", "year": "not-a-number", "rating": 7.0}

movie = Movie.model_validate(good)
print("OK:", movie.title, movie.year)
# Expected output: OK: Parasite 2019

try:
    Movie.model_validate(bad)
except ValidationError as err:
    print(err.errors())
# Expected output: [{'type': 'int_parsing', 'loc': ('year',), 'msg': ..., ...}]
```

**အဓိကအယူအဆ** — Validation မှားတဲ့ field၊ အကြောင်းပြချက်၊ တည်နေရာ အားလုံးကို error ထဲကနေ တိတိကျကျ သိနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Nested schema နဲ့ field constraints

```python
from pydantic import BaseModel, Field, ValidationError

class Nutrition(BaseModel):
    protein: float
    carbs: float

class Recipe(BaseModel):
    name: str
    ingredients: list[str]
    rating: float = Field(ge=0.0, le=5.0)
    nutrition: Nutrition

try:
    Recipe.model_validate({
        "name": "Fried Rice",
        "ingredients": ["rice", "egg"],
        "rating": 9.5,  # out of allowed range
        "nutrition": {"protein": 12.0, "carbs": 60.0},
    })
except ValidationError as err:
    for e in err.errors():
        print(e["loc"], e["msg"])
# Expected output: ('rating',) Input should be less than or equal to 5
```

**အဓိကအယူအဆ** — Business rule တွေကို schema ထဲက itself constraint အဖြစ် ထည့်ထားရင် model ရဲ့ output ကို စစ်တဲ့အခါ အလိုအလျောက် စစ်ပေးပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Simulated LLM JSON parsing

```python
import json
from pydantic import BaseModel, ValidationError

class Person(BaseModel):
    name: str
    age: int

def parse_llm_output(text: str):
    # Step 1: parse raw JSON syntax; Step 2: validate against schema
    try:
        data = json.loads(text)
    except json.JSONDecodeError as e:
        print("JSON syntax error:", e.msg)
        return None
    try:
        return Person.model_validate(data)
    except ValidationError as e:
        print("Validation error:", e.errors())
        return None

ok_text = '{"name": "Aung", "age": 30}'
syntax_bad = '{"name": "Aung", "age": }'   # invalid JSON syntax
schema_bad = '{"name": "Aung", "age": "old"}'  # valid JSON, wrong type

parse_llm_output(ok_text)
parse_llm_output(syntax_bad)
parse_llm_output(schema_bad)
# Expected output:
# Person instance created (printed implicitly as None checks)
# JSON syntax error: Expecting value
# Validation error: [{'type': 'int_parsing', 'loc': ('age',), ...}]
```

**အဓိကအယူအဆ** — JSON syntax error နဲ့ schema validation error ဟာ အလွှာစွံ ခွဲခြားနိုင်ဖို့ အရေးကြီးပါတယ်၊ ပြင်ဆင်ပုံချင်း မတူလို့ပါ။

## လေ့ကျင့်ခန်း ၄ — Error ကို feedback prompt အဖြစ် ပြောင်းခြင်း

```python
def build_repair_prompt(original_prompt: str, raw_output: str, error_text: str) -> str:
    # Include the original ask, the failed attempt, and the exact errors
    return (
        "Your previous answer failed schema validation. Fix it.\n\n"
        f"Original task:\n{original_prompt}\n\n"
        f"Your previous output:\n{raw_output}\n\n"
        f"Validation errors (fix all of these):\n{error_text}\n\n"
        "Return only corrected JSON."
    )

prompt = build_repair_prompt(
    "List one movie with title, year, rating",
    '{"title": "Up", "year": "two-thousand-nine"}',
    "[{'loc': ('year',), 'msg': 'Input should be a valid integer'}]",
)
print("Validation errors" in prompt, "Previous" not in prompt)
# Expected output: True True
```

**အဓိကအယူအဆ** — Error အပြည့်အစုံကို quote လုပ်ပြပါမှ model က ဘာကို ပြင်ရမလဲ ဆိုတာ တိကျစွာ နားလည်နိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၅ — Bounded repair loop

```python
from pydantic import BaseModel, ValidationError

class Answer(BaseModel):
    answer: int

def repair_loop(llm_fn, max_retries=3):
    feedback = ""
    for attempt in range(1, max_retries + 1):
        raw = llm_fn(feedback)  # feedback from the previous failure
        try:
            return Answer.model_validate(raw), attempt
        except ValidationError as err:
            feedback = str(err.errors())
    return None, max_retries

attempts = {"count": 0}
def fake_llm(feedback):
    # Fail twice, then succeed on the third call
    attempts["count"] += 1
    if attempts["count"] < 3:
        return {"answer": "soon"}  # wrong type on purpose
    return {"answer": 42}

result, n = repair_loop(fake_llm)
print(result, "on attempt", n)
# Expected output: answer=42 on attempt 3
```

**အဓိကအယူအဆ** — Retry ကို ကန့်သတ်ထားမှသာ ကုန်ကျစရိတ်နဲ့ latency ကို ခန့်မှန်းနိုင်တဲ့ ယုံကြည်စိတ်ချရတဲ့ system ဖြစ်လာပါတယ်။

## လေ့ကျင့်ခန်း ၆ — Deterministic fallback

```python
from pydantic import BaseModel, ValidationError

class Report(BaseModel):
    title: str
    score: int

review_queue = []  # items needing human review

def fallback():
    # Always-valid default; flagged for human review
    return Report(title="UNPARSED-REPORT", score=0)

def repair_or_fallback(llm_fn, max_retries=3):
    feedback = ""
    for _ in range(max_retries):
        try:
            return Report.model_validate(llm_fn(feedback))
        except ValidationError as err:
            feedback = str(err.errors())
    result = fallback()
    review_queue.append(result)  # queue it so a human can check later
    return result

# Case 1: LLM always fails -> fallback is used
always_bad = lambda f: {"title": 123}
r1 = repair_or_fallback(always_bad)
print(r1, "queued:", len(review_queue))
# Expected output: title='UNPARSED-REPORT' score=0 queued: 1

# Case 2: LLM succeeds after one retry -> LLM result returned
state = {"n": 0}
def sometimes_bad(f):
    state["n"] += 1
    return {"score": "high"} if state["n"] == 1 else {"title": "Weekly", "score": 9}
r2 = repair_or_fallback(sometimes_bad)
print(r2, "queued:", len(review_queue))
# Expected output: title='Weekly' score=9 queued: 1
```

**အဓိကအယူအဆ** — Fallback ဟာ model မှားတဲ့အခါ system တစ်ခုလုံး မတိမ်းမပျောက်စေဘဲ အနည်းဆုံး လုပ်ငန်းဆက်နိုင်အောင် အာမခံပေးတဲ့ အသေးစားအာမခံချက်ပါ။
