# လေ့ကျင့်ခန်းများ — Cost, Latency & Model Routing

## လေ့ကျင့်ခန်း ၁ — Per-feature cost tracker

`record_call(feature, model, prompt_tokens, completion_tokens)` ဟုခေါ်ဆွူရမယ့် function တစ်ခုရေးပါ။ Feature အလိုက် စုစည်းကုန်ကျစရိတ်ကို dictionary ထဲသိမ်းပြီး၊ `total_cost(feature)` နဲ့ ပြန်ဖတ်နိုင်စေပါ။ ဈေးနှုန်းကို per-1K-token USD အဖြစ် သတ်မှတ်ပါ။

**Hints:** PRICING dictionary ထဲမှာ input/output ဈေးနှုန်းနှစ်မျိုးထားပြီး token ကို 1000 နဲ့ စားပြီးမမြှောက်ပါ။

**Expected behavior:** `record_call("chat", "gpt-4o-mini", 1000, 500)` နှစ်ကြိမ်ခေါ်ပြီး `total_cost("chat")` ဟိတ်ကြောင်း ဈေးနှုန်းနှစ်ဆ ပြနေရမည်။

## လေ့ကျင့်ခန်း ၂ — Latency budget checker

Call duration တွေကို list ထဲပြောင်းပြီး `p95()` function နဲ့ ၉၅-percentile တွက်စေပါ။ `check_budget(latencies, budget_ms)` က budget ကျော်ရင် `"OVER BUDGET"` မဟုတ်ရင် `"OK"` ပြန်စေပါ။

**Hints:** `sorted()` နဲ့ အစီအညီချပြီး index `int(n * 0.95)` ကို သတိထားပြီး clamp လုပ်ပါ။

**Expected behavior:** `[100, 200, 300, 5000]` နှင့် budget 3000 ပေးလျှင် `"OVER BUDGET"` ပြန်ရမည်။

## လေ့ကျင့်ခန်း ၃ — Exact-match cache

`LLMCache` class တစ်ခုရေးပါ — `get(model, prompt)` က cache ထဲမှာပါရင် အဖြေဟောင်းပြန်ပြီး၊ မပါရင် `producer` function ကို ခေါ်ပြီးသိမ်းပါ။ Hit/Miss count တွေကို စာရင်းပြပါ။

**Hints:** Key ကို `hashlib.sha256` နဲ့ model+prompt ကို hash လုပ်ပြီး `hit_count`, `miss_count` ထားပါ။

**Expected behavior:** Prompt တူတစ်ခုကို သုံးကြိမ်တောင်းလျှင် hits=2, misses=1 ဖြစ်ရမည်။

## လေ့ကျင့်ခန်း ၄ — Complexity-based router

`route(prompt)` function ရေးပြီး prompt ရှည် (400+ chars) သို့မဟုတ် "analyze", "compare", "step-by-step" စတဲ့ စကားလုံးပါရင် large model name ပြန်၊ မဟုတ်ရင် small model name ပြန်စေပါ။

**Hints:** `.lower()` နဲ့ lowercase ပြောင်းပြီး keyword list ကို `any()` နဲ့ စစ်ပါ။

**Expected behavior:** တိုတိုရိုးရိုး prompt ကို small model၊ analysis prompt ကို large model ပြန်ရမည်။

## လေ့ကျင့်ခန်း ၅ — Fallback chain

Provider သုံးခု (primary, secondary, on-prem) ကို အစဉ်လိုက်စမ်းတဲ့ `with_fallback(providers, prompt)` function ရေးပါ။ ပထမနှစ်ခုက exception ပြန်လျှင် တတိယအောင် ဆက်စမ်းပြီး အောင်မြင်တဲ့ provider ရဲ့ အဖြေကို ပြန်ပါ။

**Hints:** `try/except` ထဲမှာ `provider["call"](prompt)` ကို ခေါ်ပြီး fail ရင် `continue` လုပ်ပါ။

**Expected behavior:** Primary နှင့် secondary တို့က `RuntimeError` မြှစ်လျှင် on-prem ရဲ့ အဖြေကို ရရမည်။

## လေ့ကျင့်ခန်း ၆ — Alert threshold checker

`ALERTS` thresholds dictionary နှင့် `check_alerts(metrics)` function ရေးပါ။ Metrics ထဲက တန်ဖိုးတစ်ခုမျှ threshold ကျော်ရင် alert message list တစ်ခု ပြန်ပါ၊ မကျော်ရင် empty list ပြန်ပါ။

**Hints:** Loop ထဲမှာ `if metric in ALERTS and value > ALERTS[metric]` စစ်ပြီး string စုပါ။

**Expected behavior:** `{"latency_ms": 4000}` ပေးလျှင် alert တစ်ခုပါတဲ့ list ပြန်ရမည်။
