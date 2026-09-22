## လေ့ကျင့်ခန်း ၁ — Routers ဖြင့် API ခွဲခြားခြင်း

```python
# main.py - organize endpoints into multiple routers with prefixes and tags
from fastapi import FastAPI, APIRouter

# Create a router dedicated to AI inference endpoints
inference_router = APIRouter(prefix="/inference", tags=["inference"])

@inference_router.post("/text")
async def run_text_inference(prompt: str):
    # A placeholder inference call; a real service would load a model here
    return {"task": "text", "prompt": prompt}

@inference_router.post("/embedding")
async def run_embedding(text: str):
    return {"task": "embedding", "length": len(text)}

app = FastAPI(title="AI Backend")

# Register the router; routes become /inference/text and /inference/embedding
app.include_router(inference_router)
```

**အဓိကအယူအဆ** — APIRouter သည် endpoint များကို prefix နှင့် tag အတန်းအတန်းဖြင့် စနစ်တကျ ခွဲခြားပေးပြီး FastAPI app ကြီးထဲသို့ `include_router` ဖြင့် တစ်နေရာတည်းမှ စုသွင်းနိုင်စေသည်။

## လေ့ကျင့်ခန်း ၂ — Dependency Injection အသုံးပြုခြင်း

```python
# deps.py - shared resources provided through Depends
from fastapi import Depends, Header, HTTPException

API_TOKEN = "demo-token-123"

def get_model_service():
    # In production this would hold a loaded model singleton
    class ModelService:
        def predict(self, prompt: str) -> str:
            return "echo: " + prompt
    return ModelService()

async def verify_token(x_api_token: str = Header(default="")):
    # Validate an API token sent in the request header
    if x_api_token != API_TOKEN:
        raise HTTPException(status_code=401, detail="Invalid API token")
    return x_api_token
```

```python
# main2.py - use the dependencies in an endpoint
from fastapi import FastAPI, Depends
from deps import get_model_service, verify_token

app = FastAPI()

@app.post("/predict")
async def predict(prompt: str, token: str = Depends(verify_token),
                  model=Depends(get_model_service)):
    # verify_token and get_model_service run before the endpoint body
    return {"token_ok": True, "result": model.predict(prompt)}
```

**အဓိကအယူအဆ** — Depends ဖြင့် model loading နှင့် authentication ကဲ့သို့သော ပုံစံတူလုပ်ငန်းများကို endpoint အပြင်မှ ထုတ်ပေးခြင်းဖြင့် ကုဒ်ထပ်မံရေးမှု လျော့စေပြီး စမ်းသပ်ရလွယ်သည်။

## လေ့ကျင့်ခန်း ၃ — Async Endpoints ကောင်းမွန်စွာရေးခြင်း

```python
# main3.py - async endpoints for I/O-bound model calls
import asyncio
from fastapi import FastAPI

app = FastAPI()

async def fake_model_call(prompt: str) -> str:
    # Simulate an awaitable inference call, e.g. an async HTTP client
    await asyncio.sleep(0.1)
    return "result for: " + prompt

@app.post("/generate")
async def generate(prompt: str):
    # async def lets the event loop serve other requests while awaiting
    result = await fake_model_call(prompt)
    return {"output": result}
```

**အဓိကအယူအဆ** — I/O လုပ်ငန်းများအတွက် `async def` နှင့် `await` ကို အသုံးပြုခြင်းသည် တစ်ခုတည်းသော request ကို စောင့်စဉ် event loop သည် အခြား request များကိုပါ ဆက်လက်လုပ်ဆောင်စေနိုင်သည်။

## လေ့ကျင့်ခန်း ၄ — SSE Streaming Response

```python
# main4.py - stream generated tokens to the client via Server-Sent Events
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def token_stream(prompt: str):
    # Yield tokens one by one; client receives them as SSE messages
    words = prompt.split()
    for word in words:
        await asyncio.sleep(0.2)  # simulate per-token latency
        yield f"data: {word}\n\n"
    yield "data: [DONE]\n\n"

@app.post("/stream")
async def stream(prompt: str):
    # media_type text/event-stream enables Server-Sent Events
    return StreamingResponse(token_stream(prompt),
                             media_type="text/event-stream")
```

**အဓိကအယူအဆ** — StreamingResponse ကို `text/event-stream` media type ဖြင့် ပေးပို့ခြင်းဖြင့် model ထုတ်ပေးသည့် token တစ်ခုချင်းစီကို စောင့်ဆိုင်းမှုမရှိဘဲ client ထံ တိုက်ရိုက် stream လုပ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Request Validation

```python
# main5.py - enforce input rules with Pydantic models
from pydantic import BaseModel, Field
from fastapi import FastAPI

class GenerationRequest(BaseModel):
    prompt: str = Field(min_length=1, max_length=2048)
    max_tokens: int = Field(default=64, ge=1, le=1024)
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)

app = FastAPI()

@app.post("/generate")
async def generate(request: GenerationRequest):
    # FastAPI validates automatically and returns 422 on bad input
    return {
        "prompt": request.prompt,
        "settings": {
            "max_tokens": request.max_tokens,
            "temperature": request.temperature,
        },
    }
```

**အဓိကအယူအဆ** — Pydantic BaseModel နှင့် Field ကန့်သတ်ချက်များသည် မှားယွင်းသော input များကို 422 error အဖြစ် အလိုအလျောက် ပယ်ဖျက်ပေးပြီး endpoint အတွင်းရှိ ကုဒ်ကို ရိုးရိုးသန့်သန့် ဖြစ်စေသည်။

## လေ့ကျင့်ခန်း ၆ — Error Contract တည်ဆောက်ခြင်း

```python
# main6.py - a uniform error response shape for all failures
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(Exception)
async def unhandled_error_handler(request: Request, exc: Exception):
    # Wrap any unexpected error in a consistent JSON structure
    return JSONResponse(
        status_code=500,
        content={"error": {"code": "internal_error",
                           "message": "Unexpected server error"}},
    )

@app.get("/divide")
async def divide(a: float, b: float):
    if b == 0:
        # Return the same error shape for domain-level failures
        return JSONResponse(
            status_code=400,
            content={"error": {"code": "bad_request",
                               "message": "b must not be zero"}},
        )
    return {"result": a / b}
```

**အဓိကအယူအဆ** — error တစ်ခုစီအတွက် `error.code` နှင့် `error.message` ပါဝင်သော ပုံစံတစ်မျိုးတည်းသော JSON contract ကို သတ်မှတ်ခြင်းဖြင့် client များက အခြေအနေအားလုံးကို တူညီသောနည်းဖြင့် ကိုင်တွယ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၇ — Health Endpoints

```python
# main7.py - liveness and readiness probes for container orchestration
from fastapi import FastAPI
import time

app = FastAPI()
START_TIME = time.time()
MODEL_READY = False  # set True once the model finishes loading

@app.get("/health/live")
async def liveness():
    # The process itself is alive; returns 200 even if model is loading
    return {"status": "alive"}

@app.get("/health/ready")
async def readiness():
    # Tells the orchestrator whether the service can accept traffic
    if not MODEL_READY:
        from fastapi.responses import JSONResponse
        return JSONResponse(
            status_code=503,
            content={"status": "not_ready",
                     "uptime_seconds": round(time.time() - START_TIME, 1)},
        )
    return {"status": "ready"}

@app.post("/admin/load-complete")
async def mark_ready():
    # Simulate the model loading process finishing
    global MODEL_READY
    MODEL_READY = True
    return {"message": "model marked as ready"}
```

**အဓိကအယူအဆ** — liveness probe သည် process အသက်ရှိမှုကိုသာ စစ်ပြီး readiness probe သည် model load ပြီးစီးပြီး traffic လက်ခံနိုင်မှုကို စစ်သဖြင့် container orchestrator များအတွက် ခွဲခြားသတ်မှတ်ပေးရမည်။
