## လေ့ကျင့်ခန်း ၁ — Python service အတွက် အခြေခံ Dockerfile ရေးပါ

Python 3.12 slim image ကို အခြေခံပြီး `main.py` ဖိုင်တစ်ခုတည်း run တဲ့ container ဆောက်ပါ။ Dockerfile မှာ `WORKDIR`၊ `COPY`၊ `RUN pip install` နဲ့ `CMD` ဆိုတဲ့ instruction တွေပါဝင်ရမယ်။ dependency များကို `requirements.txt` ကနေ install အောင် စီစဉ်ပါ။

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

**Hints:** docs.docker.com ရဲ့ Dockerfile reference မှာ instruction တိုင်းရဲ့ သဘောတရားကို ဖတ်နိုင်ပါတယ်။ `--no-cache-dir` က image အရွယ်အစားလျှော့ပေးပါတယ်။
**Expected behavior:** `docker build -t my-api .` အား run ပြီးနောက် `docker run my-api` နဲ့ container က Python script ကို အောင်မြင်စွာ execute ပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Multi-stage build နဲ့ image အရွယ်အစား လျှော့ပါ

အထက်ပါ Dockerfile ကို multi-stage build အဖြစ်ပြောင်းပါ။ ပထမ stage မှာ dependency တွေကို wheel အဖြစ် build ပြီး ဒုတိယ stage မှာ runtime အတွက်လိုအပ်တဲ့ ဖိုင်တွေကိုပဲ copy ယူပါ။ ရလဒ် image ထဲမှာ build tool တွေ မကျန်ရှိစေရန် သတိပြုပါ။

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/*
COPY . .
CMD ["python", "main.py"]
```

**Hints:** `COPY --from=<stage>` ဆိုတဲ့ pattern က stage တစ်ခုကနေ အခြား stage ကို ဖိုင်ယူရန် သုံးပါတယ်။ ပုံမှန်အားဖြင့် final stage မှာ compiler တွေ မလိုအပ်ပါ။
**Expected behavior:** `docker images` ဖြင့်ကြည့်လျှင် multi-stage version က single-stage version ထက် သိသိသာသာ သေးငယ်တဲ့ image ဖြစ်နေပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Compose ဖြင့် api/worker/db/redis တွေကို ချိတ်ပါ

`docker-compose.yml` တစ်ခုရေးပြီး service လေးခု — `api`၊ `worker`၊ `db` (Postgres)၊ `redis` — ကို သတ်မှတ်ပါ။ `api` နဲ့ `worker` က တူညီတဲ့ Dockerfile ကို သုံးပြီး အမျိုးမျိုး command နဲ့ run ပါ။ healthcheck နဲ့ depends_on ထည့်ပြီး startup order ကို ထိန်းပါ။

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: examplepass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
  redis:
    image: redis:7
  api:
    build: .
    command: uvicorn main:app --host 0.0.0.0 --port 8000
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy
  worker:
    build: .
    command: python worker.py
    depends_on:
      - redis
      - db
```

**Hints:** Compose docs အတွက် `depends_on` ရဲ့ long syntax က service ready ဖြစ်မှ စောင့်ခိုင်းနိုင်ပါတယ်။ `postgres:16` image ရဲ့ default healthcheck tool က `pg_isready` ပါ။
**Expected behavior:** `docker compose up` အား run လျှင် container လေးခုလုံး တစ်ပြိုင်နက် စတင်ပြီး `db` ready ဖြစ်ပါမှ `api` က စ run ပါတယ်။

## လေ့ကျင့်ခန်း ၄ — Secret တွေကို environment variable အနေနဲ့ ထည့်ပါ

API key နဲ့ database password ကို image ထဲ တိုက်ရိုက်ရေးမှု့ထည့်စရာ မလိုပဲ environment variable အနေနဲ့ ဖြတ်ပေးပါ။ Compose file ထဲမှာ `.env` file ကနေ တန်ဖိုးတွေ ဆွဲယူပြီး container တွေဆီ ပို့ပါ။ `env_file` နဲ့ variable substitution နှစ်မျိုးလုံး လေ့ကျင့်ပါ။

```yaml
services:
  api:
    build: .
    env_file:
      - .env
    environment:
      DATABASE_URL: postgres://appuser:${DB_PASSWORD}@db:5432/appdb
```

**Hints:** `.env` file ကို `.dockerignore` နဲ့ `.gitignore` ထဲ ထည့်ပြီး repository ထဲ မရောက်စေပါနဲ့။ Compose variable substitution အတွက် syntax က `${VAR}` ပါ။
**Expected behavior:** `docker compose config` အား run လျှင် substitution ဖြစ်ပြီး တန်ဖိုးတွေ ပေါင်းစပ်ပါဝင်နေတာကို မြင်ရပြီး `.env` file ကို မထည့်ပဲ build လုပ်လျှင် secret တွေ image ထဲ အလိုအလျောက် ပါမသွားပါ။

## လေ့ကျင့်ခန်း ၅ — Backend service ထဲ MCP server တစ်ခု expose လုပ်ပါ

`modelcontextprotocol.io` docs အရ Python SDK နဲ့ MCP server တစ်ခုရေးပြီး FastAPI backend တစ်ခုထဲမှာ HTTP endpoint အနေနဲ့ expose လုပ်ပါ။ ရိုးရှင်းတဲ့ tool တစ်ခု (ဥပမာ — စာသားရှည် ပြန်ပေးတဲ့ echo tool) ကို define လုပ်ပါ။

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("demo-tools")

@mcp.tool()
def echo(text: str) -> str:
    # Return the input text unchanged
    return text

if __name__ == "__main__":
    mcp.run(transport="sse")
```

FastAPI app တစ်ခုက MCP server ရဲ့ endpoint ကို `/mcp` path မှာ mount လုပ်ပြီး Compose ရဲ့ `api` service အဖြစ် run အောင် ပြင်ပါ။

**Hints:** MCP Python SDK docs မှာ transport option များ (`stdio`၊ `sse`၊ `streamable-http`) အကြောင်း ရှင်းပြထားပါတယ်။ စမ်းရန် MCP inspector tool ကို သုံးနိုင်ပါတယ်။
**Expected behavior:** Container run နေစဉ် MCP client (သို့) inspector ကနေ `echo` tool ကို ခေါ်လျှင် ပေးလိုက်တဲ့ စာသားအတိုင်း တုံ့ပြန်မှု ရရှိပါတယ်။

## လေ့ကျင့်ခန်း ၆ — အားလုံးကို ပေါင်းစပ်ပြီး Production-ready stack ဆောက်ပါ

လေ့ကျင့်ခန်း ၃ ကနေ ၆ အထိ အားလုံးကို ပေါင်းပါ — multi-stage Dockerfile၊ Compose နဲ့ service လေးခု၊ secrets ကို env ဖြင့်၊ နဲ့ MCP endpoint ပါတဲ့ `api` service တွေကို config လုပ်ပါ။ Redis ကို `worker` ရဲ့ queue အဖြစ်သုံးပြီး `api` ရဲ့ healthcheck ထည့်ပါ။ restart policy နဲ့ resource limit တွေ ပေါင်းထည့်ပြီး တစ်ခုတည်းရှိတဲ့ `docker compose up -d` command နဲ့ stack အလုံးအစင် ထကြောင်း run နိုင်အောင် စစ်ဆေးပါ။

**Hints:** Compose docs ရဲ့ `deploy.resources` section က memory/CPU limit သတ်မှတ်ရန် အသုံးပြုပါတယ်။ Healthcheck ကို `curl -f http://localhost:8000/health` စတာ့ command အနေနဲ့ သုံးနိုင်ပါတယ် (slim image မှာ `curl` မရှိလျှင် Python one-liner နဲ့ အစားထိုးပါ)။
**Expected behavior:** Stack အား တစ်ခေါက်တည်းနဲ့ စတင်ပြီး `docker compose ps` မှာ service အားလုံး healthy ဖြစ်နေပြီး MCP tool ခေါ်ဆိုမှုနဲ့ worker queue processing နှစ်ခုစလုံး အလုပ်လုပ်နေပါတယ်။
