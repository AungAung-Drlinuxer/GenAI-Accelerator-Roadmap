## လေ့ကျင့်ခန်း ၁ — Celery App နှင့် Redis Broker ဖြင့် အခြေခံ Task Queue

Celery application တစ်ခုကို Redis broker သုံးပြီး တည်ဆောက်ပုံကို ဤမှာ ကြည့်ပါ။ `celery_app.py` ဖိုင်အဖြစ် သိမ်းပြီး worker ကို `celery -A celery_app worker --loglevel=info` ဖြင့် စတင်နိုင်သည်။

```python
# celery_app.py
from celery import Celery

# Redis acts as both broker (message transport) and result backend
app = Celery(
    "ai_jobs",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/1",
)

# Serialize task arguments and results as JSON for portability
app.conf.update(
    task_serializer="json",
    result_serializer="json",
    accept_content=["json"],
)

@app.task
def generate_summary(text: str) -> str:
    # Placeholder for a real LLM call; keeps the example runnable offline
    return f"summary-of: {text[:50]}"

if __name__ == "__main__":
    # Calling .delay() pushes the task onto the Redis queue (async)
    result = generate_summary.delay("Long document text goes here")
    print("task id:", result.id)
    print("state:", result.status)  # PENDING or SUCCESS
```

**အဓိကအယူအဆ** — Task queue ဆိုသည်မှာ အလုပ်ရှင် process ကို Redis ကဲ့သို့ broker တစ်ခုမှတဆင့် worker process များဆီသို့ တာဝန်များ လွှဲပြောင်းပေးသည့် ယန္တရားဖြစ်သည်။

## လေ့ကျင့်ခန်း ၂ — Retry နှင့် Exponential Backoff

Network ချို့ယွင်းမှုများအတွက် retry policy နှင့် backoff ကို သတ်မှတ်ပုံ။

```python
# retry_tasks.py
import random
import time
from celery_app import app

@app.task(
    bind=True,
    max_retries=5,
    # Retry after 2^attempt seconds: 2, 4, 8, 16, 32
    default_retry_delay=2,
    autoretry_for=(ConnectionError,),
    retry_backoff=True,          # enable exponential backoff
    retry_backoff_max=60,        # cap the delay at 60 seconds
    retry_jitter=True,           # add randomness to avoid thundering herd
)
def call_external_api(self, prompt: str) -> dict:
    # Simulated flaky downstream service
    if random.random() < 0.4:
        raise ConnectionError("upstream API unreachable")
    return {"prompt": prompt, "ok": True}

@app.task(bind=True, max_retries=3)
def manual_retry(self, payload: dict) -> dict:
    try:
        if not payload.get("id"):
            raise ValueError("missing id")
        return {"processed": payload["id"]}
    except ValueError as exc:
        # Raise to trigger retry with custom backoff
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)
```

**အဓိကအယူအဆ** — Exponential backoff နှင့် jitter သည် upstream ပျက်နေချိန်တွင် တနေ့လျှောက် retry တောင်းခံမှုများကို လျှော့ပေးပြီး worker pool ကို အလွန်အမင်း မနှိပ်စေရန် ကာကွယ်သည်။

## လေ့ကျင့်ခန်း ၃ — Idempotency Key ဖြင့် Task ထပ်မံလုပ်ဆောင်မှု တားဆီးခြင်း

Task တစ်ခုပြန် run သွားပါက ရလဒ် နှစ်ဆယ်ဖြစ်မလာစေရန် Redis SETNX ဖြင့် စောင့်ကြည့်နည်း။

```python
# idempotent_tasks.py
import json
from celery_app import app

def make_key(user_id: str, action: str) -> str:
    # Deterministic key: same input always maps to the same Redis key
    return f"idem:{user_id}:{action}"

@app.task(bind=True, max_retries=2)
def charge_credits(self, user_id: str, amount: int) -> dict:
    redis = app.backend.client  # reuse the Redis connection
    key = make_key(user_id, "charge")
    # setnx stores the key only if it does not exist yet
    lock = redis.set(key, "1", nx=True, ex=3600)
    if not lock:
        # Duplicate delivery detected; skip without raising an error
        return {"skipped": True, "reason": "already processed"}
    try:
        # Real billing logic would go here
        return {"user_id": user_id, "charged": amount}
    except Exception:
        redis.delete(key)  # release the key so a retry can reprocess
        raise
```

**အဓိကအယူအဆ** — Idempotency key သည် တူညီသော task နှစ်ကြိမ် delivery ဖြစ်သွားခြင်း (at-least-once delivery) ကြောင့် ဖြစ်ပေါ်နိုင်သည့် side effect နှစ်ဆယ်ကို တားဆီးပေးသည်။

## လေ့ကျင့်ခန်း ၄ — Scheduling နှင့် Dead-Letter Handling

Periodic tasks နှင့် အကြိမ်များစွာ ကျရှုံးသော task များကို dead-letter queue ဆီ ပို့ပုံ။

```python
# scheduling_tasks.py
from celery import Celery
from celery.schedules import crontab
from celery_app import app

# Beat schedule: periodic jobs are dispatched by a separate beat process
app.conf.beat_schedule = {
    "nightly-cache-warmup": {
        "task": "scheduling_tasks.warm_cache",
        "schedule": crontab(hour=2, minute=0),  # every day at 02:00
    },
    "health-check": {
        "task": "scheduling_tasks.health_check",
        "schedule": 30.0,  # every 30 seconds
    },
}

@app.task
def warm_cache() -> str:
    return "cache warmed"

@app.task
def health_check() -> str:
    return "healthy"

@app.task(bind=True, max_retries=3, default_retry_delay=5)
def fragile_job(self, payload: dict) -> dict:
    try:
        # Placeholder that always fails for demonstration
        raise RuntimeError("simulated permanent failure")
    except RuntimeError as exc:
        if self.request.retries >= self.max_retries:
            # After exhausting retries, record the payload for inspection
            app.backend.client.rpush("dead_letter", str(payload))
            return {"dead_lettered": True}
        raise self.retry(exc=exc)

# Inspect dead letters with: redis-cli lrange dead_letter 0 -1
```

**အဓိကအယူအဆ** — Retry အကြိမ်အရေတွက် ကုန်ဆုံးသွားသော task များကို dead-letter list တွင် သိမ်းဆည်းခြင်းဖြင့် စနစ်ကို ဆက်လက်လည်ပတ်စေပြီး နောက်ပိုင်း စစ်ဆေးပြင်ဆင်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၅ — Long-Running LLM Jobs ကို Request Path မှ ခွဲထုတ်ခြင်း

Web request တွင် LLM generation ကို တိုက်ရိုက် မစောင့်စေဘဲ Celery worker ဆီ လွှဲပြီး job id ဖြင့် status စစ်နိုင်သည့် ပုံစံ။

```python
# llm_jobs.py
import time
from celery_app import app

@app.task(bind=True, max_retries=3, autoretry_for=(Exception,), retry_backoff=True)
def run_llm_generation(self, prompt: str) -> dict:
    # Long-running LLM call simulated with sleep; the web request returns
    # immediately with a task id instead of blocking on this work
    time.sleep(10)
    return {"prompt": prompt, "output": "generated-text-placeholder"}

# Example FastAPI-style caller logic (sketch, not a full web framework file)
def enqueue_generation(prompt: str) -> dict:
    async_result = run_llm_generation.delay(prompt)
    # Return the job id so the client can poll status later
    return {"job_id": async_result.id, "status": "queued"}

def get_job_status(job_id: str) -> dict:
    result = app.AsyncResult(job_id)
    response = {"job_id": job_id, "status": result.status}
    if result.ready():
        response["output"] = result.result
    return response

if __name__ == "__main__":
    info = enqueue_generation("Summarize this report")
    print(info)
    time.sleep(2)
    print(get_job_status(info["job_id"]))
```

**အဓိကအယူအဆ** — ကြာမြင့်သော LLM အလုပ်များကို HTTP request cycle ပြင်ပရှိ worker pool ဆီ ရေှ့တင်ပို့ခြင်းအားဖြင့် web server ၏ timeout နှင့် throughput ပြဿနာများကို ရှောင်ရှောင်ပေးသည်။

## လေ့ကျင့်ခန်း ၆ — LLM ကို Request Path အပြင်ဘက်မှ အလုပ်ရှုပ်စေခြင်း

```python
# celery_app.py
import time
from celery import Celery

app = Celery("tasks", broker="redis://localhost:6379/0", backend="redis://localhost:6379/1")

@app.task(name="summarize_text")
def summarize_text(text: str):
    # Simulate a slow LLM call so the web worker is never blocked
    time.sleep(10)
    return {"summary": text[:50] + "..." if len(text) > 50 else text}
```

```python
# api.py
from fastapi import FastAPI
from celery_app import app, summarize_text

api = FastAPI()

@api.post("/summarize")
def summarize(payload: dict):
    # Push work to Celery and return immediately with the task id
    async_result = summarize_text.delay(payload["text"])
    return {"task_id": async_result.id, "status": "queued"}

@api.get("/result/{task_id}")
def result(task_id: str):
    # Poll task status without blocking the web worker
    res = app.AsyncResult(task_id)
    return {"status": res.status, "result": res.result if res.ready() else None}
```

Server ကို စတင်ရန် command များ:

```bash
# Terminal 1 — start the Celery worker
celery -A celery_app.app worker --loglevel=info

# Terminal 2 — start the FastAPI server
uvicorn api:api --reload
```

အသုံးပြုပုံ စမ်းသပ်ရန် command:

```bash
# Queue a summary job and get the task id back instantly
curl -X POST http://127.0.0.1:8000/summarize \
    -H "Content-Type: application/json" \
    -d '{"text": "Myanmar is a country in Southeast Asia with a rich culture and long history."}'

# Poll the result while the worker is still sleeping
curl http://127.0.0.1:8000/result/<task_id>
```

**အဓိကအယူအဆ** — ကြာမြင့်စွာ အလုပ်လုပ်ရမည့် LLM ခေါ်ဆိုမှုကို Celery task အဖြစ် queue ထဲသို့ တန်းပို့လိုက်ပြီး `task_id` ကို ချက်ချင်း ပြန်ဖြင့် request path ကို အလုပ်ရှုပ်ခြင်းမှ ကင်းစင်စေကာ၊ အသုံးပြုသူက `/result/{task_id}` မှတစ်ဆင့် status နှင့် ရလဒ်ကို နောက်မှ ဝင်ရောက်စစ်ဆေးနိုင်သည်။
