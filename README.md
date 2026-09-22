# GenAI Accelerator Roadmap — Production AI Engineering (self-study track)

ဒီ repository သည် **production-ready AI engineering** ကို ၆ ပတ်အတွင်း လေ့လာနိုင်ရန် ရေးဆွဲထားသော self-study curriculum ဖြစ်ပါတယ်။

> **မှတ်ချက် — Original Material:** ဒီ study track သည် အများက တွေ့မြင်နိုင်သော publicly published 6-week AI-engineering roadmap outline တစ်ခုကို အခြေခံပြီး၊ ကျွန်ုပ်တို့ကိုယ်တိုင် ရေးဆွဲထားသော **ဆရာမဲ့ သင်ခန်းစာများ** ဖြစ်ပါတယ်။ နည်းပညာအချက်အလက်များအားလုံးကို official open documentation (Python, FastAPI, Pydantic, PostgreSQL/pgvector, Docker, Celery, Langfuse, Sentry, Caddy စသည့် docs များ) မှ ကိုးကား၍ ကိုယ်ပိုင်အဖြစ် ပြန်ရေးဖော်ထားပါတယ်။ မည်သည့် commercial course ၏ lessons များကိုမှ မကူးယူထားပါ။

## ဒီ track မှာ ဘာသင်မလဲ

- **Python project workflow** — `uv`, `pyproject.toml`, lockfiles နဲ့ deterministic environments
- **Prompt engineering** — system/user prompts, few-shot, output format constraints, temperature/top-p
- **LLM APIs** — OpenAI-compatible Chat Completions, streaming, retries/backoff, token/cost accounting, Ollama on-prem
- **Quality toolchain** — `ruff`, `pytest`, secrets hygiene, pre-commit gates
- **Pydantic v2 contracts** — typed LLM I/O, schema-driven prompting, structured output repair loops
- **Context engineering** — token budgets, summarisation, truncation honesty, prompt caching
- **Modular architecture** — workflows, typed nodes, dependency injection, failure isolation
- **FastAPI AI backends** — routers, DI, async, SSE streaming, error contracts
- **Background jobs** — Celery + Redis, retries, idempotency, dead-letter handling
- **PostgreSQL + pgvector + Alembic** — vector schema design, HNSW/IVFFlat, migrations
- **Docker & MCP** — multi-stage builds, Compose wiring, MCP server exposure
- **RAG** — chunking, embeddings, hybrid search + RRF + reranking, RAG evaluation & grounding
- **Monitoring & evals** — Langfuse tracing, eval pipelines, regression gates, guardrails, PII/injection defence, cost/latency SLOs
- **Deployment** — container deploy + Caddy TLS, CI/CD with digests, observability & Sentry, security hardening

## ၆ ပတ် အစီအစဉ်

### Week 1 — Foundations of AI Engineering (`week_01_foundations`)
- `01_python_uv_workflow` — uv နဲ့ reproducible Python environment တည်ဆောက်ပုံ
- `02_prompt_engineering` — prompt တည်ဆောက်မှုအခြေခံများနှင့် versioning
- `03_llm_apis_providers` — hosted/on-prem LLM APIs ခေါ်ဆိုပုံ၊ retries, cost
- `04_quality_toolchain` — ruff, pytest, secrets hygiene တည်ဆောက်ပုံ

### Week 2 — AI System Design Principles (`week_02_system_design`)
- `01_pydantic_contracts` — Pydantic v2 နဲ့ LLM input/output contracts
- `02_structured_output` — structured output နှင့် bounded repair loops
- `03_context_engineering` — context window စီမံခြင်း၊ token budgets
- `04_modular_architecture` — workflows, nodes, dependency injection

### Week 3 — AI Architectures & Containerization (`week_03_architectures`)
- `01_fastapi_service` — FastAPI နဲ့ AI backend service design
- `02_celery_redis_jobs` — Celery + Redis background jobs
- `03_postgres_pgvector_alembic` — vector store schema နှင့် migrations
- `04_docker_mcp` — Docker containers နှင့် MCP integration

### Week 4 — Retrieval Augmented Generation (`week_04_rag`)
- `01_ingestion_chunking` — document ingestion နှင့် chunking strategies
- `02_embeddings_vector_store` — embeddings နှင့် vector storage
- `03_hybrid_search_rrf` — hybrid search, RRF, reranking
- `04_rag_evaluation` — RAG evaluation နှင့် grounding checks

### Week 5 — LLM Monitoring & Evaluations (`week_05_monitoring_evals`)
- `01_tracing_langfuse` — Langfuse tracing နဲ့ debugging
- `02_eval_pipelines` — eval datasets နှင့် CI regression gates
- `03_guardrails_safety` — guardrails, PII redaction, prompt injection defence
- `04_cost_latency_slo` — cost/latency accounting နှင့် model routing

### Week 6 — Deploying AI Applications (`week_06_deployment`)
- `01_container_deploy` — container deploy, Caddy reverse proxy + TLS
- `02_cicd_pipelines` — CI/CD pipelines, digest deploys, rollback
- `03_observability_alerts` — structured logging, Sentry, SLOs, alerts
- `04_security_secrets` — secrets management, hardening, key rotation

## Folder ဖွဲ့စည်းပုံ

track တစ်ခုလုံးကို အောက်ပါအတိုင်း ဖွဲ့စည်းထားပါတယ်—

```
week_XX_name/
  01_module_name/
    README.md          # module အကြောင်း အနှစ်ချုပ်
    explanation.md     # သီအိုရီ + code ဥပမာများ (Burmese prose, English code)
    exercise.md        # လက်တွေ့စာတမ်း (ယူဉီးမြတ်)
    solution.md        # ဖြေရှင်းချက် ရည်ညွှန်း (ကြည့်ခဲ့ပြီးမှ ဖွင့်ပါ)
```

- `explanation.md` — concept ကို စတင်ဖတ်ရမည့် အဓိက document
- `exercise.md` — ကိုယ်တိုင် လက်တွေ့လုပ်ရမည့် tasks
- `solution.md` — မိမိဖြေရှင်းပြီးနောက်မှသာ တိုက်စစ်ရန်

## လေ့လာပုံ နည်းလမ်း

1. Module ၏ `explanation.md` ကို ဖတ်ပါ — code ဥပမာများကို ကိုယ်တိုင် run ကြည့်ပါ
2. `exercise.md` ထဲက tasks များကို ကိုယ်တိုင် ဖြေရှင်းပါ — solution မကြည့်ခင် အနည်းဆုံး ၃၀ မိနစ် ကြိုးစားပါ
3. ရလဒ်ကို `solution.md` နဲ့ တိုက်စစ်ပြီး ကွဲလွဲချက်များ သိမြင်ပါ
4. ပိုင်းလိုက် ရှင်းပြထားသော **mini-project** ကို ကိုယ်ပိုင် repo တွင် တည်ဆောက်ပါ
5. တစ်ချိန်လုံး `ROADMAP.md` ရှိ checkpoint မေးခွန်းများနဲ့ ကိုယ့်ကိုယ်ကို စစ်ပါ

## လိုအပ်ချက်များ

- **Python fundamentals** — functions, classes, type hints, virtual environments, pip
- ဒီ repository ရှိ **Python for AI course** ကို တက်ပြီးသူဖြစ်ရမည် (prerequisite)
- အသုံးပြုမည့် tools: Python 3.10+, `uv`, Docker, Git — အားလုံးကို ပထမပတ်မှာ တင်ပြပေးပါမယ်
- LLM API key တစ်ခု (သို့မဟုတ်) Ollama local model — Week 1 module 3 တွင် ရှင်းပြပါမယ်
