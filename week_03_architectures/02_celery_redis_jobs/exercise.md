## လေ့ကျင့်ခန်း ၁ — Celery App နှင့် Redis Broker အခြေခံ

`celery_app.py` ဖိုင်တစ်ခု ရေးပါ။ Broker နှင့် result backend နှစ်ခုစလုံးအတွက် Redis ကို အသုံးပြုပါ။ အလွယ်တစ်ခုဖြစ်သော `add(x, y)` task ကို define လုပ်ပါ။ ထို့နောက် terminal နှစ်ခုဖွင့်ကာ — တစ်ခုတွင် worker ကို `celery -A celery_app worker --loglevel=info` ဖြင့် run ပြီး 另一个တစ်ခုမှ Python shell အတွင်း `add.delay(3, 4)` ကို ခေါ်ပါ။

```python
# celery_app.py
from celery import Celery

# Use Redis as both broker and result backend
app = Celery(
    "myapp",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/1",
)

@app.task
def add(x, y):
    # Simple task that returns the sum of two numbers
    return x + y
```

**Hints:** Redis ကို `docker run -p 6379:6379 redis:7` ဖြင့် စတင်နိုင်သည်။ `pip install celery redis` လိုအပ်သည်။ `.delay()` က `AsyncResult`  object ပြန်သည်။
**Expected behavior:** Worker log တွင် task လက်ခံရရှိကြောင်း မြင်ရပြီး shell မှ `.get(timeout=5)` ဖြင့် `7` ကို ရရှိသည်။

## လေ့ကျင့်ခန်း ၂ — Retry နှင့် Exponential Backoff

ပြင်ပ service ကို ခေါ်သည့် `fetch_price(symbol)` task တစ်ခု ရေးပါ။ Network error ဖြစ်လာပါက `autoretry_for` ဖြင့် အလိုအလျောက် retry လုပ်စေပါ။ Backoff policy အနေဖြင့် `retry_backoff=2` နှင့် `retry_backoff_max=60`, `max_retries=5` ကို သတ်မှတ်ပါ။ စမ်းသပ်ရန် URL အဖြစ် မရှိသော host တစ်ခုကို အသုံးပြုပါ။

```python
import requests

@app.task(
    bind=True,
    autoretry_for=(requests.exceptions.ConnectionError,),
    retry_backoff=2,          # 2, 4, 8, 16, 32 ... seconds
    retry_backoff_max=60,
    max_retries=5,
)
def fetch_price(self, symbol):
    # Deliberately unreachable host to observe retry behavior
    resp = requests.get(f"https://invalid.example.internal/{symbol}", timeout=5)
    return resp.json()
```

**Hints:** `bind=True` က `self` (task instance) ကို ပေးသည်။ Worker log တွင် `Retry in Xs` စာသားကို ကြည့်နိုင်သည်။
**Expected behavior:** Worker log အတွင်း နောက်ဆုံး `MaxRetriesExceededError` သို့ ရောက်သွားသည့်အထိ retry အကြိမ်ရေ တိုးလာသည်ကို မြင်ရပြီး delay တန်ဖိုးများသည် နှစ်ဆတိုးသွားသည်။

## လေ့ကျင့်ခန်း ၃ — Idempotency Key ဖြင့် Task ထပ်မံမှု ကာကွယ်ခြင်း

`charge_user(user_id, amount)` ဟုခေါ်သော ငွေဖြတ်တောင်းမှု task တစ်ခု ရေးပါ။ Caller ဘက်မှ idempotency key တစ်ခု ဖန်တီးပြီး task အတွင်း Redis `SET NX` ဖြင့် စစ်ဆေးပါ — key ရှိပြီးသားဖြစ်လျှင် ထပ်မဆောင်ရွက်ဘဲ ရှိပြီးသား result ကို ပြန်ပါ။

```python
import json
import uuid

@app.task
def charge_user(user_id, amount, idem_key):
    # SET NX returns True only if the key did not exist before
    claimed = app.backend.client.set(f"idem:{idem_key}", "processing", nx=True, ex=3600)
    if not claimed:
        # Same key seen before: return stored result instead of charging again
        return "already-processed"
    # Imagine a real payment call happens here
    result = {"user": user_id, "charged": amount}
    app.backend.client.set(f"idem:{idem_key}", json.dumps(result), ex=3600)
    return result
```

**Hints:** `app.backend.client` က Redis client ကို တိုက်ရိုက်ရရှိစေသည်။ အလားတူ `idem_key = str(uuid.uuid4())` ကို caller က ဖန်တီးပြီး တူညီသော key ဖြင့် task ကို နှစ်ကြိမ် ခေါ်ကြည့်ပါ။
**Expected behavior:** ပထမ ခေါ်ဆိုမှုတွင် result dict ပြန်ပြီး ဒုတိယ ခေါ်ဆိုမှုတွင် `"already-processed"` ကို ရရှိသည်။

## လေ့ကျင့်ခန်း ၄ — Celery Beat ဖြင့် Scheduled Jobs

`celery_app.py` အတွင်း beat schedule သတ်မှတ်ပြီး `cleanup_logs()` task ကို စက္ကန့် ၃၀ တစ်ကြိမ် အလိုအလျောက် run စေပါ။

```python
app.conf.beat_schedule = {
    "cleanup-every-30-seconds": {
        "task": "celery_app.cleanup_logs",
        "schedule": 30.0,
        "args": (),
    },
}

@app.task
def cleanup_logs():
    # Simulate log rotation: just print a timestamped line
    import datetime
    return f"logs cleaned at {datetime.datetime.utcnow():%Y-%m-%d %H:%M:%S}"
```

**Hints:** Beat process ကို သီးခြား run ရမည် — `celery -A celery_app beat` — တစ်ဖက်တွင် worker ဆက် run ထားပါ။ `beat_schedule` ကိu `app.conf` တွင် သတ်မှတ်ရမည်။
**Expected behavior:** Beat process က စက္ကန့် ၃၀ အကြား တစ်ကြိမ် တာဝန်ကို queue ထဲ တွန်းပို့ပြီး worker log တွင် ၃၀ စက္ကန့်ခန့် အကြာအလယ် `cleanup_logs` run နေသည်ကို မြင်ရသည်။

## လေ့ကျင့်ခန်း ၅ — Dead-Letter Queue ဖြင့် ကျန်ရှိ task များ ကိုင်တွယ်ခြင်း

Retry အားလုံး ကုန်ဆုံးပြီးနောက် ကျရောက်လာသော task များကို သီးခြား `dead_letter` queue ထဲ ပို့စေပါ။ Task အတွင်း `on_failure` handler တစ်ခု ရေးပြီး ဖော်ရှိမရသော task ကို `app.send_task("dead_letter", args=...)` ဖြင့် တစ်ဆင့်ပို့ပါ၊ `dead_letter` task မှာ မိမိသတင်းစကားကို log လုပ်ရုံသာ ဖြစ်စေပါ။

```python
@app.task(bind=True, max_retries=2, autoretry_for=(ValueError,),
          retry_backoff=1)
def parse_file(self, path):
    # Always fails: this task is our dead-letter test subject
    raise ValueError(f"cannot parse {path}")

@app.task
def dead_letter(task_name, args, exc):
    # Simply record that a task permanently failed
    print(f"DEAD LETTER: {task_name} args={args} error={exc}")
    return "recorded"

def route_to_dead_letter(task_name, args, exc):
    # Called manually or from on_failure to forward failed work
    app.send_task("celery_app.dead_letter",
                  args=[task_name, args, str(exc)])
```

**Hints:** `on_failure` ကို task class တွင် သတ်မှတ်နိုင်သည် — `class ParseTask(Task): def on_failure(self, exc, task_id, args, kwargs, einfo): ...` ဟူ၍ ရေးပြီး `base=ParseTask` ဖြင့် သုံးပါ။
**Expected behavior:** `parse_file.delay("bad.csv")` ကို ခေါ်ပြီး retry နှစ်ကြိမ် ကုန်ဆုံးသွားသောအခါ worker log တွင် `DEAD LETTER: ...` စာသား ထွက်ပေါ်လာသည်။

## လေ့ကျင့်ခန်း ၆ — LLM ကို Request Path အပြင်ဘက်မှ အလုပ်ရှုပ်စေခြင်း

FastAPI endpoint တစ်ခု ရေးပါ — `/summarize` POST request လက်ခံပြီး `summarize_text` Celery task ကို queue ထဲ တွန်းပိုးကာ `task_id` ကို ချက်ချင်း ပြန်ပါ။ အခြား endpoint `/result/{task_id}` မှ status (PENDING / STARTED / SUCCESS) နှင့် result ကို စစ်ဆေးနိုင်စေပါ။ `summarize_text` task အတွင်း `time.sleep(10)` ဖြင့် long-running LLM call ကို အတုယူပါ (စင်စစ် API ခေါ်ဆိုမှု မလိုပါ)။

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

**Hints:** `uvicorn api:api --reload` ဖြင့် server ကို run ပါ။ `res.ready()` က task ပြီး/မပြီး စစ်ပေးသည် — ဒါကြောင့် result ကို တန်း ထုတ်ယူခြင်းမှ ကာကွယ်သည်။ `time.sleep` ကို task အတွင်းထားပြီး worker log တွင် ၁၀ စက္ကန့် ကြာမြင့်နေသည်ကို လေ့လာပါ။
**Expected behavior:** `/summarize` request က ၁ စက္ကန့်အတွင်း `task_id` ပြန်သည် (web server မှာ blocking မဖြစ်)၊ `/result/{task_id}` က ပထမတွင် `PENDING` ပြပြီး ၁၀ စက္ကန့်အကြာတွင် `SUCCESS` နှင့် summary result ကို ပြသည်။
