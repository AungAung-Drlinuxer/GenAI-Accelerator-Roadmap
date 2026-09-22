# အဖြေများ — Cost, Latency & Model Routing

## လေ့ကျင့်ခန်း ၁ — Per-feature cost tracker

```python
# Per-feature cost accounting with per-1K-token prices in USD
PRICING = {
    "gpt-4o-mini": {"input": 0.15 / 1000, "output": 0.60 / 1000},
    "gpt-4o":      {"input": 2.50 / 1000, "output": 10.00 / 1000},
}

costs = {}  # feature name -> accumulated USD

def record_call(feature: str, model: str, prompt_tokens: int, completion_tokens: int):
    # Compute the call cost and add it to the feature total
    rates = PRICING[model]
    call_cost = prompt_tokens * rates["input"] + completion_tokens * rates["output"]
    costs[feature] = costs.get(feature, 0.0) + call_cost

def total_cost(feature: str) -> float:
    # Return the accumulated cost for one feature
    return costs.get(feature, 0.0)

record_call("chat", "gpt-4o-mini", 1000, 500)
record_call("chat", "gpt-4o-mini", 1000, 500)
print(f"chat total: ${total_cost('chat'):.4f}")
# Expected output: chat total: $0.9000
```

**အဓိကအယူအဆ** — Feature အလိုက် cost စာရင်းခွဲမှသာ ဘယ် feature က စရိတ်အများဆုံးစားတယ်ဆိုတာကို တိတ်ကျတိတ်ကျသိနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Latency budget checker

```python
def p95(latencies: list) -> float:
    # Compute the 95th percentile of a latency list (ms)
    if not latencies:
        return 0.0
    s = sorted(latencies)
    idx = min(int(len(s) * 0.95), len(s) - 1)
    return s[idx]

def check_budget(latencies: list, budget_ms: float) -> str:
    # Compare p95 against the latency budget
    value = p95(latencies)
    return "OVER BUDGET" if value > budget_ms else "OK"

print(check_budget([100, 200, 300, 5000], 3000))
print(check_budget([100, 200, 300, 400], 3000))
# Expected output:
# OVER BUDGET
# OK
```

**အဓိကအယူအဆ** — Latency ကို ပျမ်းမျှနှင့်မဟုတ်ဘဲ percentile နဲ့တိုင်းတာမှ user အများစုရဲ့ အတွေ့အကြုံကို မှန်ကန်စွာကိုယ်စားပြုနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Exact-match cache

```python
import hashlib

class LLMCache:
    def __init__(self):
        # Store cached answers and hit/miss statistics
        self.store = {}
        self.hit_count = 0
        self.miss_count = 0

    def _key(self, model: str, prompt: str) -> str:
        # Stable hash key from model + prompt
        return hashlib.sha256(f"{model}::{prompt}".encode()).hexdigest()

    def get(self, model: str, prompt: str, producer):
        # Return cached answer on hit, call producer on miss
        key = self._key(model, prompt)
        if key in self.store:
            self.hit_count += 1
            return self.store[key]
        self.miss_count += 1
        answer = producer()
        self.store[key] = answer
        return answer

cache = LLMCache()
fake = lambda: "Paris"
cache.get("gpt-4o-mini", "Capital of France?", fake)
cache.get("gpt-4o-mini", "Capital of France?", fake)
cache.get("gpt-4o-mini", "Capital of France?", fake)
print(f"hits={cache.hit_count}, misses={cache.miss_count}")
# Expected output: hits=2, misses=1
```

**အဓိကအယူအဆ** — Cache hit rate မြင့်လာတာဟာ token ကုန်ကြေးနှင့် latency နှစ်မျိုးလုံး တပြိုင်တည်းလျှော့ကျတာကို ဆိုလိုပါတယ်။

## လေ့ကျင့်ခန်း ၄ — Complexity-based router

```python
SMALL_MODEL = "gpt-4o-mini"
LARGE_MODEL = "gpt-4o"

COMPLEX_KEYWORDS = ["analyze", "compare", "step-by-step", "reason", "evaluate"]

def route(prompt: str) -> str:
    # Long prompts or complex keywords go to the large model
    if len(prompt) > 400:
        return LARGE_MODEL
    lowered = prompt.lower()
    if any(k in lowered for k in COMPLEX_KEYWORDS):
        return LARGE_MODEL
    return SMALL_MODEL

short_prompt = "Summarize these meeting notes in one line."
long_prompt = "Analyze this quarterly report and compare revenue trends " + "with details. " * 80

print(route(short_prompt))
print(route("Compare the two proposals step-by-step and reason about risks."))
print(route(long_prompt))
# Expected output:
# gpt-4o-mini
# gpt-4o
# gpt-4o
```

**အဓိကအယူအဆ** — Request အများစုကို သေးငယ်တဲ့ model နဲ့ လုပ်ဆောင်စေခြင်းက cost ကို သိသိသာသာချွေတာပေးပြီး ခက်ခဲတဲ့ request တွေအတွက် quality ကို ဆက်ထိန်းပေးနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၅ — Fallback chain

```python
def broken_primary(prompt: str) -> str:
    # Simulate a provider outage
    raise RuntimeError("primary provider down")

def broken_secondary(prompt: str) -> str:
    # Simulate a second provider failure
    raise RuntimeError("secondary provider rate-limited")

def onprem_model(prompt: str) -> str:
    # Simulate a self-hosted model that always works
    return f"on-prem answer to: {prompt}"

PROVIDERS = [
    {"name": "primary", "call": broken_primary},
    {"name": "secondary", "call": broken_secondary},
    {"name": "on-prem", "call": onprem_model},
]

def with_fallback(providers: list, prompt: str) -> str:
    # Try each provider in order until one succeeds
    for provider in providers:
        try:
            return provider["call"](prompt)
        except RuntimeError:
            print(f"{provider['name']} failed, falling back...")
    raise RuntimeError("all providers failed")

print(with_fallback(PROVIDERS, "What is the capital of France?"))
# Expected output:
# primary failed, falling back...
# secondary failed, falling back...
# on-prem answer to: What is the capital of France?
```

**အဓိကအယူအဆ** — Provider တစ်ခုတည်းပေါ်မှီခိုမှုဟာ outage တစ်ခုနဲ့ service တစ်ခုလုံးရပ်စေနိုင်လို့ fallback chain ဟာ production အတွက် မဖြစ်မနေလိုအပ်ပါတယ်။

## လေ့ကျင့်ခန်း ၆ — Alert threshold checker

```python
ALERTS = {
    "latency_ms": 3000,
    "usd_per_day": 50.0,
    "error_rate": 0.05,
}

def check_alerts(metrics: dict) -> list:
    # Compare each metric against its threshold; collect alert messages
    alerts = []
    for metric, value in metrics.items():
        if metric in ALERTS and value > ALERTS[metric]:
            alerts.append(f"ALERT: {metric}={value} exceeds threshold {ALERTS[metric]}")
    return alerts

print(check_alerts({"latency_ms": 4000}))
print(check_alerts({"latency_ms": 1500, "usd_per_day": 12.0}))
# Expected output:
# ['ALERT: latency_ms=4000 exceeds threshold 3000']
# []
```

**အဓိကအယူအဆ** — Threshold ကျော်တဲ့ အခြေအနေကို အချိန်နှင့်တပြိုင်တည်းသိအောင် alert ထည့်သွင်းခြင်းက ကုန်ကျစရိတ်နှင့် performance ပြဿနာတွေကို အရေးကြီးစောင်းမတ်
