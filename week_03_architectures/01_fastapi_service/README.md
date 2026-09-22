# FastAPI Service Design for AI Backends

AI backend တွေအတွက် FastAPI နဲ့ production-ready service design — routers, dependency injection, async endpoints, SSE streaming, validation, error contract နဲ့ health endpoints တွေကို အခြေခံကစပြီး လက်တွေ့ကျနေအောင် သင်ကြားပေးမယ့် module ပါ။

## ဒီ module မှာ ဘာသင်မလဲ

- FastAPI application တစ်ခုကို routers နဲ့ ဖွဲ့စည်းပြီး တည်ငင်ရှုံ့မရှိအောင် ခွဲခြားနည်း
- Dependency injection ကို အသုံးပြုပြီး model loading, config, auth စတာတွေကို သန့်သန့်ရှင်ရှင် manage လုပ်နည်း
- Async endpoints ရေးနည်းနဲ့ blocking work တွေကို မှန်ကန်စွာ handle လုပ်နည်း
- Server-Sent Events (SSE) နဲ့ AI output တွေကို streaming လုပ်ပြနည်း
- Request validation နဲ့ တူညီသော error contract တစ်ခု ထားရှိနည်း
- Container orchestration တွေက health check လုပ်နိုင်ဖို့ `/health` endpoints တွေ တပ်ဆင်နည်း

## သင်ခန်းစာများ

1. **Project Structure နဲ့ Routers** — `APIRouter` နဲ့ resource-based routing ကို ခွဲခြားဖွဲ့စည်းနည်း။
2. **Dependency Injection** — `Depends` ကိုသုံးပြီး settings, DB sessions, model handles တွေကို inject လုပ်နည်း။
3. **Async Endpoints** — `async def`, thread offloading, နဲ့ concurrency အခြေခံများ။
4. **Streaming with SSE** — `StreamingResponse` နဲ့ `text/event-stream` format အသုံးပြုနည်း။
5. **Request Validation** — Pydantic models, nested schemas, နဲ့ custom validators။
6. **Error Contract** — `HTTPException`, exception handlers, နဲ့ တစ်သတ်မတတ် error response format ဒီဇိုင်း။
7. **Health Endpoints** — liveness/readiness probe အတွက် `/health` နဲ့ `/health/ready` တည်ဆောက်နည်း။

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံ (function, class, type hints) နဲ့ HTTP request/response သဘောတရား နားလည်ထားရမယ်
- Week 2 က Pydantic data validation အပိုင်းကို ဖြတ်သွားထားရမယ်
- `pip install "fastapi[standard]"` နဲ့ local environment တည်ဆောက်နိုင်ရမယ်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Inference API service တစ်ခုကို production မှာ deploy မယ်ဆိုရင် ဒီ module က design pattern တွေကို တိုက်ရိုက် အသုံးချနိုင်ပါတယ်။
- Chatbot သို့မဟုတ် generation service တွေရဲ့ output ကို token-by-token ပြပေးချင်တဲ့အခါ SSE streaming knowledge က မဖြစ်မနေ လိုအပ်ပါတယ်။
- Kubernetes သို့မဟုတ် Docker Swarm ပေါ်မှာ container health probes တွေနဲ့ ဆက်သွယ်ရင် health endpoint design က အရေးကြီးပါတယ်။
- Team တစ်ခုလုံး same API တွေ ရေးနေတဲ့အခါ error contract တစ်ခုတည်းက integration ကို လွယ်ကူစေပါတယ်။

## ကိုးကား

- FastAPI Bigger Applications (Routing): https://fastapi.tiangolo.com/tutorial/bigger-applications/
- FastAPI Dependencies: https://fastapi.tiangolo.com/tutorial/dependencies/
- FastAPI Async: https://fastapi.tiangolo.com/async/
- FastAPI Streaming Responses: https://fastapi.tiangolo.com/advanced/custom-response/
- FastAPI Request Body / Data Validation: https://fastapi.tiangolo.com/tutorial/body/
- FastAPI Handling Errors: https://fastapi.tiangolo.com/tutorial/handler-errors_errors/
- FastAPI Path Operations Advanced (Additional Responses): https://fastapi.tiangolo.com/advanced/additional-responses/
