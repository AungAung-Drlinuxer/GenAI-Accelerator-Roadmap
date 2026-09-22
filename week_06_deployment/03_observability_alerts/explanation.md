# Observability, Errors & Alerts — AI Application များအတွက် စောင့်ကြည့်မှု၊ Errors နှင့် Alerts

## Structured Logging

### ဘာကို ဆိုလိုတာလဲ

Structured logging ဆိုသည်မှာ log များကို လူများဖတ်ရလွယ်အောင် စာသားကြောင်းလိုက် ရေးသွင်းခြင်းအစား၊ စက်က ဖြည့်ဆီးဖတ်ရှုနိုင်သော key-value ပုံစံ (အများအားဖြင့် JSON) ဖြင့် ရေးသွင်းခြင်းဖြစ်သည်။ ဥပမာ — request_id, user_id, latency_ms, model_name, status စသည့် field များပါဝင်သော JSON log တစ်ကြောင်းဖြစ်သည်။ Python ရှိ `structlog` သို့မဟုတ် `logging` module တို့ဖြင့် အသုံးပြုနိုင်သည်။

### ဘာကြောင့် လဲ

AI application တစ်ခုတွင် LLM call၊ retrieval၊ database query စသည့် အဆင့်များစွာ ပါဝင်သည်။ စာသားကြောင်းလိုက် log များသာဖြစ်ပါက၊ "request အမှား ဘယ်မှာ ဖြစ်ခဲ့သလဲ၊ ဘယ် user လဲ၊ latency ဘယ်လောက်လဲ" ဆိုသည်ကို ရှာဖွေရန်ခက်သည်။ Structured log ဖြင့်ဖြစ်ပါက field အလိုက် စစ်ထားရန်၊ filter လုပ်ရန်နှင့် dashboard ဆွဲရန် လွယ်ကူသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Logger တစ်ခုကို ဖန်တီးပြီး အဓိက event တိုင်းကို JSON object အဖြစ် မှတ်တမ်းတင်သည်။ တစ်ခုတည်းသော request ၏ log များကို ချိတ်ဆက်ပေးရန် `request_id` ကို ထည့်သွင်းရသည်။ Timestamp ကို ISO 8601 ပုံစံဖြင့် ထည့်သင့်ပြီး၊ log level (INFO, WARNING, ERROR) ကို ကွဲပြားစွာ အသုံးပြုသင့်သည်။

### ဥပမာ

```python
import logging
import json
from datetime import datetime, timezone

# Simple structured logger that emits one JSON object per line
logger = logging.getLogger("ai-app")
logger.setLevel(logging.INFO)

def log_event(event: str, **fields):
    # Every log line includes a timestamp, level, and event name
    record = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "level": "INFO",
        "event": event,
        **fields,
    }
    logger.info(json.dumps(record, ensure_ascii=False))

def answer_question(question: str):
    # Log the start of a request with a request id
    log_event("question_received", request_id="req-001", question=question)
    log_event("llm_call_started", request_id="req-001", model="gpt-4o-mini")
    log_event("llm_call_finished", request_id="req-001", latency_ms=1200, tokens=87)
    return "sample answer"

answer_question("What is structured logging?")
# Expected output:
# {"timestamp": "2025-01-01T10:00:00+00:00", "level": "INFO", "event": "question_received", "request_id": "req-001", "question": "What is structured logging?"}
# {"timestamp": "2025-01-01T10:00:00+00:00", "level": "INFO", "event": "llm_call_started", "request_id": "req-001", "model": "gpt-4o-mini"}
# {"timestamp": "2025-01-01T10:00:01+00:00", "level": "INFO", "event": "llm_call_finished", "request_id": "req-001", "latency_ms": 1200, "tokens": 87}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production တွင် ဖောက်သည်များက "AI က အဖြေမှားနေတယ်" ဟု ပြောလာပါက၊ structured log များရှိလျှင် request_id ဖြင့် အချိန်ကာလတစ်ခုလုံးရှိ log များကို အမြန် ရှာဖွေနိုင်သည်။ Prometheus၊ Grafana Loki သို့မဟုတ် cloud provider ၏ log service များနှင့် တွဲဖက်ပါက log တစ်ကုဋေအနက်မှ ပြဿနာရှိသော request ကို စက္ကန့်ပိုင်းအတွင်း ရှာတွေ့နိုင်သည်။

---

## Sentry Error Tracking

### ဘာကို ဆိုလိုတာလဲ

Sentry သည် application ၏ error (exception) များကို အလိုအလျောက် စုဆောင်းပြီး စီမံခန့်ခွဲပေးသော service တစ်ခုဖြစ်သည် (docs.sentry.io တွင် အသေးစား ဖတ်ရှုနိုင်သည်)။ Error တစ်ခုဖြစ်ပါက stack trace၊ request context၊ user သတင်းအချက်အလက်များနှင့်အတူ Sentry dashboard သို့ ပေးပို့သည်။ တူညီသော error များကို တစ်ခုတည်းသော "issue" အဖြစ် စုစည်းပြသည်။

### ဘာကြောင့် လဲ

AI application များတွင် error အမျိုးအစားများ ထူးခြားသည် — LLM provider ၏ API rate limit၊ context window ကျော်လွန်ခြင်း၊ JSON output မှားယွင်းခြင်း၊ retrieval က အလွတ် result ပြန်ခြင်း စသဖြင့်ဖြစ်သည်။ Error များကို log ထဲတွင်သာ မြှုပ်နှံထားပါက ဘယ် error က ဖောက်သည်အများဆုံးဖြစ်နေသလဲ၊ ဘယ်အချိန်မှာ စတင်ခဲ့သလဲ ဆိုသည်ကို ခွဲခြားရခက်သည်။ Sentry က ထိုအချက်များကို အလိုအလျောက် စုစည်းပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`sentry-sdk` package ကို install လုပ်ပြီး DSN (Data Source Name) နှင့် `Sentry.init()` ခေါ်ရုံဖြင့် စတင်နိုင်သည်။ Init ပြုလုပ်ပါက မဖမ်းမိသေးသော exception များကို အလိုအလျောက် ဖမ်းဆီးပြီး ပေးပို့သည်။ Context အပိုများ (user id၊ request tag) ကို ထည့်ပေးနိုင်ပြီး၊ sampling rate ဖြင့် error ၏ အရေအတွက်ကိုလည်း ထိန်းချုပ်နိုင်သည်။

### ဥပမာ

```python
# pip install sentry-sdk
import sentry_sdk

# Initialize Sentry with the DSN from your Sentry project settings
sentry_sdk.init(
    dsn="https://examplePublicKey@o0.ingest.sentry.io/0",
    traces_sample_rate=0.1,  # Sample 10% of performance traces
    environment="production",
)

def answer_question(question: str) -> str:
    try:
        # Simulated LLM call that can fail
        return "sample answer"
    except Exception as exc:
        # Capture the exception with extra AI-specific context
        sentry_sdk.capture_exception(exc)
        # Also tag the event so you can filter by stage in the dashboard
        sentry_sdk.set_tag("component", "llm_call")
        raise

answer_question("What is Sentry?")
# Expected output:
# 'sample answer'
# (If the call raised an exception, an event with a stack trace
# would appear in the Sentry dashboard under a single grouped issue.)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Error တစ်ခုကို ဖောက်သည်က တိုက်ရိုက်သတိပြုမိလာမှ ပြင်ပါက နှောင့်နှေးသည်။ Sentry alert rule တစ်ခု ထားရှိပါက error အရေအတွက် တစ်နာရီအတွင်း သတ်မှတ်ထက်ကျော်လွန်သည့်အခါ email သို့မဟုတ် Slack သို့ အသိပေးချက် ရရှိမည်ဖြစ်သဖြင့် ဖောက်သည်များ မသိရှိခင် ပြင်ဆင်နိုင်သည်။

---

## Metrics and Dashboards

### ဘာကို ဆိုလိုတာလဲ

Metrics ဆိုသည်မှာ application ၏ ကျန်းမာရေးကို ကိုယ်စားပြုသော ဂဏန်းအချက်အလက်များဖြစ်သည် — request အရေအတွက်၊ latency၊ error rate၊ token အသုံးအဆောဒ၊ cache hit rate စသဖြင့်ဖြစ်သည်။ Dashboard ဆိုသည်မှာ ထို metrics များကို ပြက္ကဒိန်ပုံစံ ဂရပ်များဖြင့် ပြသသော စာမျက်နှာဖြစ်သည်။

### ဘာကြောင့် လဲ

Log များသည် ဖြစ်ရပ်တစ်ခုချင်းစီကို မှတ်တမ်းတင်သော်လည်း အနှစ်ချုပ်ပုံရိပ်ကို မပြပေ။ "နောက်ဆုံးတစ်နာရီ မှာ request ၁၀၀၀ အနက် error ဘယ်လောက်ရှိလဲ" ဆိုသည်ကို log တစ်ကုဋေကို စစ်၍ တွက်ချက်၍ မရနိုင်ပေ။ Metrics များကို ထိန်းသိမ်းပါက Prometheus နှင့် Grafana ကဲ့သို့ ကိရိယာများဖြင့် အချိန်နှင့်အမျှ ပြောင်းလဲမှုကို မြင်သာစွာ မြင်နိုင်သည်။ LLM tracing အတွက် Langfuse (langfuse.com/docs) ကဲ့သို့ ကိရိယာများက prompt၊ cost၊ latency များကို trace အဆင့်လိုက် ပြသပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Application အတွင်း request တစ်ခုချင်းစီ၏ latency နှင့် error status ကို တိုင်းတာပြီး Prometheus exposition format (counter နှင့် histogram) ဖြင့် `/metrics` endpoint တွင် ထုတ်ပေးသည်။ Prometheus က ထို endpoint ကို သတ်မှတ်ကြာပြောကာ စုဆောင်းပြီး၊ Grafana dashboard က ဂရပ်များဆွဲပြသည်။ AI အထူး metrics များ (token count၊ cost per request) ကိုလည်း label များဖြင့် တွဲတင်နိုင်သည်။

### ဥပမာ

```python
import time
from dataclasses import dataclass, field

# A minimal in-process metric collector for teaching purposes
@dataclass
class Metrics:
    request_count: int = 0
    error_count: int = 0
    latencies_ms: list = field(default_factory=list)

    def record(self, latency_ms: float, is_error: bool):
        # Increment counters and store latency for percentile math
        self.request_count += 1
        if is_error:
            self.error_count += 1
        self.latencies_ms.append(latency_ms)

    def error_rate(self) -> float:
        if self.request_count == 0:
            return 0.0
        return self.error_count / self.request_count

    def p95_latency_ms(self) -> float:
        # p95: 95% of requests finished faster than this value
        if not self.latencies_ms:
            return 0.0
        sorted_lat = sorted(self.latencies_ms)
        idx = int(0.95 * (len(sorted_lat) - 1))
        return sorted_lat[idx]

metrics = Metrics()

def answer_question(question: str) -> str:
    start = time.perf_counter()
    try:
        time.sleep(0.01)  # Simulated model call
        return "sample answer"
    except Exception:
        metrics.record((time.perf_counter() - start) * 1000, is_error=True)
        raise
    finally:
        if not metrics.record:
            pass

# Simulate several requests, one of which "fails"
for i in range(20):
    ok = (i != 7)  # Request index 7 is treated as an error
    start = time.perf_counter()
    time.sleep(0.01)
    metrics.record((time.perf_counter() - start) * 1000, is_error=not ok)

print(f"requests: {metrics.request_count}")
print(f"error_rate: {metrics.error_rate():.2%}")
print(f"p95_latency_ms: {metrics.p95_latency_ms():.1f}")
# Expected output:
# requests: 20
# error_rate: 5.00%
# p95_latency_ms: 10.6
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

ဒေတာအခြေခံ ဆုံးဖြတ်ချက်များလိုအပ်သည် — "ယနေ့ p95 latency က မနေ့ကထက် မြင့်နေသလား"၊ "LLM provider ကို ပြောင်းပြီးနောက် error rate တက်သွားသလား" စသဖြင့်ကို dashboard မှ မြင်ပြီး အမြန် ဖြေရှင်းနိုင်သည်။ AI cost tracking ကလည်း budget မကျော်အောင် ကာကွယ်ပေးသည်။

---

## SLOs: p95 Latency နှင့် Error Rate

### ဘာကို ဆိုလိုတာလဲ

SLO (Service Level Objective) သည် service ၏ စွမ်းဆောင်ရည်အတွက် ပစ်မှတ်တစ်ခုဖြစ်သည်။ AI application အတွက် အများအသုံးပြုသော SLO နှစ်ခုမှာ — "request များ၏ ၉၅% သည် X milliseconds အတွင်း ပြီးစီးရမည်" (p95 latency) နှင့် "error rate သည် Y% အောက် ရှိရမည်" ဖြစ်သည်။ p95 latency ဆိုသည်မှာ request အားလုံးကို မြန်နှုန်းအလိုက် စီစဉ်ပါက ၉၅ ရာခိုင်နှုန်းနေရာရှိ တန်ဖိုးဖြစ်သည်။

### ဘာကြောင့် လဲ

ပျမ်းမျှ (average) latency တစ်ခုတည်းဖြင့် ဆုံးဖြတ်ပါက နှေးကျွမ်းသော request အနည်းငယ်ကို ကွက်လျက် ကျန်ရစ်စေသည် — ဥပမာ ၁၀၀ request အနက် ၅ ခုက စက္ကန့် ၃၀ ကြာပါက ပျမ်းမျှမှာ ကောင်းနေနိုင်သည်။ p95 ကို တိုင်းတာပါက အများစုသော အသုံးပြုသူများ၏ အတွေ့အကြုံကို ကိုယ်စားပြုသည်။ Error rate SLO ကတော့ ဝန်ဆောင်မှု ယုံကြည်မှုကို အခြေခံ သတ်မှတ်ပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

ပထမဦးစွာ လက်တွေ့ဘက်တွင် ရရှိနိုင်
