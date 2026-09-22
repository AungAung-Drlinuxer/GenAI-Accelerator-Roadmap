# Week 3 — AI Architectures & Containerization

## ဒီပတ်မှာ ဘာသင်မလဲ

ဒီပတ်မှာ AI backend တစ်ခုကို production အဆင့်မှာ တည်ဆောက်ဖို့ လိုအပ်တဲ့ architecture အခြေခံများနဲ့ containerization နည်းပညာများကို လေ့လာပါမယ်။ FastAPI နဲ့ service design ၊ Celery + Redis နဲ့ background jobs ၊ PostgreSQL + pgvector နဲ့ vector search ၊ နောက်ဆုံးမှာ Docker နဲ့ တစ်ခုတည်းသော system အဖြစ် ချိတ်ဆက်ပါမယ်။

## Modules

- `01_fastapi_service/` — **FastAPI Service Design for AI Backends** — Routers, dependency injection, async endpoints, SSE streaming, request validation, error contract နဲ့ health endpoints တွေကို fastapi.tiangolo.com က official docs အတိုင်း လက်တွေ့ရေးပါမယ်။
- `02_celery_redis_jobs/` — **Background Jobs with Celery + Redis** — Task queues, worker pools, retries with backoff, idempotency keys, scheduling, dead-letter handling တွေကို Celery official docs အရ လေ့လာပြီး LLM အလုပ်ရှည်များကို request path ကနေ ခွဲထုတ်ပါမယ်။
- `03_postgres_pgvector_alembic/` — **PostgreSQL + pgvector + Alembic** — documents/chunks အတွက် schema design, embedding column types, HNSW vs IVFFlat index များ, distance operators, Alembic migrations နဲ့ index build cost များကို pgvector repo နဲ့ SQLAlchemy/Alembic docs အတိုင်း သင်ပါမယ်။
- `04_docker_mcp/` — **Containerization & MCP Integration** — Python service များအတွက် Dockerfile, multi-stage builds, Compose နဲ့ api/worker/db/redis ချိတ်ဆက်ခြင်း, secrets ကို env အနေနဲ့ ထိန်းခြင်း၊ နောက်ဆုံး MCP server တစ်ခုကို backend service ထဲ ထည့်သွင်းပြပါမယ်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ်ပြီးဆုံးရင် သင်ဟာ —

- AI inference လုပ်ငန်းများကို လက်ခံဆောင်ရွက်ပေးနိုင်တဲ့ FastAPI service တစ်ခုကို validation နဲ့ error contract ပါဝါတဲ့ နည်းနဲ့ design ဆွဲနိုင်မယ်။
- အချိန်ကြာတဲ့ LLM jobs များကို Celery worker များဆီ တွန်းပို့ပြီး retry/idempotency နဲ့ လုံခြုံစွာ ဆောင်ရွက်နိုင်မယ်။
- embedding များသိမ်းဖို့ PostgreSQL + pgvector schema တစ်ခုကို Alembic migration များနဲ့ version control လုပ်နိုင်မယ်။
- api, worker, database, redis တွေပါဝင်တဲ့ Docker Compose stack တစ်ခုကို တစ်နေရာတည်းကနေ လည်ပတ်စေနိုင်မယ်။

## လေ့လာရန် အစီအစဉ်

1. `01_fastapi_service/` — ပထမနေ့ (၁) ရက် — API layer အခြေခံ
2. `02_celery_redis_jobs/` — ဒုတိယနေ့ (၂) ရက် — async job processing
3. `03_postgres_pgvector_alembic/` — တတိယနေ့ (၂) ရက် — data layer နဲ့ vector search
4. `04_docker_mcp/` — စတုတ္ထနေ့ (၂) ရက်) — အားလုံးကို container ထဲ ပေါင်းစပ်ခြင်း

## Checkpoint

1. FastAPI မှာ dependency injection ကို ဘာကြောင့် သုံးသင့်ပြီး database connection တစ်ခုကို ဘယ်လို share လုပ်သလဲ။
2. Celery task တစ်ခ�်ကို idempotent ဖြစ်စေဖို့ idempotency key ကို ဘယ်လို အသုံးချသင့်လဲ။
3. pgvector မှာ HNSW နဲ့ IVFFlat index နှစ်ခုကွာခြားချက်က ဘာလဲ၊ ဘယ်အချိန်မျိုးမှာ ဘယ်ဟာ ရွေးသင့်လဲ။
4. multi-stage Docker build က image size ကို ဘယ်လို လျှော့ချပေးလဲ၊ compose file ထဲမှာ api, worker, db, redis တွေကို ဘယ်လို ချိတ်ဆက်သလဲ။
