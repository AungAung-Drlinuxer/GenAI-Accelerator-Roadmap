# Structured Output & Repair Loops — ရှင်းလင်းချက်

## ၁။ Schema-first design

### ဘာကို ဆိုလိုတာလဲ

Schema-first design ဆိုတာ model ကို prompt ရေးပြီးမှ output ဘယ်လိုမှန်မလဲ မမျှော်လင့်ဘဲ၊ **အရင်ဆုံး output ရဲ့ data schema ကို သတ်မှတ်ပြီးမှ** system တစ်ခုလုံးကို တည်ဆောက်တဲ့ နည်းပညာပါ။ Pydantic က Python class တွေနဲ့ schema သတ်မှတ်ပြီး runtime မှာ စစ်ဆေးပေးပါတယ်။

### ဘာကြောင့် လဲ

LLM output က raw string ဖြစ်တတ်လို့ ဘယ် key တွေ ပါမလဲ၊ ဘယ် type ဖြစ်မလဲ ဆိုတာ မသေချာပါဘူး။ Schema ကို code မှာ တစ်နေရာတည်း သတ်မှတ်ထားရင် — (၁) prompt ရေးသူ ရှင်းရှင်းလင်းလင်း သိရပြီး (၂) တခြား code တွေကလည်း အဲဒီ schema အပေါ် အားရစွာ အခြေခံနိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Pydantic BaseModel class တစ်ခု ရေးပြီး field တွေကို type hint နဲ့ သတ်မှတ်ရုံပါပဲ။ `model_validate()` နဲ့ dict ကနေ class instance ဆောက်တဲ့အခါ validation ဖြစ်ပြီး မမှန်ရင် `ValidationError` ပြန်ပေးပါတယ်။

### ဥပမာ

```python
from pydantic import BaseModel

class Product(BaseModel):
    name: str
    price: float
    in_stock: bool

# Valid data passes validation cleanly
item = Product.model_validate({"name": "Coffee", "price": 4.5, "in_stock": True})
print(item.name, item.price, item.in_stock)
# Expected output: Coffee 4.5 True
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production မှာ LLM ထုတ်တဲ့ output ကို တိုက်ရိုက် အသုံးမချခင် schema နဲ့ စစ်ဆေးတာဟာ system တစ်ခုလုံးရဲ့ တုန်မလှုံ့မှု အခြေခံပါ။ မမှန်တဲ့ data တစ်ခု ဆိုက်လို့ system တစ်ခုလုံး ပျက်စေနိုင်လို့ပါ။

## ၂။ JSON mode / function calling / structured outputs

### ဘာကို ဆိုလိုတာလဲ

OpenAI တို့လို LLM platform တွေမှာ model က JSON အပြည့်အစုံ၊ သတ်မှတ်ထားတဲ့ JSON Schema နဲ့ ကိုက်ညီတဲ့ output ထုတ်ပေးဖို့ mode တွေ ရှိပါတယ်။ JSON mode က output က JSON syntax ဖြစ်ရုံ အာမခံပြီး，structured outputs / function calling က schema အတိုင်း ကိုက်အောင် အာမခံပေးပါတယ်။

### ဘာကြောင့် လဲ

Prompt မှာ "JSON ပြန်ပေး" လို့ ရေးထားရုံနဲ့ model က markdown code fence နဲ့ ပတ်ပြီး ပြန်တတ်ပါတယ်။ Platform-level support သုံးရင် parsing လွယ်ပြီး error နှုန်းလည်း သိသိသိသိ လျော့သွားပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

API ရဲ့ response format setting မှာ JSON schema တပ်ပြီး ဖိတ်ခေါ်ရုံပါပဲ။ Python SDK မှာ Pydantic model ကို တိုက်ရိုက် ပေးလို့ရတဲ့ helper တွေလည်း ရှိပါတယ်။ (ဒီ example ဟာ API key ရှိမှ အလုပ်လုပ်ပါမယ်။)

### ဥပမာ

```python
from pydantic import BaseModel
from openai import OpenAI

class Step(BaseModel):
    explanation: str
    output: str

class MathReasoning(BaseModel):
    steps: list[Step]
    final_answer: str

client = OpenAI()

completion = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Solve 8x + 7 = -23"}],
    response_format=MathReasoning,
)
# The SDK validates the reply against MathReasoning automatically
result = completion.choices[0].message.parsed
print(result.final_answer)
# Expected output: x = -3.75  (a string field containing the answer)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

JSON ကို `json.loads()` နဲ့ parse လုပ်ရတာဟာ production pipeline ရဲ့ ပထမဆုံး အာရုံစိုက်စရာ အချက်ပါ။ Platform က schema enforcement ပေးရင် model က output ဘက် အာရုံစိုက်စရာ မလိုတော့ဘဲ ကိုယ့် logic အပေါ် အာရုံစိုက်နိုင်ပါတယ်။

## ၃။ Validation errors as feedback

### ဘာကို ဆိုလိုတာလဲ

Model ထုတ်တဲ့ output က schema နဲ့ မကိုက်ရင် error ကို ဖျောက်ပစ်မှ မဟုတ်ဘဲ — error message အပြည့်အစုံကို model ကို ပြန်ပြပြီး "ဒီပုံစံအတိုင်း ထပ်ရေးပါ" လို့ ခိုင်းတာပါ။

### ဘာကြောင့် လဲ

Error message တွေက model အတွက် တကယ့် အသုံးဝင်တဲ့ ညွှန်ကြားချက်တွေပါ — ဘယ် field မှားလဲ၊ ဘာလိုချင်လဲ ဆိုတာ တိတိကျကျ ပြောပြထားလို့ ဒုတိယအကြိမ်မှာ အောင်မြင်နိုင်机会 များပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

ပထမ ခေါ်တာက parsed response ထုတ်တဲ့ prompt နဲ့ model output တွေကို သမိုင်းမှာ ထည့်ပြီး၊ validation မှားရင် error စာသား ထပ်ဆွဲထည့်ပြီး နောက်တစ်ခေါက် ဖိတ်ခေါ်ပါတယ်။

### ဥပမာ

```python
from pydantic import BaseModel, ValidationError

class Ticket(BaseModel):
    subject: str
    priority: int  # must be 1, 2, or 3 by our business rule

raw = {"subject": "Login broken", "priority": "high"}

try:
    Ticket.model_validate(raw)
except ValidationError as err:
    # Feed this exact error text back into the LLM prompt as feedback
    print(err.errors())
# Expected output: a list of error dicts, e.g. [{'type': 'int_parsing', 'loc': ('priority',), 'msg': ...}]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Error ကို log မှာ ပစ်ထားရုံနဲ့ user ကို ဘာမှ မရပါဘူး။ Error ကို feedback loop ထဲ ထည့်တာက model ရဲ့ မှားယွင်းမှုကို self-correct လုပ်ခွင့်ပေးလိုက်တာပါ။

## ၄။ Bounded repair retries

### ဘာကို ဆိုလိုတာလဲ

Repair loop ကို အကန့်အသတ်နဲ့ လုပ်ခြင်းပါ — ဥပမာ အကြိမ် ၃ ကြိမ်ထက် ပိုမပြန်စမ်းဘဲ၊ လွန်ရင် fallback ကို ကူးပါတယ်။

### ဘာကြောင့် လဲ

Bound မရှိတဲ့ loop က ချိန်၊ ကုန်ကျစရိတ်နဲ့ user experience အားလုံးကို ဖျက်ဆီးနိုင်ပါတယ်။ Model တစ်ခုက အမြဲတမ်း မှားနေရင် loop က အဆုံးမဲ့ လည်နေမှာပါ။

### ဘယ်လို အလုပ်လုပ်လဲ

Retry counter တစ်ခုထားပြီး တကြိမ်မှာ validation မအောင်ရင် error feedback ထည့်ပြီး ထပ်ဖိတ်ရုံပါပဲ။ Counter ပြည့်သွားရင် deterministic fallback ကို ခေါ်ပါတယ်။

### ဥပမာ

```python
from pydantic import BaseModel, ValidationError

class Order(BaseModel):
    order_id: str
    total: float

def repair_loop(get_output, max_retries=3):
    # get_output is a callable simulating an LLM response
    history = []
    for attempt in range(1, max_retries + 1):
        raw = get_output(history)
        history.append(str(raw))
        try:
            return Order.model_validate(raw)
        except ValidationError as err:
            history.append(str(err.errors()))
    return None  # signal: hand over to fallback

print(repair_loop(lambda h: {"order_id": 1, "total": "abc"}))
# Expected output: None  (all attempts failed validation)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Latency budget၊ cost budget နဲ့ graceful degradation ဟာ production AI system တိုင်းမှာ လိုအပ်ပါတယ်။ Bounded retry က "အကောင်းဆုံးအားထုတ်၊ ဒါပေမဲ့ နယ်နိမ့်ရင် ရပ်တန့်" ဆိုတဲ့ စနစ်ကို ဖန်တီးပေးပါတယ်။

## ၅။ Deterministic fallbacks

### ဘာကို ဆိုလိုတာလဲ

Model နဲ့ retry အားလုံး မအောင်မြင်ရင် ခေါ်သုံးတဲ့ သင်္ချာနည်းကျ၊ ရလဒ် ခန့်မှန်းနိုင်တဲ့ code path တစ်ခုပါ။

### ဘာကြောင့် လဲ

System တစ်ခုလုံး လုံးဝ ရပ်မသွားစေဖို့၊ user ကို အနည်းဆုံး အသုံးဝင်တဲ့ အဖြေတစ်ခု ပြန်ပေးနိုင်ဖို့ပါ။ Fallback က တစ်ခါတည်း မှန်ကန်တာ သေချာပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Fallback function တစ်ခု ကြိုရေးထားပြီး repair loop က None ပြန်ရင် ခေါ်ရုံပါပဲ။ Default value၊ template answer၊ ဒါမှမဟုတ် human review queue ဆီ တွန့်ပို့တာတွေ ဖြစ်နိုင်ပါတယ်။

### ဥပမာ

```python
def fallback_order(raw_text):
    # Deterministic path: return a safe, valid default and flag for review
    return Order(order_id="PENDING-REVIEW", total=0.0)

result = repair_loop(lambda h: {"bad": "data"})
final = result if result is not None else fallback_order("irrelevant")
print(final)
# Expected output: order_id='PENDING-REVIEW' total=0.0
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

LLM က outage ဒါမှမဟုတ် ရလဒ်ညံ့ဖြစ်နိုင်လို့ — fallback ရှိနေရင် service ဟာ "AI-powered best case" နဲ့ "guaranteed worst case" ကြားမှာ မည်သည့်အခါမှ မပျက်ပါ။

## အနှစ်ချုပ်

Schema-first design က output ရဲ့ ပုံစံကို အရင် သတ်မှတ်စေပြီး၊ platform ရဲ့ structured outputs / function calling က model ကို အဲဒီပုံစံနဲ့ ထုတ်စေတယ်။ မှားရင် validation error ကို feedback အဖြစ် ပြန်ပေးပြီး bounded retry နဲ့ ထပ်စမ်းတယ်၊ နောက်ဆုံးမှာ deterministic fallback နဲ့ စနစ်ကို ခိုင်မာစေတယ် — ဒါဟာ ယုံကြည်စိတ်ချရတဲ့ AI system design ရဲ့ အသည်းနှလုံးပါ။
