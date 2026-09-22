# လေ့ကျင့်ခန်းများ — Structured Output & Repair Loops

## လေ့ကျင့်ခန်း ၁ — Pydantic schema ဆောက်ခြင်း

`Movie` ဆိုတဲ့ Pydantic model ဆောက်ပါ — `title: str`, `year: int`, `rating: float` တွေ ပါရမယ်။ Dict နှစ်ခု (တစ်ခု မှန်၊ တစ်ခု မှား) ကို `model_validate()` နဲ့ စစ်ပြီး မှားတဲ့အတွက် ဖမ်းယူထားတဲ့ error ရဲ့ `.errors()` ရလဒ်ကို ပရင့်ထုတ်ပါ။

**Hints:** `from pydantic import BaseModel, ValidationError` သုံးပါ။ `try/except ValidationError` နဲ့ error ကို ဖမ်းပါ။

**Expected behavior:** မှန်တဲ့ dict က instance ဆောက်ပြီး၊ မှားတဲ့ dict က error list တစ်ခု ပရင့်ထုတ်သည်။

## လေ့ကျင့်ခန်း ၂ — Nested schema နဲ့ field constraints

`Recipe` model ထဲမှာ `ingredients: list[str]` နဲ့ nested `Nutrition` model (protein, carbs တို့လိုမျိုး) ပါဝင်စေပြီး `rating` က 0.0 မှ 5.0 အတွင်း သာ ခံနိုင်စေပါ။

**Hints:** Pydantic ရဲ့ `Field(ge=0, le=5)` constraint နဲ့ nested BaseModel သုံးပါ (docs.pydantic.dev ကို ကြည့်ပါ)။

**Expected behavior:** 5.0 ထက်ကြီးတဲ့ rating က validation မအောင်ပဲ error ပြသည်။

## လေ့ကျင့်ခန်း ၃ — Simulated LLM JSON parsing

LLM က JSON string ပြန်တယ်လို့ ယူဆပြီး `json.loads()` နဲ့ parse လုပ်ပါ၊ parse အောင်မှ ကြိုပြီးရေးထားတဲ့ Pydantic model နဲ့ စစ်ပါ။ JSON string နှစ်မျိုး — တစ်ခု မှန်၊ တစ်ခု JSON syntax ပျက်နေတာ — ကို စမ်းပြပါ။

**Hints:** `json.JSONDecodeError` နဲ့ `ValidationError` နှစ်မျိုးလုံးကို သီးသန့် ဖမ်းပါ။

**Expected behavior:** Syntax ပျက်နေတာက JSONDecodeError ပြ၊ သာမန်မှားတာက ValidationError ပြသည်။

## လေ့ကျင့်ခန်း ၄ — Error ကို feedback prompt အဖြစ် ပြောင်းခြင်း

`build_repair_prompt(original_prompt, raw_output, error_text)` function ရေးပါ။ အဲဒီ function က model ကို — original prompt၊ မှားတဲ့ output၊ validation error စာသား — သုံးခုစလုံး ပါဝင်တဲ့ "ထပ်မှန်အောင် ရေးပါ" ဆိုတဲ့ prompt တစ်ခု ဆောက်ပေးရမယ်။

**Hints:** System message မှာ schema ဖော်ပြချက် ထည့်ပြီး user message မှာ error ကို quote လုပ်ပြပါ။

**Expected behavior:** ပြန်လာတဲ့ string ထဲမှာ original prompt နဲ့ error message နှစ်ခုလုံး ပါဝင်သည်။

## လေ့ကျင့်ခန်း ၅ — Bounded repair loop

`repair_loop(llm_fn, max_retries=3)` function ရေးပါ — `llm_fn(feedback)` က dict ပြန်တယ်၊ validation မှားရင် error string ကို feedback အဖြစ် ထည့်ပြီး ထပ်ဖိတ်ပါ၊ `max_retries` ပြည့်ရင် `None` ပြန်ပါ။ စမ်းသပ်မှုအတွက် ပထမနှစ်ကြိမ် မှားပြီး တတိယကြိမ်မှာ မှန်တဲ့ fake `llm_fn` နဲ့ စမ်းပါ။

**Hints:** `for attempt in range(...)` နဲ့ list ထဲ feedback history သိမ်းပါ။

**Expected behavior:** နှစ်ကြိမ်မှားပြီး တတိယအကြိမ် အောင်မြင်တဲ့ instance ပြန်သည်။

## လေ့ကျင့်ခန်း ၆ — Deterministic fallback

ကျင့်ခန်း ၅ ရဲ့ `repair_loop` က `None` ပြန်ရင် ခေါ်မယ့် `fallback()` function ရေးပါ — အစားထိုးရလဒ်တစ်ခု (ဥပမာ default value တွေပါတဲ့ မှန်ကန်တဲ့ instance) ပြန်ပေးရမယ်၊ ဒါမှမဟုတ် "human review" queue ဆိုတဲ့ list ထဲ ထည့်ပါ။ အခြေအနေနှစ်မျိုးလုံး (အောင်/မအောင်) စမ်းပြပါ။

**Hints:** Fallback ရလဒ်က Pydantic validation ကို တစ်ခါတည်း ဖြတ်ရမယ်။ Global list တစ်ခုကို review queue အဖြစ် သုံးပါ။

**Expected behavior:** Loop အောင်ရင် LLM ရလဒ်၊ မအောင်ရင် fallback ရလဒ် ပြန်သည်။
