# Week 3 — FastAPI Service Design for AI Backends

AI backend တစ်ခုကို FastAPI နဲ့ ဒီဇိုင်းလုပ်တဲ့အခါ အဓိက နားလည်ရမယ့် အချက်တွေကတော့ routers, dependency injection, async endpoints, streaming responses (SSE), request validation, error contract နဲ့ health endpoints တို့ ဖြစ်ပါတယ်။ Ref: fastapi.tiangolo.com

## 1. Routers

### ဘာကို ဆိုလိုတာလဲ
Router ဆိုတာ URL path တွေနဲ့ HTTP method (GET, POST စသဖြင့်) တွေကို စုစည်းပေးထားတဲ့ အုပ်စု တစ်ခုပါ။ FastAPI မှာ `APIRouter` class ကို အသုံးပြုပြီး endpoint တွေကို ဖိုင်တစ်ခုချင်းစီ ခွဲခြား ရေးသားနိုင်ပါတယ်။

### ဘာကြောင့် လဲ
Application တစ်ခုလုံးကို ဖိုင်တစ်ခုတည်းမှာ ရေးလိုက်ရင် ရှည်လျားပြီး ထိန်းသိမ်းရခက်ပါတယ်။ Router တွေက feature အလိုက် ကုဒ်ကို ခွဲပေးလို့ ဖိုင် အစီအစဉ် သန့်သန့်ရှင်းရှင်း ရှိစေပြီး team နဲ့ အလုပ်လုပ်တဲ့အခါမှာလည်း ပိုအဆင်ပြေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
`APIRouter()` တစ်ခုကို ဖန်တီးပြီး သူ့ထဲမှာ endpoint တွေကို သတ်မှတ်ပါတယ်။ အဲဒီ router ကို main application ထဲ `app.include_router()` နဲ့ တွဲသွင်းပါတယ်။ `prefix` နဲ့ `tags` parameter တွေက OpenAPI docs အတွက် အုပ်စုခွဲပေးပါတယ်။

### ဥပမာ

```python
from fastapi import FastAPI, APIRouter

app = FastAPI()
predictions_router = APIRouter(prefix="/predictions", tags=["predictions"])

@predictions_router.post("")
def create_prediction(text: str):
    # Simple endpoint grouped under the predictions router
    return {"input": text, "score": 0.0}

app.include_router(predictions_router)

# Expected output:
# POST /predictions  ->  {"input": "...", "score": 0.0}
# GET  /docs          ->  grouped under "predictions" tag
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI service တွေမှာ model inference, embedding, health, monitoring စတဲ့ endpoint အမျိုးမျိုး ရှိတတ်ပါတယ်။ Router တွေနဲ့ ခွဲထားရင် inference နဲ့ infrastructure endpoint တွေကို သီးခြား deploy၊ test လုပ်ဖို့ လွယ်ပြီး container orchestration (ဥပမာ Kubernetes) နဲ့ တွဲဖက်တဲ့အခါလည်း လမ်းကြောင်း သတ်မှတ်ရ ရှင်းပါတယ်။

## 2. Dependency Injection

### ဘာကို ဆိုလိုတာလဲ
Dependency injection (DI) ဆိုတာ endpoint တစ်ခု လိုအပ်တဲ့ object တွေ (ဥပမာ model client, settings, database session) ကို အဲဒီ endpoint ထဲကို FastAPI က အလိုအလျောက် ထည့်ပေးတဲ့ စနစ်ပါ။ `Depends()` နဲ့ သတ်မှတ်ပါတယ်။

### ဘာကြောင့် လဲ
ML model တစ်ခုကို load လုပ်ဖို့ အချိန်ကြာတတ်ပါတယ်။ Request တစ်ခါတိုင်း model ကို အသစ် load လုပ်နေရင် အလွန်ကြာပါမယ်။ DI က model ကို တစ်ခါတည်း ဖန်တီးပြီး ခေါ်သုံးမဲ့ endpoint တိုင်းမှာ မျှဝေ သုံးစွဲနိုင်စေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Function တစ်ခုကို ရေးပြီး endpoint ရဲ့ parameter ထဲမှာ `Depends()` နဲ့ ညွှန်းပါ။ FastAPI က function ရဲ့ return type အပေါ် မူတည်ပြီး အလိုအလျောက် ခေါ်ယူ အသုံးပြေးပေးပါတယ်။

### ဥပမာ

```python
from fastapi import Depends, FastAPI
from typing import Any

app = FastAPI()

class ModelContainer:
    def __init__(self):
        # Load model once at startup (simulate heavy resource)
        self.model: Any = "loaded-model"
        self.call_count = 0

    def predict(self, text: str) -> str:
        self.call_count += 1
        return f"prediction-for-{text}"

def get_model() -> ModelContainer:
    # Singleton container shared across requests
    return _model_container

_model_container = ModelContainer()

@app.post("/infer")
def infer(text: str, model: ModelContainer = Depends(get_model)):
    # FastAPI injects the same container instance every request
    return {"result": model.predict(text), "calls_so_far": model.call_count}

# Expected output:
# POST /infer?text=hello -> {"result": "prediction-for-hello", "calls_so_far": 1}
# POST /infer?text=world -> {"result": "prediction-for-world", "calls_so_far": 2}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
GPU memory, connection pool, API client တို့လို ရင်းနှီးမှု ကြီးမားတဲ့ အရာတွေကို DI နဲ့ စီမံခြင်းက production AI service ရဲ့ စွမ်းဆောင်ရည် အခြေခံ ဖြစ်ပါတယ်။ Testing အတွက်လည်း တကယ့် model အစား fake object ထည့်ပေးလို့ ရလို့ test ရေးရ ပိုလွယ်ပါတယ်။

## 3. Async Endpoints နှင့် Streaming Responses (SSE)

### ဘာကို ဆိုလိုတာလဲ
Async endpoint ဆိုတာ request တစ်ခုကို စောင့်နေစဉ်မှာ server က အခြား request တွေကိုပါ ဆက်လက် လုပ်ဆောင်ပေးနိုင်တဲ့ endpoint ပါ။ Streaming response (SSE — Server-Sent Events) က server က client ဆီ အပိုင်းအစ အားဖြင့် တဆက်တည်း ပို့ပေးတဲ့ နည်းပါ။ LLM output တွေကို token အလိုက် တိုက်ရိုက် မြင်ရစေဖို့ အသုံးများပါတယ်။

### ဘာကြောင့် လဲ
AI model inference က အချိန်ကြာတတ်ပါတယ်။ Async မသုံးရင် request တစ်ခု မပြီးမချင်း နောက် request တွေ တစ်ခါတည်း ပိတ်သွားနိုင်ပါတယ်။ LLM က generated text ကို တစ်ပြိုင်တည်း မပို့နိုင်တော့ user က ကြာမြင့်စွာ စောင့်ရပါမယ်။ SSE နဲ့ token တွေ ထွက်သလို ပြလို့ စိတ်ခံစားမှု ပိုကောင်းပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Endpoint function ကို `async def` နဲ့ ရေးပြီး CPU-bound အလုပ်တွေကို thread/process ပေးတဲ့ နည်းနဲ့ ဖြေရှင်းပါ။ Streaming အတွက် `StreamingResponse` သို့မဟုတ် FastAPI ရဲ့ server အလိုက် support ပေးတဲ့ `text/event-stream` content type နဲ့ async generator ကို return ပါ။

### ဥပမာ

```python
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def token_stream(prompt: str):
    # Async generator that yields chunks as they become ready
    for token in ["Hello", " world", " from", " AI"]:
        await asyncio.sleep(0.1)  # simulate generation latency
        yield f"data: {token}\n\n"
    yield "data: [DONE]\n\n"

@app.post("/chat/stream")
async def chat_stream(prompt: str):
    # SSE response: server pushes tokens to the client incrementally
    return StreamingResponse(token_stream(prompt),
                             media_type="text/event-stream")

# Expected output (client receives over time):
# data: Hello
# data:  world
# data:  from
# data:  AI
# data: [DONE]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Chatbot, code assistant တို့လို service တွေမှာ SSE streaming က user experience ရဲ့ အချက်အချာ ဖြစ်နေပါပြီ။ Async endpoint တွေကလည်း container ထဲမှာ CPU ကန့်သတ်ချက် ရှိတဲ့ အခြေအနေမှာ request throughput ကို အထောက်အကူ ပြုပါတယ်။

## 4. Request Validation, Error Contract နှင့် Health Endpoints

### ဘာကို ဆိုလိုတာလဲ
Request validation က client က ပို့လာတဲ့ data က form rule တွေနဲ့ ကိုက်ညီမှု ရှိ၊ မရှိ စစ်ဆေးခြင်းပါ။ Error contract က error ဖြစ်တဲ့အခါ server က ပြန်ပေးမယ့် response format ကို သတ်မှတ် response တစ်ခုတည်း အမြဲတမ်း ပြန်ပေးတဲ့ သဘောသဘာဝပါ။ Health endpoint က service အလုပ်လုပ်နေမှု ရှိ၊ မရှိကို ဖော်ပြတဲ့ endpoint ပါ။

### ဘာကြောင့် လဲ
AI backend တွေမှာ မှားယွင်းတဲ့ input (ဥပမာ empty text, လွန်ကဲရှည် text) က model crash သို့မဟုတ် မျှော်မှန်းထားတာထက် ပိုကုန်ကျတတ်ပါတယ်။ Error contract ရှိရင် client developer တွေက error ကို စနစ်တကျ ဖန်တီးနိုင်ပြီး health endpoint ရှိရင် container orchestration က service ပျက်တာကို အလိုအလျောက် သိရပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Validation အတွက် Pydantic model တွေကို request body အဖြစ် အသုံးပြုပြီး FastAPI က အလိုအလျောက် စစ်ပေးပါတယ်။ Error အတွက် exception handler တွေကို define လုပ်ပြီး format တစ်ခုတည်းနဲ့ ပြန်ပါ။ Health ကို `/health` path မှာ ရိုးရိုးရှင်းရှင်း ရေးပါ။

### ဥပမာ

```python
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from pydantic import BaseModel, Field

app = FastAPI()

class InferRequest(BaseModel):
    # Field constraints trigger automatic validation
    text: str = Field(..., min_length=1, max_length=1000)

@app.exception_handler(RequestValidationError)
async def validation_handler(request: Request, exc: RequestValidationError):
    # Fixed error contract: every error returns this JSON shape
    return {"error": {"type": "validation_error", "details": exc.errors()}}

@app.post("/infer")
def infer(body: InferRequest):
    return {"text_length": len(body.text)}

@app.get("/health")
def health():
    return {"status": "ok"}

# Expected output:
# POST /infer  {"text": ""}        -> {"error": {"type": "validation_error", ...}}
# POST /infer  {"text": "hello"}   -> {"text_length": 5}
# GET  /health                    -> {"status": "ok"}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Kubernetes တို့လို system တွေက `/health` (liveness) နဲ့ readiness endpoint တွေကို အခြေခံပြီး container ကို restart ချင်၊ လွဲပြောင်းချင် ဆုံးဖြတ်ပါတယ်။ Error contract က frontend, mobile app တွေရဲ့ error handling ကို ယုံကြည်စိတ်ချရစေပြီး validation က service ရဲ့ လုံခြုံမှုနဲ့ ကုန်ကျစရိတ် နှစ်ခုလုံးကို ကာကွယ်ပေးပါတယ်။

## အနှစ်ချုပ်

- **Routers** — endpoint တွေကို feature အလိုက် ဖိုင်ခွဲ စုစည်းခြင်း၊ `include_router()` နဲ့ app ထဲ တွဲသွင်းခြင်း။
- **Dependency Injection** — model client စတဲ့ ရင်းနှီးမှု ကြီးမားတဲ့ object တွေကို request တိုင်း မျှဝေ သုံးစွဲစေခြင်း၊ `Depends()` နဲ့ သတ်မှတ်ခြင်း။
- **Async Endpoints** — request တွေကို တစ်ခုနဲ့တစ်ခု မပိတ်ဘဲ တပြိုင်တည်း လက်ခံဆောင်ရွက်စေခြင်း။
- **Streaming (SSE)** — LLM output တွေကို token အလိုက် တိုက်ရိုက် စိတ်ဝင်စားစရာ ကြည့်ရှုနိုင်စေခြင်း၊ `StreamingResponse` + `text/event-stream`။
- **Validation** — Pydantic `Field` constraints တွေနဲ့ input data ကို အလိုအလျောက် စစ်ဆေးခြင်း။
- **Error Contract** — error ဖြစ်ပါက ပုံစံတစ်ခုတည်း ဖြင့် အမြဲ တုံ့ပြန်စေခြင်း၊ exception handler နဲ့ သတ်မှတ်ခြင်း။
- **Health Endpoints** — container orchestration က service အခြေအနေကို စစ်နိုင်ရန် `/health` စတဲ့ endpoint ပေးခြင်း။
