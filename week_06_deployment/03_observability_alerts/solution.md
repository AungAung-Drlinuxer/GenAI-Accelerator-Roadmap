## လေ့ကျင့်ခန်း ၁ — Structured Logging နှင့် JSON Log Format

AI application တစ်ခုမှ request တစ်ခု၏ log ကို JSON format ဖြင့် ရေးသွင်းသည့် function ကို ရေးပါ။ နောက်ပိုင်း filter လုပ်ရလွယ်ကူစေရန် structured log ကို အသုံးပြုပါသည်။

```python
import json
import time
import logging
import uuid

logger = logging.getLogger("ai-app")

def log_event(event_name: str, level: str = "info", **fields) -> None:
    # Build a structured log record as a plain dictionary
    record = {
        "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "level": level,
        "event": event_name,
        "request_id": fields.pop("request_id", str(uuid.uuid4())),
    }
    # Merge extra fields such as model, latency_ms, status
    record.update(fields)
    # Emit as a single JSON line so log collectors can parse it easily
    logger.log(getattr(logging, level.upper(), logging.INFO), json.dumps(record, ensure_ascii=False))

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    log_event("inference_request", model="gpt-4o-mini", latency_ms=830, status="ok")
    log_event("inference_request", level="error", model="gpt-4o-mini", status="timeout")
```

**အဓိကအယူအဆ** — Log တစ်ခုစီကို JSON dictionary အဖြစ် structured စွာရေးသွင်းခြင်းဖြင့် နောက်ပိုင်း error ရှာဖွေရာနှင့် metrics တွက်ချက်ရာတွင် key အလိုက် filter လုပ်နိုင်ပါသည်။

## လေ့ကျင့်ခန်း ၂ — Sentry ဖြင့် Error Tracking ချိတ်ဆက်ခြင်း

Sentry SDK ကို install လုပ်ကာ application ၏ unhandled exception များကို Sentry dashboard ဆီပို့ပေးသည့် setup ကို စမ်းသပ်ပါ (docs.sentry.io အရ)။

```python
# pip install sentry-sdk
import sentry_sdk

# Initialize Sentry with a DSN from your Sentry project settings
sentry_sdk.init(
    dsn="https://examplePublicKey@o0.ingest.sentry.io/0",  # replace with your real DSN
    traces_sample_rate=0.1,      # sample 10% of performance traces
    send_default_pii=False,      # do not send personal data
)

def divide(a: float, b: float) -> float:
    if b == 0:
        # Raise a clear error instead of returning silently
        raise ValueError("division by zero is not allowed")
    return a / b

def run_job() -> None:
    try:
        result = divide(10, 0)
        print("result:", result)
    except ValueError as exc:
        # Capture the exception with extra context for debugging
        sentry_sdk.capture_exception(exc)
        sentry_sdk.set_tag("component", "math_worker")
        print("error captured and sent to Sentry")

if __name__ == "__main__":
    run_job()
```

**အဓိကအယူအဆ** — Sentry SDK ကို DSN ဖြင့် init လုပ်ပြီး `capture_exception` ကို အသုံးပြုခြင်းဖြင့် production error များကို tag နှင့် context အပြည့်အစုံဖြင့် စုစည်းစေနိုင်ပါသည်။

## လေ့ကျင့်ခန်း ၃ — Latency နှင့် Error Metrics စုဆောင်ခြင်း

Request တိုင်း၏ latency နှင့် error ဖြစ်ပွားမှုကို in-memory list ထဲမှတ်ပြီး p95 latency နှင့် error rate တွက်ချက်သည့် mini metrics collector ကို ရေးပါ။

```python
import random
from collections import defaultdict

class MetricsCollector:
    def __init__(self) -> None:
        # Store latency samples and counters per endpoint
        self.latencies = defaultdict(list)
        self.counters = defaultdict(lambda: {"total": 0, "errors": 0})

    def record(self, endpoint: str, latency_ms: float, is_error: bool) -> None:
        self.latencies[endpoint].append(latency_ms)
        self.counters[endpoint]["total"] += 1
        if is_error:
            self.counters[endpoint]["errors"] += 1

    def p95_latency(self, endpoint: str) -> float:
        # Sort samples and take the value at the 95th percentile position
        samples = sorted(self.latencies[endpoint])
        if not samples:
            return 0.0
        idx = int(0.95 * (len(samples) - 1))
        return samples[idx]

    def error_rate(self, endpoint: str) -> float:
        stats = self.counters[endpoint]
        if stats["total"] == 0:
            return 0.0
        return stats["errors"] / stats["total"]

if __name__ == "__main__":
    m = MetricsCollector()
    for _ in range(100):
        lat = random.uniform(100, 900)
        err = random.random() < 0.03  # simulate ~3% errors
        m.record("/chat", lat, err)
    print(f"p95 latency: {m.p95_latency('/chat'):.0f} ms")
    print(f"error rate: {m.error_rate('/chat'):.2%}")
```

**အဓိကအယူအဆ** — Request တိုင်းမှ latency sample များနှင့် error count များကို စုဆောင်းထားပါက p95 latency နှင့် error rate ကို code ဖြင့်တိုက်ရိုက်တွက်ချက်နိုင်ပါသည်။

## လေ့ကျင့်ခန်း ၄ — SLO စစ်ဆေးပြီး Alert ထုတ်ပေးခြင်း

p95 latency နှင့် error rate အတွက် SLO threshold သတ်မှတ်ကာ ကျော်လွန်ပါက alert message ထုတ်ပေးသည့် checker ကို ရေးပါ။

```python
from dataclasses import dataclass

@dataclass
class SLO:
    name: str
    p95_latency_budget_ms: float   # e.g. 1000 ms
    error_rate_budget: float        # e.g. 0.05 means 5%

def check_slo(slo: SLO, p95_latency_ms: float, error_rate: float) -> list:
    # Return a list of alert messages for any SLO violation
    alerts = []
    if p95_latency_ms > slo.p95_latency_budget_ms:
        alerts.append(
            f"ALERT: {slo.name} p95 latency {p95_latency_ms:.0f} ms "
            f"exceeds budget {slo.p95_latency_budget_ms:.0f} ms"
        )
    if error_rate > slo.error_rate_budget:
        alerts.append(
            f"ALERT: {slo.name} error rate {error_rate:.2%} "
            f"exceeds budget {slo.error_rate_budget:.2%}"
        )
    if not alerts:
        alerts.append(f"OK: {slo.name} is within SLO")
    return alerts

if __name__ == "__main__":
    chat_slo = SLO("chat-api", p95_latency_budget_ms=1000, error_rate_budget=0.05)
    for msg in check_slo(chat_slo, p95_latency_ms=1250.0, error_rate=0.07):
        print(msg)
```

**အဓိကအယူအဆ** — SLO ဆိုသည်မှာ latency နှင့် error rate အတွက် ကြိုတင်သတ်မှတ်ထားသော လက်ခံနိုင်သည့်အနိမ့်ဆုံးအဆင့်အတန်းဖြစ်ပြီး ၎င်းကို ကျော်လွန်ပါက alert ထုတ်ရမည်ဖြစ်သည်။

## လေ့ကျင့်ခန်း ၅ — On-call Playbook အတွက် Automated Triage Step

Alert တစ်ခုရရှိပါက လိုက်လုပ်ရမည့် အဆင့်တွင် အလုပ်အကိုင်များကို log ထဲမှ error pattern အလိုက် ရွေးချယ်ပြသည့် triage helper ကို ရေးပါ။

```python
def triage_alert(alert_message: str) -> list:
    # Map known error patterns to concrete on-call actions
    playbook = {
        "timeout": [
            "Check upstream model provider status page",
            "Inspect p95 latency dashboard for the last 30 minutes",
            "Retry failed jobs from the dead-letter queue",
        ],
        "error rate": [
            "Open Sentry and review the top issues by event count",
            "Check the latest deployment in the release history",
            "Roll back the last deploy if the error started right after it",
        ],
        "latency": [
            "Check GPU/instance utilization metrics",
            "Verify cache hit rate has not dropped",
            "Scale up replicas if traffic spiked",
        ],
    }
    actions = []
    lowered = alert_message.lower()
    for pattern, steps in playbook.items():
        if pattern in lowered:
            actions.extend(steps)
    if not actions:
        actions.append("No matching playbook entry; escalate to the secondary on-call")
    return actions

if __name__ == "__main__":
    alert = "ALERT: chat-api error rate 7% exceeds budget 5%"
    for i, step in enumerate(triage_alert(alert), start=1):
        print(f"Step {i}: {step}")
```

**အဓိကအယူအဆ** — On-call playbook ဆိုသည်မှာ alert တစ်မျိုးစီအတွက် လိုက်လုပ်ရမည့် အဆင့်တွင်အလုပ်အကိုင်စာရင်း ကြိုတင်ရေးသွင်းထားခြင်းဖြစ်ပြီး alert ဖြစ်ပွားချိန်တွင် စဉ်းစားစရာမလိုဘဲ အမြန်လိုက်လုပ်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၆ — Langfuse-style Trace Logging အတုအသုံးပြုခြင်း

LLM call တစ်ခုချင်းစီအတွက် trace နှင့် span record များကို JSON file ထဲသိမ်းဆည်းပြီး observability review လုပ်နိုင်စေသည့် logger ကို ရေးပါ (langfuse.com/docs အယူအဆအရ)။

```python
import json
import time
import uuid

class TraceLogger:
    def __init__(self, filepath: str = "traces.jsonl") -> None:
        self.filepath = filepath

    def log_trace(self, user_id: str, prompt: str, output: str,
                  latency_ms: float, model: str) -> None:
        # Each line is one trace record in JSON Lines format
        record = {
            "trace_id": str(uuid.uuid4()),
            "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
            "user_id": user_id,
            "model": model,
            "prompt": prompt,
            "output": output,
            "latency_ms": latency_ms,
        }
        with open(self.filepath, "a", encoding="utf-8") as f:
            f.write(json.dumps(record, ensure_ascii=False) + "\n")

if __name__ == "__main__":
    t = TraceLogger()
    t.log_trace("user-42", "What is SLO?", "SLO is a reliability target.",
                latency_ms=640.0, model="gpt-4o-mini")
    # Read back all traces to inspect them later
    with open("traces.jsonl", encoding="utf-8") as f:
        for line in f:
            print(json.loads(line)["model"])
```

**အဓိကအယူအဆ** — LLM application တွင် request တိုင်း၏ prompt, output, latency နှင့် model ကို trace record အဖြစ် သိမ်းဆည်းခြင်းဖြင့် နောက်ပိုင်း quality စစ်ဆေးရန်နှင့် ပြဿနာရှာဖွေရန် လိုအပ်သောအချက်အလက်များ ရရှိပါသည်။
