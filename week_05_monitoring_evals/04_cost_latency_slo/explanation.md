# Cost, Latency & Model Routing — အသေးစိတ်ရှင်းလင်းချက်

## ၁။ Token/cost accounting per feature

### ဘာကို ဆိုလိုတာလဲ

LLM application တစ်ခုထဲမှာ feature အများအပြားရှိတတ်ပါတယ် — chat, summarization, classification စသဖြင့်။ Cost accounting ဆိုတာ feature တစ်ခုချင်းစီအတွက် input token၊ output token နှင့် ကုန်ကျစရိတ်ကို သီးခြားခွဲပြီး မှတ်တမ်းတင်ခြင်းကို ဆိုလိုပါတယ်။

### ဘာကြောင့် လဲ

ဘိလ်တစ်ခုတည်းပေးရုံနဲ့ ဘယ် feature က ဘယ်လောက်စားတယ်ဆိုတာ မသိရပါဘူး။ Feature အလိုက် ခွဲမှသာ ဘယ် feature ကို optimize လုပ်သင့်တယ်ဆိုတာကို ဆုံးဖြတ်နိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

API response ရဲ့ `usage` field ထဲက `prompt_tokens` နှင့် `completion_tokens` ကိုယူပြီး၊ provider ရဲ့ per-token ဈေးနှုန်းနဲ့ မြှောက်ကာ feature tag တစ်ခုနှင့်အတူ tracing tool (ဥပမာ Langfuse) ထဲ မှတ်တမ်းတင်ပါတယ်။

### ဥပမာ

```python
# Simple per-feature cost tracking (no external calls, offline calculation)
PRICING = {"gpt-4o-mini": {"input": 0.00015 / 1000, "output": 0.0006 / 1000}}

def compute_cost(model: str, prompt_tokens: int, completion_tokens: int) -> float:
    # Look up per-token prices and multiply by usage
    rates = PRICING[model]
    return prompt_tokens * rates["input"] + completion_tokens * rates["output"]

usage = {"prompt_tokens": 1500, "completion_tokens": 300}
cost = compute_cost("gpt-4o-mini", usage["prompt_tokens"], usage["completion_tokens"])
print(f"Cost for this call: ${cost:.6f}")
# Expected output: Cost for this call: $0.000405
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

ကုန်ကျစရိတ်ကို မြင်နိုင်မှ ဘယ် feature က budget ထက်ကျော်နေတယ်ဆိုတာကို အချိန်မီသိပြီး prompt တိုအောင်လုပ်တာ၊ model ငယ်ကို ပြောင်းတာ စတာတွေ ဆုံးဖြတ်နိုင်ပါတယ်။

## ၂။ Latency budgets

### ဘာကို ဆိုလိုတာလဲ

Latency budget ဆိုတာ feature တစ်ခုအတွက် တုံ့ပြန်ချိန်အများဆုံး (upper bound) ကို ကြိုတင်သတ်မှတ်ထားတဲ့ ပန်းတန်ဇာတိုင်တာကို ဆိုလိုပါတယ်။ ဥပမာ — chat feature အတွက် p95 က ၃ စက္ကန့်ကို မကျော်ရ။

### ဘာကြောင့် လဲ

User တွေဟာ နှောင့်နှေးတဲ့ app ကို စွန့်ခွာလိုက်ကြပါတယ်။ Budget မသတ်ထားရင် တဖြည်းဖြည်းနှောင့်နှေးလာတဲ့ system ကို သတိမထားမိဘဲ ဖြစ်နေတတ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

တစ်ချိန်ချိန်း (timestamp) ကို ခေါ်ဆွဲမှု စတင်ချိန်မှာ ဖမ်းပြီး ပြီးတဲ့အခါ ကွာဟချိန်တွက်ပါတယ်။ Percentile (p50, p95) တွေနဲ့ တိုင်းတာတာ budget နဲ့ နိုင်းယူပါတယ်။

### ဥပမာ

```python
import time
import statistics

# Collect latency samples in milliseconds
latencies_ms = []

def measure_call(duration_s: float):
    # Simulate one API call and record its duration
    time.sleep(duration_s)
    latencies_ms.append(duration_s * 1000)

measure_call(0.4)
measure_call(0.6)
measure_call(1.1)

def percentile(sorted_values: list, p: float) -> float:
    # Simple percentile: index into a sorted list
    k = int(len(sorted_values) * p)
    return sorted_values[min(k, len(sorted_values) - 1)]

s = sorted(latencies_ms)
print(f"p50 = {percentile(s, 0.50):.0f} ms, p95 = {percentile(s, 0.95):.0f} ms")
# Expected output: p50 = 600 ms, p95 = 1100 ms
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Budget ရှိမှ alert threshold သတ်မှတ်လို့ရပါတယ်။ Budget ကျော်တဲ့ request တွေကို ခွဲထုတ်ပြီး cause (long prompt, slow model, network) ကို စုံစမ်းနိုင်ပါတယ်။

## ၃။ Caching layers

### ဘာကို ဆိုလိုတာလဲ

Caching layer ဆိုတာ တူညီတဲ့ prompt အတွက် ထပ်မခေါ်ဘဲ၊ အရင်ဖြေပြီးတဲ့အဖြေကို ပြန်သုံးခြင်းကို ဆိုလိုပါတယ်။ Exact-match cache နှင့် semantic cache ဆိုပြီး နှစ်မျိုးရှိပါတယ်။

### ဘာကြောင့် လဲ

အဖြေတူတူကို ထပ်ခေါ်တိုင်း token ကုန်၊ ငွေကုန်၊ အချိန်ကုန်ပါတယ်။ Cache က ဒီသုံးမျိုးလုံးကို တပြိုင်တည်းချွေတာပေးပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Prompt + model + parameter တွေကို key အဖြစ် hash လုပ်ပြီး dictionary သို့မဟုတ် Redis ထဲသိမ်းပါတယ်။ Request အသစ်ဝင်တိုင်း key ရှိမရှိစစ်ပြီး ရှိရင် cached answer ကိုပဲ ပြန်ပေးပါတယ်။

### ဥပမာ

```python
import hashlib
import json

class ExactCache:
    def __init__(self):
        # In-memory cache for demonstration
        self.store = {}

    def _key(self, model: str, prompt: str, params: dict) -> str:
        # Build a stable cache key from model, prompt, and params
        payload = json.dumps({"model": model, "prompt": prompt, "params": params},
                             sort_keys=True)
        return hashlib.sha256(payload.encode()).hexdigest()

    def get_or_set(self, model: str, prompt: str, params: dict, producer):
        # Return cached value if present, otherwise call the producer
        key = self._key(model, prompt, params)
        if key not in self.store:
            self.store[key] = producer()
            print("MISS -> called model")
        else:
            print("HIT -> reused cached answer")
        return self.store[key]

cache = ExactCache()
fake_answer = lambda: "The capital of France is Paris."
print(cache.get_or_set("gpt-4o-mini", "What is the capital of France?", {}, fake_answer))
print(cache.get_or_set("gpt-4o-mini", "What is the capital of France?", {}, fake_answer))
# Expected output:
# MISS -> called model
# The capital of France is Paris.
# HIT -> reused cached answer
# The capital of France is Paris.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

FAQ-style feature တွေမှာ prompt တူညီတာများတတ်လို့ cache hit rate မြင့်လို့ ရလဒ်အရ ကုန်ကျစရိတ်သိသိသာသာလျှော့နိုင်ပါတယ်။ သတိထားရတာက time-sensitive အဖြေတွေကို cache လုပ်ပါက ရိုးရိုးဟောင်းနေနိုင်ပါတယ်။

## ၄။ Small-vs-large model routing

### ဘာကို ဆိုလိုတာလဲ

Model routing ဆိုတာ request တစ်ခုချင်းကို သူ့ရဲ့ ရှုပ်ထွေးမှု (complexity) ပေါ်မူတည်ပြီး သေးငယ်ပြီးစရိတ်သက်သက်သာတဲ့ model နဲ့ ခေါ်မလား၊ ကြီးမားပြီးတိကျတဲ့ model နဲ့ ခေါ်မလား ရွေးချယ်ခြင်းကို ဆိုလိုပါတယ်။

### ဘာကြောင့် လဲ

ရိုးရိုး classification သို့မဟုတ် extraction task တွေကို ကြီးမားတဲ့ model နဲ့ ခေါ်တာဟာ ငွေဖြုန်းတာပါ။ သင့်တင့်တဲ့ model ကို ရွေးချယ်မှသာ cost/quality ဟန်ချက်ညီမှ ရပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Classifier တစ်ခု (rule-based သို့မဟုတ် သေးငယ်တဲ့ model) က request ကို "simple" လား "complex" လား သတ်မှတ်ပြီး route လုပ်ပါတယ်။ အဖြေရည်မှန်မှန် စစ်ဖို့ evaluation score ကိုပါ တွဲကြည့်ပါတယ်။

### ဥပမာ

```python
# Rule-based router: short and keyword-free prompts go to the small model
SMALL = "gpt-4o-mini"
LARGE = "gpt-4o"

def route(prompt: str) -> str:
    # Simple heuristics: length and complexity keywords
    complex_signals = ["analyze", "compare", "step-by-step", "why", "reason"]
    if len(prompt) > 400 or any(s in prompt.lower() for s in complex_signals):
        return LARGE
    return SMALL

print(route("Summarize: meeting notes attached."))
print(route("Compare the trade policy of two countries and reason step-by-step."))
# Expected output:
# gpt-4o-mini
# gpt-4o
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Traffic အများစုက ရိုးရိုး task ဖြစ်တတ်လို့ routing ချက်က cost ကို သိသိသာသာချွေတာပေးပြီး၊ ခက်ခဲတဲ့ request တွေမှာ quality မပျက်စေပါဘူး။

## ၅။ On-prem fallback models နှင့် alert thresholds

### ဘာကို ဆိုလိုတာလဲ

Fallback model ဆိုတာ provider API က မသုံးနိုင်ဖြစ်တဲ့အခါ (outage, rate limit, budget ကုန်) အစားထိုးသုံးမယ့် စရိတ်သက်သက်သာတဲ့သို့မဟုတ် self-hosted model ကို ဆိုပါတယ်။ Alert threshold ဆိုတာ cost၊ latency သို့မဟုတ် error rate က သတ်မှတ်နယ်နိမိတ်ကျော်တဲ့အခါ သတိပေးချက်ထုတ်ဖို့ နယ်နိမိတ်တစ်ခုကို ဆိုပါတယ်။

### ဘာကြောင့် လဲ

Single provider ပေါ်မှာသာ မှီခိုရင် outage တစ်ခုနဲ့ service တစ်ခုလုံး ရပ်နေတတ်ပါတယ်။ Threshold မရှိရင် ပုံမှန်ထက် cost ကြီးစားနေတာကို ရက်ပိုင်းကြာမှ သိရတတ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Call chain ထဲမှာ provider အများအပြားကို အစီအစဉ်အတိုင်း စမ်းပြီး၊ အရင်တစ်ခု fail ရင်နောက်တစ်ခုကို ပြောင်းခေါ်ပါတယ်။ Metrics တွေကို monitoring tool ထဲမှတ်ပြီး threshold ကျော်ရင် alert event ထုတ်ပါတယ်။

### ဥပမာ

```python
import time

# Thresholds that trigger alerts
ALERTS = {
    "latency_ms": 3000,   # alert if a call is slower than 3 s
    "usd_per_day": 50.0,  # alert if daily spend exceeds $50
}

def maybe_alert(metric: str, value: float) -> str | None:
    # Compare a metric against its threshold
    if metric in ALERTS and value > ALERTS[metric]:
        return f"ALERT: {metric}={value} exceeds {ALERTS[metric]}"
    return None

print(maybe_alert("latency_ms", 4200))
print(maybe_alert("usd_per_day", 12.5))
# Expected output:
# ALERT: latency_ms=4200 exceeds 3000
# None
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Service မရပ်တန့်အောင် (resilience) နှင့် စရိတ်က ထိန်းချုပ်နယ်ထဲရှိနေအောင် အာမခံပေးပါတယ်။ Production system တိုင်းမှာ ဒီနှစ်ခုဟာ optional မဟုတ်ဘဲ မဖြစ်မနေ လိုအပ်ပါတယ်။

## အနှစ်ချုပ်

Cost accounting က ဘယ် feature က ဘယ်လောက်စားတယ်ဆိုတာ ပြပါတယ်။ Latency budget က user experience ကို ကာကွယ်ပါတယ်။ Cache နှင့် routing က cost ကို ချွေတာပေးပြီး၊ fallback နှင့် alert threshold တွေက system ကို တည်ငြိမ်စေပါတယ်။ ဒီအချက်ငါးခုကို တွဲစဉ်သုံးမှသာ LLM app တစ်ခုက production မှာ ရှင်သန်နိုင်ပါတယ်။
