## လေ့ကျင့်ခန်း ၁ — Python ဝန်ဆောင်မှုအတွက် အခြေခံ Dockerfile

```python
# check_dockerfile.py — verify the basic Dockerfile structure is valid
EXPECTED_LINES = [
    "FROM python:3.12-slim",
    "WORKDIR /app",
    "COPY requirements.txt .",
    "RUN pip install --no-cache-dir -r requirements.txt",
    "COPY . .",
    "CMD [\"uvicorn\", \"main:app\", \"--host\", \"0.0.0.0\", \"--port\", \"8000\"]",
]

with open("Dockerfile", "r", encoding="utf-8") as f:
    content = f.read()

missing = [line for line in EXPECTED_LINES if line not in content]
if missing:
    print("Missing recommended lines:")
    for line in missing:
        print(" -", line)
else:
    print("Dockerfile contains all recommended base steps.")
```

**အဓိကအယူအဆ** — လိုအပ်သော dependency များကို အရင် copy လုပ်ပြီး install ခြင်းအားဖြင့် Docker layer cache ကို ထိထိရောက်ရောက် အသုံးချနိုင်သည်။

## လေ့ကျင့်ခန်း ၂ — Multi-stage build ဖြင့် image အရွယ်အစား လျှော့ချခြင်း

```python
# validate_multistage.py — check that the Dockerfile uses at least two stages
with open("Dockerfile", "r", encoding="utf-8") as f:
    from_count = sum(
        1 for line in f
        if line.strip().upper().startswith("FROM")
    )

if from_count < 2:
    print(f"Only {from_count} stage(s) found; multi-stage needs >= 2 FROM lines.")
else:
    print(f"Multi-stage build detected with {from_count} stages.")

# A typical two-stage layout:
# FROM python:3.12-slim AS builder
#     RUN pip install --prefix=/install -r requirements.txt
# FROM python:3.12-slim
#     COPY --from=builder /install /usr/local
#     COPY . /app
```

**အဓိကအယူအဆ** — build tool များနှင့် intermediate artifact များကို နောက်ဆုံး image ထဲ မထည့်ဘဲ runtime အတွက် လိုအပ်သည့်အပိုင်းကိုသာ ကူးယူခြင်းဖြင့် image ကို သန့်ရှင်းစွာ ထိန်းထားနိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — Compose ဖြင့် api/worker/db/redis ချိတ်ဆက်ခြင်း

```python
# parse_compose.py — read compose.yaml and list each service and its image
import re

with open("compose.yaml", "r", encoding="utf-8") as f:
    text = f.read()

# Match top-level service names under the "services:" key
service_block = text.split("services:", 1)[-1]
services = re.findall(r"^  ([a-zA-Z0-9_-]+):", service_block, re.MULTILINE)

expected = {"api", "worker", "db", "redis"}
found = set(services)
print("Services found:", sorted(found))
print("Missing expected services:", sorted(expected - found) or "none")

# Inside compose.yaml each service uses the shared network name,
# so "redis" and "db" resolve via Docker's internal DNS.
```

**အဓိကအယူအဆ** — Compose ရှိ service အမည်များသည် ကိုယ်ပိုင် DNS အမည်အဖြစ် အလိုအလျောက် အသုံးပြုနိုင်သဖြင့် hostname ကို hardcoded IP မလိုအပ်ဘဲ ခေါ်ဆိုနိုင်သည်။

## လေ့ကျင့်ခန်း ၄ — Secret များကို environment variable များအဖြစ် ထည့်သွင်းခြင်း

```python
# check_env.py — fail fast if required secrets are missing at startup
import os
import sys

REQUIRED_KEYS = ["DATABASE_URL", "REDIS_URL", "API_SECRET_KEY"]

missing = [key for key in REQUIRED_KEYS if not os.environ.get(key)]

if missing:
    sys.exit(f"Missing required environment variables: {', '.join(missing)}")

print("All required environment variables are present.")
# Compose can load these from a .env file placed next to compose.yaml;
# never commit the real .env file to version control.
```

**အဓိကအယူအဆ** — Secret များကို code တွင် တိုက်ရိုက်ရေးသွင်းခြင်း မပြုဘဲ environment variable မှတဆင့် ယူသုံးပြီး မရှိပါက application စတင်ချိန်တွင် ချက်ချင်း ရပ်တန့်စေခြင်းသည် ဘေးအန္တရာယ် နည်းစေသည်။

## လေ့ကျင့်ခန်း ၅ — Backend ဝန်ဆောင်မှုတွင် MCP server ထုတ်ပြခြင်း

```python
# mcp_server.py — minimal MCP server exposed over HTTP using FastMCP
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("inventory-service")

@mcp.tool()
def check_stock(item_id: str) -> str:
    # Example tool an AI client can call
    return f"Stock lookup for {item_id}: 42 units on hand"

if __name__ == "__main__":
    # Streams JSON-RPC over stdio; see modelcontextprotocol.io docs
    mcp.run()
```

**အဓိကအယူအဆ** — MCP protocol သည် tools နှင့် data sources များကို ပြောင်းလွယ်Flexibလသော interface ဖြင့် AI client များထံ တည်ငြိမ်စွာ တင်ပြပေးနိုင်သောကြောင့် backend ဝန်ဆောင်မှုတစ်ခုအတွင်း တွဲဖက်ထည့်သွင်းရလွယ်ကူသည်။

## လေ့ကျင့်ခန်း ၆ — Container အတွင်း MCP နှင့် API ကို တွဲဖက် run ခြင်း

```python
# main.py — FastAPI app that also mounts the MCP server route
from fastapi import FastAPI
from mcp.server.fastmcp import FastMCP

app = FastAPI()
mcp = FastMCP("backend-mcp")

@mcp.tool()
def list_models() -> list[str]:
    # A simple tool exposing model metadata to AI clients
    return ["gpt-class", "claude-class", "local-class"]

@app.get("/health")
def health() -> dict:
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**အဓိကအယူအဆ** — ရိုးရာ HTTP API နှင့် MCP tool များကို container တစ်ခုတည်းအတွင်း ပူးတွဲထားခြင်းဖြင့် deployment ရိုးရိုးရှင်းရှင်းဖြင့် AI agent များအတွက် လုပ်ဆောင်ချက်များ ပေးနိုင်သည်။
