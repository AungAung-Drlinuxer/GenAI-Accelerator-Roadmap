# Background Jobs with Celery + Redis

Week 3 အတွက် ဒီ module မှာ Celery နှင့် Redis ကို အသုံးပြုပြီး background job များကို မည်သို့ စီမံခန့်ခွဲမည်ကို လေ့လာပါမည်။ AI application တွေမှာ LLM ကိုခေါ်သည့် အလုပ်များသည် ခဏခဏ ကြာမြင့်တတ်သဖြင့် HTTP request path မှ ခွဲထုတ်ပြီး အရေးကြီးသည့် နည်းပညာဖြစ်သည်။ (Reference: docs.celeryq.dev, redis.io/docs)

## Task Queue ဆိုတာ ဘာလဲ

### ဘာကို ဆိုလိုတာလဲ
Task queue ဆိုသည်မှာ ဆော့ဖ်ဝဲအလုပ်တွေကို "နောက်မှ လုပ်မည်" ဟု စာတန်းတန်းလိုက် သိမ်းဆည်းထားသည့် စနစ်ဖြစ်သည်။ Web request တစ်ခုလာသောအခါ အချိန်ကြာသည့်အလုပ်ကို ချက်ချင်း လုပ်စေမည် မဟုတ်ဘဲ queue ထဲသို့ ထည့်ပြီး "task" တစ်ခုအဖြစ် မှတ်တမ်းတင်သည်။ Celery သည် Python world တွင် အသုံးအများဆုံး task queue framework ဖြစ်ပြီး၊ Redis သည် message broker အဖြစ် အလုပ်တွေကို သိမ်းဆည်းပေးသည့် in-memory data store ဖြစ်သည်။

### ဘာကြောင့် လဲ
LLM generation တစ်ခုသည် စက္ကန့်အတော်များများ ကြာနိုင်သည်။ HTTP request တစ်ခုအတွင်းမှာ စောင့်နေပါက server ၏ worker thread တွေ ပိတ်သွားပြီး အခြား user တွေရဲ့ request တွေပါ နှောင့်နှေးသွားနိုင်သည်။ ထို့အပြင် request timeout ကြောင့် အလုပ်မပြီးမီ ဆက်သွယ်မှု ပြတ်တောက်နိုင်သည်။ Task queue ကို အသုံးပြုခြင်းအားဖြင့် user ချက်ချင်း တုံ့ပြန်မှုရပြီး၊ အလုပ်ကြီးကတော့ နောက်ကွယ်မှ ဆက်လက် လုပ်ဆောင်သွားမည်ဖြစ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Celery architecture တွင် အစိတ်အပိုင်း သုံးခုရှိသည် — (1) **Producer** သည် task ကို queue ထဲထည့်သည်၊ (2) **Broker** (Redis) သည် စာတန်းကို သိမ်းဆည်းထားသည်၊ (3) **Worker** သည် queue မှ task ကို ယူ၍ လုပ်ဆောင်သည်။ Worker များကို တစ်ပြိုင်တည်း အများအပြား ဖွင့်ထားနိုင်သည်ကို "worker pool" ဟုခေါ်သည်။ Pool အမျိုးအစားများတွင် prefork (process အခြေပြု)၊ gevent (lightweight concurrency) စသည်တို့ပါဝင်ပြီး CPU-heavy အလုပ်အတွက် prefork ကို အသုံးများသည်။

### ဥပမာ

```python
# celery_app.py - define a Celery app backed by Redis
from celery import Celery

# Redis as broker (where tasks are queued) and backend (where results are stored)
app = Celery(
    "ai_jobs",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/1",
)

@app.task
def summarize_text(text: str) -> str:
    # Simulate a slow summarization job
    return f"Summary of: {text[:20]}..."

# Producer side (e.g., inside a Flask/FastAPI view):
result = summarize_text.delay("A very long document about AI engineering...")
print(result.id)
# Expected output: a task id like "8f5c2e1a-3b7d-4c9e-9a1f-2d4e5f6a7b8c"
```

Worker ကို စတင်ရန် `celery -A celery_app worker --loglevel=info -c 4` ဟု ရိုက်ရမည် (`-c 4` သည် concurrent worker လေးခု ဖွင့်ခြင်းဖြစ်သည်)။

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production AI service များတွင် user တစ်ယောက်ရဲ့ document summarization request က အခြား user တွေရဲ့ health check ရိုးရိုးလေးကိုပါ နှေးသွားစေလို့ မရပါ။ Queue ကို ခွဲခြားထားခြင်းအားဖြင့် လမ်းခွဲများ သီးသန့်ဖြစ်ပြီး တစ်ဦး၏အလုပ်ကြီးက အခြားသူများကို မထိခိုက်စေပါ။

## Retry နှင့် Backoff

### ဘာကို ဆိုလိုတာလဲ
Retry ဆိုသည်မှာ task တစ်ခု ပျက်သွားပါက ထပ်မံ ကြိုးစားခြင်းဖြစ်သည်။ Backoff ဆိုသည်မှာ retry တိုင်း စောင့်ဆိုင်းချိန်ကို တဖြည်းဖြည်း တိုးပွားစေခြင်းဖြစ်သည် — ဥပမာ ၁ စက္ကန့်၊ ၂ စက္ကန့်၊ ၄ စက္ကန့် စသဖြင့် exponential ပုံစံဖြင့် တိုးသည်။ Celery တွင် `retry_backoff` parameter ဖြင့် ဒီအပြုအမူကို သတ်မှတ်နိုင်သည်။

### ဘာကြောင့် လဲ
LLM API တွေသည် network နှင့် rate limit ကြောင့် ရံဖန်ရံခါ ယာယီပျက်စီးတတ်သည်။ ချက်ချင်း retry လုပ်ပါက rate limit ကို ပို၍ ဆိုးရွားစေနိုင်သည်၊ မှာယူရန် ပို၍ ဝန်ထုပ်ဝန်ပိုးဖြစ်စေသည်။ Backoff ဖြင့် စောင့်ဆိုင်းချိန် တိုးလာသောကြောင့် service က ပြန်လည်ထူထောင်ချိန် ရရှိစေပြီး storm သဖွယ် retry အများအပြား မဖြစ်ပွားစေပါ။

### ဘယ်လို အလုပ်လုပ်လဲ
Celery task အတွင်းရှိ exception တစ်ခုသည် `autoretry_for` စာရင်းထဲ ပါဝင်ပါက Celery သည် ထပ်မံ ကြိုးစားရန် task ကို ပြန်ထည့်သည်။ Retry အကြိမ်ရေနှင့် ကန့်သတ်ချိန်ကို `max_retries` နှင့် `retry_backoff_max` ဖြင့် ထိန်းချုပ်နိုင်သည်။ အကြိမ်အရေအတွက် ကုန်လျှင် task သည် အောင်မြင်မှုမရှိသည့် အနေအထားဖြင့် ပျက်သွားမည်ဖြစ်သည်။

### ဥပမာ

```python
import time
import random
from celery_app import app

@app.task(
    bind=True,
    max_retries=5,
    autoretry_for=(ConnectionError,),
    retry_backoff=2,        # wait 2, 4, 8, 16, 32 seconds between attempts
    retry_backoff_max=60,   # never wait longer than 60 seconds
    retry_jitter=True,      # add random jitter so workers do not retry in sync
)
def call_llm_api(prompt: str) -> str:
    # Simulate a flaky external API call
    if random.random() < 0.3:
        raise ConnectionError("LLM provider temporarily unreachable")
    return f"LLM response for: {prompt}"

# Retries only help for TRANSIENT errors; invalid input
# should fail immediately without retrying.
# Expected output: task succeeds after 0-5 attempts, each with growing delay.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Transient failure နှင့် permanent failure ကို ခွဲခြားနားလည်ခြင်းသည် production တွင် အလွန်အရေးကြီးသည်။ Retry policy မကောင်းပါက API provider ကို ပို၍ ဖိစီးပြီး၊ ကောင်းသော policy ကတော့ availability ကို မြှင့်တင်ပေးသည်။ Jitter ထည့်သွင်းခြင်းအားဖြင့် worker အများအပြား တစ်ပြိုင်တည် မတိုက်ခိုက်မိစေပါ။

## Idempotency နှင့် Dead-Letter စီမံခန့်ခွဲမှု

### ဘာကို ဆိုလိုတာလဲ
Idempotency ဆိုသည်မှာ task တစ်ခုကို အကြိမ်ကြိမ် လုပ်ဆောင်သည့်တိုင်း ရလဒ်တူညီစေခြင်းဖြစ်သည်။ Idempotency key သည် ထိုကို ထောက်ပံ့သည့် unique identifier ဖြစ်သည်။ Dead-letter ဆိုသည်မှာ retry အားလုံး ကုန်ပြီး နောက်ဆုံး ပျက်သွားသည့် task များကို သီးသန့် နေရာတွင် သိမ်းဆည်းခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Message broker တွေသည် "at-least-once" delivery ပေးတတ်သဖြင့် task တစ်ခုကို နှစ်ကြိမ်ရရှိနိုင်သည်။ Task ထဲမှာ email ပို့ခြင်း၊ ငွေတောင်းခံခြင်းကဲ့သို့ side-effect ပါဝင်ပါက နှစ်ကြိမ် လုပ်ဆောင်မိပါက ပြဿနာကြီးဖြစ်သည်။ ပျက်သွားသော task တွေကို တိတ်တဆိုင် ပစ်ထားပါက စုံစမ်းစစ်ဆေးရန် အခွင့်အလမ်း ဆုံးရှုံးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Idempotency key ကို Redis ထဲ SET NX (set-if-not-exists) ဖြင့် သိမ်းပြီး ဒုတိယအကြိမ် ရောက်လာပါက skip လုပ်စေသည်။ Celery မှာ dead-letter ကို built-in မပေးထားသော်လည်း `on_failure` handler သို့မဟုတ် သီးသန့် "failed" queue ထဲသို့ ကိုယ်တိုင် ပို့နိုင်သည်။ Redis Streams ကိုလည်း failed task များ မှတ်တမ်းတင်ရန် အသုံးပြုနိုင်သည်။

### ဥပမာ

```python
import json
from celery_app import app

@app.task(bind=True, max_retries=3)
def generate_report(self, user_id: int, idempotency_key: str) -> str:
    lock_key = f"idem:{idempotency_key}"
    # SET NX returns True only the first time this key is stored
    if not app.backend.client.set(lock_key, "in-progress", nx=True, ex=3600):
        print(f"Task {idempotency_key} already processed, skipping.")
        return "skipped"

    try:
        # heavy LLM report generation happens here
        result = f"report for user {user_id}"
        app.backend.client.set(lock_key, "done", ex=3600)
        return result
    except Exception as exc:
        # all retries exhausted -> record for dead-letter inspection
        app.backend.client.rpush(
            "dead_letter_queue",
            json.dumps({"task": self.name, "key": idempotency_key,
                        "args": [user_id], "error": str(exc)}),
        )
        raise

# Expected output: duplicate deliveries print "skipped" instead of re-running.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
ဒုတိယအကြိမ် လုပ်ဆောင်မိခြင်းက ငွေကြေးအရှုံးပေါက်နိုင်ပြီး၊ ပျက်သွားသော task တွေကို လေ့လာနိုင်ခြင်းက system ၏ အားနည်းချက်ကို ဖော်ပြပေးသည်။ Dead-letter queue ကို ပုံမှန် စစ်ဆေးခြင်းအားဖြင့် ကြိုတင်သတိပေးချက်များ တည်ဆောက်နိုင်သည်။

## Scheduling နှင့် Long-Running LLM Jobs

### ဘာကို ဆိုလိုတာလဲ
Scheduling သည် task များကို သတ်မှတ်ချိန်အလိုက် အလိုအလျောက် လုပ်ဆောင်စေခြင်းဖြစ်သည် — ဥပမာ ညတိုင်း cron လုပ်ဆောင်ချက်ကဲ့သို့။ Celery Beat သည် ဒီကိစ္စအတွက် scheduler ဖြစ်သည်။ Long-running job ဆိုသည်မှာ မိနစ်ပေါင်းများစွာ ကြာနိုင်သော LLM အလုပ်ကြီးများကို ဆိုလိုသည်။

### ဘာကြောင့် လဲ
Document re-indexing၊ model batch evaluation ကဲ့သို့သော အလုပ်များကို လူက လက်ဖြင့် လုပ်ရန် မသင့်ပါ။ နောက်ခံ scheduled job များဖြင့် အလိုအလျောက် လုပ်ဆောင်စေခြင်းက operational ဝန်ထုပ်ဝန်ပိုးကို လျှော့ချပေးသည်။ Request path မှ ခွဲထုတ်ခြင်းက user experience ကို အာမခံပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Celery Beat သည် schedule config အတိုင်း သတ်မှတ်ချိန်ရောက်သည့်အခါ broker ထဲသို့ task အသစ်ထည့်ပေးသည်။ Long-running job တစ်ခုအတွက် လုပ်ငန်းစဉ်ကို အဆင့်ဆင့် ခွဲပြီး status ကို Redis ထဲ မှတ်တမ်းတင်ထားပါက user က progress ကို စစ်ဆေးနိုင်သည် — ဥပမာ polling endpoint တစ်ခုဖြင့်။

### ဥပမာ

```python
from celery_app import app
from celery.schedules import crontab
import redis

r = redis.Redis(host="localhost", port=6379, db=1)

@app.on_after_configure.connect
def setup_periodic_tasks(sender, **kwargs):
    # Runs every night at 2:30 AM
    sender.add_periodic_task(
        crontab(hour=2, minute=30),
        reindex_documents.s(),
    )

@app.task(bind=True)
def reindex_documents(self):
    doc_ids = ["doc1", "doc2", "doc3", "doc4"]
    total = len(doc_ids)
    job_id = self.request.id

    for i, doc_id in enumerate(doc_ids):
        # Simulate slow embedding generation for each document
        _generate_embedding(doc_id)
        # Store progress in Redis so the user can poll it
        r.hset(f"job:{job_id}", mapping={
            "status": "running",
            "processed": i + 1,
            "total": total,
        })

    r.hset(f"job:{job_id}", mapping={
        "status": "completed",
        "processed": total,
        "total": total,
    })
    return {"job_id": job_id, "indexed": total}

def _generate_embedding(doc_id):
    # Placeholder for a slow LLM embedding call
    pass
```

Progress ကို စစ်ဆေးရန် polling endpoint တစ်ခုကို အောက်ပါအတိုင်း ရေးနိုင်သည် —

```python
from flask import Flask, jsonify

api = Flask(__name__)

@api.get("/jobs/<job_id>/status")
def job_status(job_id):
    data = r.hgetall(f"job:{job_id}")
    if not data:
        return jsonify({"error": "job not found"}), 404
    return jsonify({
        "status": data[b"status"].decode(),
        "processed": int(data[b"processed"]),
        "total": int(data[b"total"]),
    })
```

### အားသာချက်များနှင့် သတိပြုရန်များ
Scheduled task များက idempotent ဖြစ်ရမည် — ဆိုလိုသည်မှာ အကြိမ်ကြိမ် run သော်လည်း ရလဒ် တူညီရမည်။ Long-running job များကို ခွဲချင်း သေချာထားရမည်မှာ — worker ပျက်သွားပါက ပြန်စနိုင်ရန် checkpoint များ သိမ်းဆည်းထားသင့်သည်။ Celery Beat ကို single instance အဖြစ်သာ run ရမည် — မဟုတ်ပါက task များ ထပ်မံ deliver ဖြစ်နိုင်သည်။

### အနှစ်ချုပ်
Scheduling နှင့် long-running job များသည် production LLM system များ၏ မလွဲမသော အစိတ်အပိုင်းများဖြစ်သည်။ Celery Beat က အချိန်သတ်မှတ် လုပ်ဆောင်ချက်များကို အလိုအလျောက် စီမံပေးပြီး Redis-backed progress tracking က user များအား အလုပ်ကြီးများ၏ အခြေအနေကို မြင်တွေ့စေသည်။ ဤနှစ်ခုကို ပေါင်းစပ်အသုံးပြုခြင်းဖြင့် စနစ်ကို တိကျစွာ၊ အလိုအလျောက် လည်ပတ်နိုင်သော ပုံစံသို့ ရောက်ရှိစေနိုင်သည်။
