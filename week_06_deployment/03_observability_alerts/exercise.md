## လေ့ကျင့်ခန်း ၁ — Structured Logging အခြေခံ

Python ၏ `logging` module ကို သုံးပြီး JSON format ဖြင့် log ထုတ်ပြခြင်း။

```python
import logging
import json

class JsonFormatter(logging.Formatter):
    def format(self, record):
        # Build a dictionary of log fields
        payload = {
            "level": record.levelname,
            "message": record.getMessage(),
            "time": self.formatTime(record),
        }
        return json.dumps(payload, ensure_ascii=False)

logger = logging.getLogger("ai_app")
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("inference completed")
```

**Hints:** `python-json-logger` library ကိုလည်း စမ်းကြည့်နိုင်သည်။ `ensure_ascii=False` က Burmese စာသားများအတွက် အရေးကြီးသည်။

**Expected behavior:** တစ်ကြောင်းလျှင်တစ်ခီ JSON object ဖြစ်သော `{"level": "INFO", "message": "inference completed", "time": ...}` ကဲ့သို့ output ထွက်သည်။

## လေ့ကျင့်ခန်း ၂ — LLM Request Log Fields

LLM inference request တစ်ခုအတွက် structured log entry တည်ဆောက်ခြင်း။ `request_id`, `model`, `prompt_tokens`, `latency_ms`, `status` fields များပါဝင်ရမည်။

```python
import uuid
import time

def log_llm_call(logger, model: str, prompt: str, answer_fn):
    # Wrap a call to a model with structured logging
    request_id = str(uuid.uuid4())
    start = time.perf_counter()
    try:
        result = answer_fn(prompt)
        latency_ms = (time.perf_counter() - start) * 1000
        logger.info("llm_call", extra={
            "request_id": request_id,
            "model": model,
            "latency_ms": round(latency_ms, 1),
            "status": "ok",
        })
        return result
    except Exception as e:
        logger.exception("llm_call failed", extra={
            "request_id": request_id,
            "model": model,
            "status": "error",
        })
        raise
```

**Hints:** `extra` dict ထဲ့ field တွေကို formatter က `record.__dict__` မှ ရနိုင်သည်။ `logger.exception` က stack trace ပါထည့်ပေးသည်။

**Expected behavior:** အောင်မြင်ပါက `status: ok` နှင့် latency ပါသော log တစ်ကြောင်းထွက်ပြီး၊ exception ဖြစ်ပါက stack trace ပါသော error log ထွက်သည်။

## လေ့ကျင့်ခန်း ၃ — Sentry နှင့် Error Tracking

Sentry SDK (docs.sentry.io) ကို သုံးပြီး unhandled exception ကို ဖမ်းယူတင်ပြခြင်း။

```python
import sentry_sdk

sentry_sdk.init(
    dsn="YOUR_SENTRY_DSN",  # from your Sentry project settings
    traces_sample_rate=0.1,
)

def risky_divide(a: int, b: int):
    return a / b

if __name__ == "__main__":
    try:
        risky_divide(10, 0)
    except ZeroDivisionError:
        sentry_sdk.capture_exception()
        print("captured and reported to Sentry")
```

**Hints:** DSN ကို Sentry project settings → Client Keys (DSN) တွင် ရနိုင်သည်။ Local စမ်းရန် `sentry_sdk.init()` မှ ပြန်တင်ခဲ့ပြီး Sentry UI။ Issues page တွင် ကြည့်ပါ။

**Expected behavior:** Script run လျှင် Sentry dashboard ၌ `ZeroDivisionError` issue အသစ်တစ်ခု ပေါ်လာပြီး stack trace နှင့် context များ ပြသည်။

## လေ့ကျင့်ခန်း ၄ — Metrics Recording နှင့် p95 Latency

Request latency များကို မှတ်တမ်းတင်ပြီး p95 latency တွက်ခြင်း။

```python
import time
import random
from collections import deque

latencies = deque(maxlen=1000)

def handle_request():
    # Simulate a request with random latency
    start = time.perf_counter()
    time.sleep(random.uniform(0.01, 0.2))
    latencies.append((time.perf_counter() - start) * 1000)

def p95(values):
    # 95th percentile: 95% of values are at or below this
    if not values:
        return None
    ordered = sorted(values)
    idx = int(len(ordered) * 0.95) - 1
    return ordered[max(idx, 0)]

for _ in range(100):
    handle_request()
print("p95 latency (ms):", p95(latencies))
```

**Hints:** `deque(maxlen=...)` က sliding window အဖြစ် အလုပ်လုပ်သည်။ Production တွင် Prometheus client library (`prometheus_client`) ၏ `Histogram` ကို သုံးနိုင်သည်။

**Expected behavior:** 100 request အပြီးတွင် p95 latency တန်ဖိုး (ခန့်မှန်းခြေအားဖြင့် 190 ms အနီးဝန်းကျင်၊ တကယ့်တန်ဖိုးမူကား) ထွက်ပြီး၊ တစ်ကြိမ်ချင်း အနည်းငယ်ကွဲနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — SLO Alerting Logic

p95 latency နှင့် error rate အတွက် SLO များ သတ်မှတ်ပြီး alert ဆုံးဖြတ်ခြင်း။

```python
def check_slos(latencies, errors, total, latency_slo_ms=500, error_rate_slo=0.01):
    # Return alerts when SLOs are violated
    alerts = []
    ordered = sorted(latencies)
    idx = max(int(len(ordered) * 0.95) - 1, 0)
    if ordered and ordered[idx] > latency_slo_ms:
        alerts.append("LATENCY_BREACH")
    if total > 0 and (errors / total) > error_rate_slo:
        alerts.append("ERROR_RATE_BREACH")
    return alerts

# Simulated data: 2 out of 20 requests failed, p95 over budget
lats = [100, 120, 130, 150, 700, 720, 160, 140, 135, 125,
        115, 145, 155, 165, 175, 185, 195, 205, 215, 600]
print(check_slos(lats, errors=2, total=20))
```

**Hints:** Alert level များ (warning vs critical) ထပ်ထည့်ကြည့်ပါ။ Error rate SLO ကို percentage (1%) အဖြစ် သတ်မှတ်ထားသည်။

**Expected behavior:** `['LATENCY_BREACH', 'ERROR_RATE_BREACH']` ကို ထုတ်ပေးသည် — latency အချို့ 500 ms ကျော်ပြီး error rate က 10% ဖြစ်နေသောကြောင့်။

## လေ့ကျင့်ခန်း ၆ — On-call Playbook တည်ဆောက်ခြင်း

လက်တွေ့ production issue တစ်ခုအတွက် on-call playbook document ရေးသားခြင်း။ အောက်ပါ section များပါဝင်သော `playbook.md` file တည်ဆောက်ပါ — (၁) alert name နှင့် severity၊ (၂) လက်တွေ့ symptom ဖော်ပြချက်၊ (၃) triage ဆင့်များ (log ကြည့် → Sentry issue ဖွင့် → deployment စစ် → rollback ဆုံးဖြတ်)၊ (၄) escalation path (ဘယ်သူ့ကို ဘယ်အချိန်မှာ ခေါ်မလဲ)၊ (၅) post-incident note template။ ထို့နောက် အောက်ပါ alert payload ကို playbook နှင့် တွဲဖက်စမ်းကြည့်ပါ။

```json
{
  "alert": "HighP95Latency",
  "severity": "critical",
  "service": "llm-api",
  "p95_latency_ms": 3200,
  "error_rate": 0.04,
  "last_deploy": "2024-06-01T09:00:00Z"
}
```

**Hints:** Triage ဆင့်တိုင်းတွင် "စစ်ဆေးရမည့် command / URL" ကို တိကျစွာ ရေးပါ (ဥပမာ — log query string, Sentry issue link pattern)။ Langfuse (langfuse.com/docs) ကဲ့သို့ tracing tool ၌ trace များကို ဘယ်လိုကြည့်ရမလဲဆိုသည်ကိုလည်း ထည့်သွင်းပါ။

**Expected behavior:** On-call ဝင်စ လူတစ်ယောက်က document ကို ဖတ်ရုံနှင့် ၁၅ မိနစ်အတွင်း လိုအပ်သော စစ်ဆေးချက်များ ပြုလုပ်နိုင်စေရန် လုံလောက်သော၊ command များနှင့် escalation နံပါတ်များ ပါဝင်သော playbook ရလဒ်ရှိသည်။
