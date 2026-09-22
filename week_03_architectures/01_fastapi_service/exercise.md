## လေ့ကျင့်ခန်း ၁ — Health Endpoint တည်ဆောက်ခြင်း

`main.py` ထဲတွင် FastAPI app တစ်ခု ဖန်တီးပြီး `/health` endpoint ထည့်ပါ။ response မှာ `{"status": "ok"}` ဆိုသော JSON ဖြစ်ရမည်။ `uvicorn main:app --reload` ဖြင့် run ကာ browser တွင် စမ်းကြည့်ပါ။

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health_check():
    # Simple liveness endpoint for load balancers and orchestrators
    return {"status": "ok"}
```

**Hints:** `pip install fastapi uvicorn` ဖြင့် ပထမဆုံး install လုပ်ပါ။ endpoint function ကို `def` သာ သုံးလျှင် လုံလောက်ပါသည်။
**Expected behavior:** `http://127.0.0.1:8000/health` ကို ဖွင့်လျှင် `{"status":"ok"}` ပြမည်။ `/docs` တွင် endpoint ပေါ်မည်။

## လေ့ကျင့်ခန်း ၂ — Pydantic ဖြင့် Request Validation

AI backend အတွက် prompt request model တစ်ခု ဖန်တီးပါ။ `text` field (အနည်းဆုံး စာလုံး ၁ ခု)၊ `max_tokens` field (1 မှ 512 အတွင်း၊ default 128) ပါရမည်။ validation မှားသည့်အခါ FastAPI ပြမည့် 422 error structure ကို လေ့လာပါ။

```python
from pydantic import BaseModel, Field

class PromptRequest(BaseModel):
    text: str = Field(min_length=1)
    max_tokens: int = Field(default=128, ge=1, le=512)
```

**Hints:** `Field` ကို `pydantic` မှ import လုပ်ပါ။ `ge` / `le` သည် greater-or-equal / less-or-equal ဖြစ်သည်။
**Expected behavior:** `max_tokens: 0` ပို့လျှင် 422 နှင့် `detail` ထဲတွင် error အကြောင်းပေါ်မည်။ မှန်သော input တွင် 200 return မည်။

## လေ့ကျင့်ခန်း ၃ — Router ခွဲခြားခြင်း

အပေါ်မှ endpoints များကို `routers/inference.py` ထဲသို့ `APIRouter` ဖြင့် ပြောင်းရွှေ့ပါ။ prefix ကို `/api/v1` ဟု သတ်မှတ်ပြီး `main.py` တွင် `include_router` ဖြင့် ချိတ်ပါ။

```python
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1")

@router.post("/generate")
def generate(req: dict):
    # Placeholder generation endpoint, organized in its own router file
    return {"input": req}
```

**Hints:** `main.py` ထဲတွင် `app.include_router(router)` ကို သုံးပါ။
**Expected behavior:** `/api/v1/generate` သို့ POST ပို့လျှင် အလုပ်လုပ်ပြီး `/generate` ဟောင်းမှ 404 ရမည်။

## လေ့ကျင့်ခန်း ၄ — Dependency Injection ဖြင့် Model Loader

`get_model()` ဆိုသော dependency function တစ်ခု ရေးပါ။ ၎င်းက fake model object (ဥပမာ dict) တစ်ခုကို ပြန်ပေးပြီး endpoint တစ်ခုစီတွင် parameter အဖြစ် လက်ခံပါ။ app တစ်ခုလုံးအတွက် model ကို တစ်ကြိမ်တည်းသာ load ဖြစ်စေရန် `lifespan` context နှင့် ပေါင်းစပ်ပါ။

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI, Depends

_FAKE_MODEL = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Load shared resources once at startup
    _FAKE_MODEL["weights"] = "loaded"
    yield
    _FAKE_MODEL.clear()

def get_model():
    # Dependency that exposes the shared model to endpoints
    return _FAKE_MODEL
```

**Hints:** `FastAPI(lifespan=lifespan)` ဟု ရေးပါ။ endpoint တွင် `model: dict = Depends(get_model)` သုံးပါ။
**Expected behavior:** endpoint ခေါ်တိုင်း model ကို ပြန် load မလုပ်ဘဲ startup တွင် တစ်ကြိမ်တည်းသာ load ဖြစ်မည်။

## လေ့ကျင့်ခန်း ၅ — Async Endpoint နှင့် Error Contract

`async def` endpoint တစ်ခု ရေးပြီး `await asyncio.sleep()` ဖြင့် inference latency ကို စုံပါ။ `text` မရှိပါက `HTTPException(status_code=400, detail="empty prompt")` ပြန်ပါ။ endpoint အားလုံး အသုံးပြုမည့် error format `{"error": {"code": ..., "message": ...}}` ကို သတ်မှတ်ပြီး exception handler ဖြင့် စနစ်တကျ ပြန်ပါ။

**Hints:** `@app.exception_handler(HTTPException)` ကို သုံး၍ custom response ပြန်နိုင်သည်။ async endpoint ထဲတွင် blocking call မထည့်ပါ။
**Expected behavior:** error ဖြစ်သည့်အခါ သတ်မှတ် contract အတိုင်း JSON ပြန်မည်၊ status code မှန်မည်။

## လေ့ကျင့်ခန်း ၆ — Streaming Response (SSE) ဖြင့် Token Stream

`StreamingResponse` သုံး၍ token များကို SSE format (`data: token\n\n`) ဖြင့် တစ်ခုပြီးတစ်ခု stream ပြုလုပ်ပါ။ generator ထဲတွင် `await asyncio.sleep(0.1)` ထည့်၍ စစ်မှန်သော async streaming ဖြစ်စေပါ။

```python
import asyncio
from fastapi.responses import StreamingResponse

async def token_stream(prompt: str):
    # Yield tokens one by one in Server-Sent Events format
    for word in prompt.split():
        await asyncio.sleep(0.1)
        yield f"data: {word}\n\n"
    yield "data: [DONE]\n\n"

@app.post("/stream")
async def stream(prompt: str):
    return StreamingResponse(
        token_stream(prompt),
        media_type="text/event-stream",
    )
```

**Hints:** `curl -N -X POST "http://127.0.0.1:8000/stream?prompt=hello%20world"` ဖြင့် စမ်းပါ။ media_type ကို `text/event-stream` ဟု တပ်ပါ။
**Expected behavior:** token များ တစ်ခုစီ တဖြည်းဖြည်း ရောက်လာမည်၊ နောက်ဆုံးတွင် `[DONE]` ရမည်၊ တစ်ပြိုင်တည်း အားလုံး မရောက်ပါ။
