# 6-Week Study Roadmap

ဒီ roadmap သည် weekly phased plan ဖြစ်ပြီး တစ်ပတ်လျှင် module ၄ ခုစီ ပါဝင်ပါတယ်။ နေ့စဉ် ၁.၅–၂ နာရီ လေ့လာမည်ဟု ယူဆထားပါတယ်။

## Week 1 — Foundations of AI Engineering

**Week goal:** AI engineering အတွက် တည်ဆောက်စရာ environment, toolchain နှင့် LLM API အခြေခံများကို သျှင်းလင်းစွာ နားလည်စေရန်။

**Modules:**
- `week_01_foundations/01_python_uv_workflow` — uv project setup, lockfiles, deterministic environments
- `week_01_foundations/02_prompt_engineering` — prompts, few-shot, format constraints, versioning
- `week_01_foundations/03_llm_apis_providers` — Chat Completions, streaming, retries, Ollama
- `week_01_foundations/04_quality_toolchain` — ruff, pytest, secrets hygiene

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** `uv` နဲ့ reproducible project တစ်ခု တည်ဆောက်နိုင်ခြင်း၊ LLM API ကို retry/timeout နှင့် ခေါ်နိုင်ခြင်း၊ prompt များကို version ခွဲစီမံနိုင်ခြင်း၊ pytest နဲ့ test ရေးနိုင်ခြင်း။

**Checkpoint self-test:**
1. `uv.lock` file က ဘာကို အာမခံပေးလဲ? ဘာကြောင့် LLM application တွင် အရေးကြီးလဲ?
2. temperature နှင့် top-p က output အပေါ် ဘယ်လိုသက်ရောက်လဲ?
3. LLM API call တစ်ခုမှာ retry နှင့် exponential backoff ဘာကြောင့် လိုအပ်လဲ?
4. API key ကို `.env` ထဲမှာ သိမ်းပြီး git မထဲသွင်းရန် ဘယ်လို စီမံမလဲ?

## Week 2 — AI System Design Principles

**Week goal:** LLM ရဲ့ messy output ကို typed contracts နဲ့ control လုပ်ပြီး system design အခြေခံများ ယူနိုင်ရန်။

**Modules:**
- `week_02_system_design/01_pydantic_contracts` — Pydantic v2 BaseModel, Field, validators, JSON schema
- `week_02_system_design/02_structured_output` — JSON mode, function calling, repair loops, fallbacks
- `week_02_system_design/03_context_engineering` — token budgets, summarisation, caching
- `week_02_system_design/04_modular_architecture` — workflows, typed nodes, DI

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** Pydantic schema တစ်ခုကို LLM prompt အဖြစ် အသုံးချနိုင်ခြင်း၊ validation error ကို feedback အဖြစ်ပြီး repair loop ဆွဲနိုင်ခြင်း၊ context window ကို token budget အလိုက် စီမံနိုင်ခြင်း၊ orchestration နှင့် steps ကို ခွဲခြားနိုင်ခြင်း။

**Checkpoint self-test:**
1. LLM output ကို Pydantic နဲ့ validate လို့ မအောင်မြင်ရင် နောက်ဆုံး fallback ကို ဘယ်လို ရွေးမလဲ?
2. Repair loop ကို bounded (ကန့်သတ်ထားသော) retries နဲ့ ရေးရတာ ဘာကြောင့်လဲ?
3. Context window ထဲ ဘာတွေ ထည့်သင့်ပြီး ဘာတွေ မထည့်သင့်လဲ?
4. Dependency injection က testing ကို ဘယ်လို အလွယ်တကူ ဖြစ်စေလဲ?

## Week 3 — AI Architectures & Containerization

**Week goal:** AI backend တစ်ခုအတွက် service, queue, database, container အခြေခံအုတ်မြစ် တည်ဆောက်တတ်ရန်။

**Modules:**
- `week_03_architectures/01_fastapi_service` — routers, DI, async, SSE streaming
- `week_03_architectures/02_celery_redis_jobs` — task queues, retries, idempotency, dead-letter
- `week_03_architectures/03_postgres_pgvector_alembic` — vector columns, HNSW/IVFFlat, migrations
- `week_03_architectures/04_docker_mcp` — Dockerfiles, Compose, MCP server

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** FastAPI service တစ်ခုကို streaming response နှင့် ရေးနိုင်ခြင်း၊ ကြာမြင့်သော LLM job ကို Celery worker ထဲ ရွှေ့နိုင်ခြင်း၊ pgvector schema နှင့် Alembic migration ရေးနိုင်ခြင်း၊ api/worker/db/redis ပါဝင်သော docker-compose.yml ဆွဲနိုင်ခြင်း။

**Checkpoint self-test:**
1. ကြာမြင့်သော LLM generation ကို request path ထဲက ဘာကြောင့် ဖယ်ရှင်းသလဲ? ဘယ်နေရာထားသင့်လဲ?
2. HNSW နှင့် IVFFlat index တို့ရဲ့ ကွာခြားချက်က ဘာလဲ?
3. Idempotency key က duplicate job execution ကို ဘယ်လို တားဆီးလဲ?
4. Multi-stage Docker build က image size ကို ဘယ်လို လျှော့ချပေးလဲ?

## Week 4 — Retrieval Augmented Generation

**Week goal:** Documents ကိ ingest → chunk → embed → retrieve → rerank → answer pipeline တစ်ခုကို တကယ်လက်တွေ့ အလုပ်လုပ်တတ်ရန်။

**Modules:**
- `week_04_rag/01_ingestion_chunking` — parsing, chunk strategies, overlap, metadata
- `week_04_rag/02_embeddings_vector_store` — models, distance metrics, normalization, versioning
- `week_04_rag/03_hybrid_search_rrf` — keyword + vector recall, RRF, reranking, filters
- `week_04_rag/04_rag_evaluation` — golden sets, recall@k/MRR/nDCG, faithfulness, CI gates

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** idempotent ingestion pipeline ရေးနိုင်ခြင်း၊ pgvector ပေါ်တွင် hybrid search query ဆွဲနိုင်ခြင်း၊ RRF နှင့် reranking ထည့်နိုင်ခြင်း၊ golden dataset တစ်ခုနဲ့ RAG quality တိုင်းနိုင်ခြင်း။

**Checkpoint self-test:**
1. Chunk size ကြီးလာရင် recall နှင့် cost အပေါ် ဘယ်လိုသက်ရောက်လဲ?
2. Cosine distance နှင့် inner product ကြားမှာ ဘယ်အချိန် ဘယ်ဟာ သင့်တော်လဲ?
3. RRF က keyword results နှင့် vector results ကို ဘယ်လို ပေါင်းလဲ?
4. Faithfulness check က RAG hallucination ကို ဘယ်လို ဖမ်းဖို့ ရနိုင်လဲ?

## Week 5 — LLM Monitoring & Evaluations

**Week goal:** ထုတ်လုပ်ပြီး system တစ်ခုကို တိုင်းတာနိုင်ရန်၊ ကာကွယ်နိုင်ရန်၊ စျေးကွက်ရှင်းနိုင်ရန်။

**Modules:**
- `week_05_monitoring_evals/01_tracing_langfuse` — spans, generations, sessions, trace debugging
- `week_05_monitoring_evals/02_eval_pipelines` — datasets, LLM-judge scoring, CI regression gates
- `week_05_monitoring_evals/03_guardrails_safety` — PII redaction, injection defence, refusal patterns
- `week_05_monitoring_evals/04_cost_latency_slo` — cost accounting, caching, model routing

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** Langfuse နဲ့ trace ဖမ်းနိုင်ခြင်း၊ CI ထဲမှာ eval ပြေးပြီး threshold မှာ deploy တားနိုင်ခြင်း၊ prompt injection ကာကွယ်မှု ရေးနိုင်ခြင်း၊ feature အလိုက် cost/latency တွက်နိုင်ခြင်း။

**Checkpoint self-test:**
1. Trace တစ်ခုကို ကြည့်ပြီး ဆိုးသော answer တစ်ခုရဲ့ အကြောင်းရင်းကို ဘယ်လို ရှာမလဲ?
2. LLM-judge scoring ရဲ့ အားနည်းချက်က ဘာလဲ? Deterministic check နဲ့ ဘယ်လို ဖြည့်ဆည်းမလဲ?
3. Retrieved documents ထဲမှာ injection instructions ပါလာရင် ဘယ်လို ကာကွယ်မလဲ?
4. ဘယ်လိုအချိန်မျိုးမှာ small model ကို route လုပ်သင့်လဲ?

## Week 6 — Deploying AI Applications

**Week goal:** အထိထိပ်ရှုံ့မခံနိုင်သော production deploy pipeline တစ်ခု အပြီးအစီး တည်ဆောက်တတ်ရန်။

**Modules:**
- `week_06_deployment/01_container_deploy` — image shipping, Caddy + automatic HTTPS, zero-downtime
- `week_06_deployment/02_cicd_pipelines` — build/test/scan, digest deploys, smoke tests, rollback
- `week_06_deployment/03_observability_alerts` — structured logging, Sentry, SLOs, on-call playbook
- `week_06_deployment/04_security_secrets` — vault, least-privilege roles, scanning, key rotation

**ဒီပတ်ကုန်ရင် လုပ်နိုင်ရမည့်အရာ:** TLS ပါတဲ့ reverse proxy နောက်ကွယ်မှာ service deploy လုပ်နိုင်ခြင်း၊ CI/CD pipeline တစ်ခုကိ immutable digest နဲ့ deploy လုပ်နိုင်ခြင်း၊ p95 latency/error rate SLO တပ်ဆင်နိုင်ခြင်း၊ secrets ကို vault မှာ စီမံနိုင်ခြင်း။

**Checkpoint self-test:**
1. Deploy ကို digest အလိုက် လုပ်ခြင်းက tag အလိုက် လုပ်ခြင်းထက် ဘာကြောင့် ပိုစိတ်ချရလဲ?
2. Smoke test တစ်ခုက ဘယ်လိုအရာတွေကို စစ်သင့်လဲ?
3. p95 latency SLO တစ်ခု ချိုးပျက်ရင် alert တစ်ခုက ဘယ်အချက်အလက်တွေ ပါသင့်လဲ?
4. DB role တစ်ခုကို least-privilege ဖြစ်စေရန် ဘယ်လို သတ်မှတ်မလဲ?

## နေ့စဉ် လေ့လာပုံ အကြံပြုချက်

နေ့စဉ် ၁.၅–၂ နာရီ အတွက် အကြံပြု routine:

- **ရက်သတ္တ ၁–၂:** module ရဲ့ `explanation.md` ဖတ်ပါ — code ဥပမာများကို ကိုယ်ပိုင် environment မှာ run ကြည့်ပါ
- **ရက်သတ္တ ၃–၄:** `exercise.md` ကို ဖြေရှင်းပါ — solution ကို မကြည့်ခင် ကြိုးစားပါ၊ errors ကို ကိုယ်တိုင် debug လုပ်ပါ
- **ရက်သတ္တ ၅:** `solution.md` နဲ့ တိုက်စစ်ပါ — ကွဲလွဲချက်တွေကို မှတ်တမ်းတင်ပါ
- **ရက်သတ္တ ၆:** mini-project ကို ဆက်လုပ်ပါ — အရင်ပတ်တွေရဲ့ code တွေနဲ့ ပေါင်းစပ်ပါ
- **ရက်သတ္တ ၇:** checkpoint self-test ဖြေပါ — ဖြေမရတဲ့ အချက်တွေကို ပြန်ဖတ်ပါ၊ နောက်ပတ်အတွက် အဆင့်သင့်

ပတ်တစ်ခုစီမှာ module ၂ ခုစာ နောက်ကျခဲ့ရင် နောက်ထပ် ၂ ခုကို နှုန်းဖြင့် ဖြတ်ဖို့ထက် ပြီးသွားတဲ့အပိုင်းကို နားလည်အောင် အချိန်ပိုပေးတာက ပိုအကျိုးရှိပါတယ်။

## Capstone အကြံပြုချက်

Track ပြီးဆုံးရင် အောက်ပါ **production RAG + agent service** ကို ကိုယ်ပိုင် repo တစ်ခုမှာ တည်ဆောက်ပါ —

- **Stack:** uv + FastAPI (SSE streaming) + Celery/Redis (ingestion jobs) + PostgreSQL/pgvector (hybrid search) + Docker Compose
- **AI layer:** Pydantic contracts နဲ့ structured output, bounded repair loop, context budget စီမံမှု
- **RAG:** idempotent ingestion, semantic chunking, hybrid search + RRF + reranking, citation ပါ answer
- **Quality:** golden dataset တစ်ခုနဲ့ eval suite၊ CI ထဲမှာ regression gate (recall@k threshold)
- **Observability:** Langfuse tracing (feature tag များနဲ့)၊ structured logging၊ Sentry
- **Guardrails:** PII redaction၊ prompt injection defence၊ cost/latency metrics တစ်ခုစီ
- **Deploy:** GitHub Actions CI/CD — build → test → scan → digest deploy၊ Caddy နောက်ကွယ်မှာ TLS၊ health check နှင့် rollback plan

ဒီ project က interview မှာပါ အသုံးဝင်မည့် portfolio ဖြစ်ပါတယ် — README ထဲမှာ architecture diagram, eval scores နှင့် design decisions တွေကို ရှင်းပြထားပါ။
