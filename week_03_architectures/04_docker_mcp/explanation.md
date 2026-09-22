# Containerization & MCP Integration

## 1. Python service များအတွက် Dockerfile

### ဘာကို ဆိုလိုတာလဲ
Dockerfile ဆိုသည်မှာ container image တစ်ခုကို ဘယ်လို တည်ဆောက်ရမလဲဆိုသည့် အဆင့်လိုက် ညွှန်ကြားချက်များ ပါဝင်သော ဖိုင်တစ်ခုဖြစ်သည်။ Python backend service တစ်ခုအတွက် မည်သည့် base image ကို အသုံးပြုမည်၊ dependency များကို မည်သို့ install လုပ်မည်၊ application code ကို မည်သည့်နေရာတွင် ထည့်သွင်းမည်ဆိုသည်ကို သတ်မှတ်ပေးသည်။

### ဘာကြောင့် လဲ
Python project တစ်ခုသည် လုပ်ဖော်ကိုင်ဖက်တစ်ယောက်၏ စက်ပေါ်တွင် အလုပ်လုပ်ပြီး အခြားစက်ပေါ်တွင် မလုပ်ခြင်းသည် နားရည်းစားမှု ဖြစ်တတ်သည်။ Python version ကွဲလွဲမှု၊ system library မတူမှုတို့ကြောင့်ဖြစ်သည်။ Dockerfile ဖြင့် environment တစ်ခုလုံးကို သေသေချာချာ သတ်မှတ်နိုင်သဖြင့် "ကျွန်ုပ်စက်ပေါ်တွင် လုပ်တယ်" ဟူသော ပြဿနာ ပျောက်သွားသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Dockerfile တစ်ခုတွင် `FROM` (base image ရွေးချယ်ခြင်း)၊ `WORKDIR` (အလုပ်လုပ်မည့် directory)၊ `COPY` (ဖိုင်များ ကူးယူခြင်း)၊ `RUN` (command ကို image build အတွင်း လုပ်ဆောင်ခြင်း)၊ `CMD` (container စတင်သည့်အခါ လုပ်ဆောင်မည့် command) စသော instruction များ အဆင့်လိုက် အလုပ်လုပ်သည်။ Docker engine သည် instruction တစ်ခုစီအတွက် layer တစ်ခု ဖန်တီးပြီး မပြောင်းလဲသော layer များကို cache မှ ပြန်အသုံးပြုသည်။

### ဥပမာ

```python
# Example: read a Dockerfile-like structure and print the build steps in order
dockerfile_steps = [
    "FROM python:3.12-slim",      # base image with Python preinstalled
    "WORKDIR /app",               # set working directory inside container
    "COPY requirements.txt .",    # copy dependency file first (better caching)
    "RUN pip install --no-cache-dir -r requirements.txt",
    "COPY . .",                   # copy the rest of the application code
    "CMD [\"uvicorn\", \"main:app\", \"--host\", \"0.0.0.0\", \"--port\", \"8000\"]",
]

for step in dockerfile_steps:
    print(step)
# Expected output:
# FROM python:3.12-slim
# WORKDIR /app
# COPY requirements.txt .
# RUN pip install --no-cache-dir -r requirements.txt
# COPY . .
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production တွင် application တစ်ခုကို server တစ်ခုမှ အခြားတစ်ခုသို့ ရွှေ့ယူရသည်မှာ မဖြစ်မနေ ဖြစ်လာသည်။ Dockerfile ရှိပါက `docker build` နှင့် `docker run` command နှစ်ခုသာလိုသဖြင့် deployment ကို တုန်းရိုက်၍ ပြန်လုပ်နိုင်သည်။ ထိ့အပြင် `requirements.txt` ကို code များမထက် အရင် COPY လုပ်ခြင်းဖြင့် code ပြင်တိုင်း dependency များကို ပြန် install မလုပ်ရသော layer caching အကျိုးကျေးဇူးလည်း ရရှိသည်။

## 2. Multi-stage builds

### ဘာကို ဆိုလိုတာလဲ
Multi-stage build ဆိုသည်မှာ Dockerfile တစ်ခုအတွင်း `FROM` statement အများအပြား ထည့်သွင်းပြီး ပထမအဆင့် (build stage) တွင် dependency များ၊ compiler toolchain များ အသုံးပြု၍ artifact များ တည်ဆောက်ပြီး နောက်ဆုံးအဆင့် (runtime stage) တွင် လိုအပ်သော ဖိုင်များကိုသာ `COPY --from` ဖြင့် ယူငင်သည့် နည်းစနစ်ဖြစ်သည်။

### ဘာကြောင့် လဲ
Python project တစ်ခုတွင် `pip install` လုပ်စဉ် compilation လိုအပ်သော package များအတွက် gcc၊ build header များ စသည့် ကိရိယာများ လိုအပ်တတ်သည်။ ၎င်းတို့အားလုံးကို final image တွင် ထားရှိပါက image အရွယ်အစား ကြီးမားပြီး လုံခြုံရေးအရလည်း အန္တရာယ် များသည်။ Multi-stage build ဖြင့် build ကိရိယာများကို စွန့်ပစ်၍ ပေါ့ပါးသော runtime image တစ်ခု ရရှိသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Docker သည် stage တစ်ခုစီကို သီးခြား environment အဖြစ် build လုပ်သည်။ အနည်းငယ်ဆုံး `AS builder` ကဲ့သို့ အမည်ပေးထားသော stage မှ လိုအပ်သည့်ဖိုင်များကိုသာ နောက်ဆုံး stage သို့ ကူးယူသည်။ ဥပမာအားဖြင့် virtual environment တစ်ခုတွင် dependency များကို install လုပ်ပြီး ၎င်း venv folder တစ်ခုလုံးကိုသာ runtime stage သို့ ကူးယူနိုင်သည်။

### ဥပမာ

```python
# Example: simulate which files survive from builder stage to runtime stage
builder_stage_contents = [
    "gcc", "python-dev headers", "pip cache",
    "/opt/venv/bin/", "/opt/venv/lib/python3.12/site-packages/",
]

# only paths under /opt/venv are copied in: COPY --from=builder /opt/venv /opt/venv
copied = [item for item in builder_stage_contents if item.startswith("/opt/venv")]

print("Builder stage:", builder_stage_contents)
print("Runtime stage :", copied)
# Expected output:
# Builder stage: ['gcc', 'python-dev headers', 'pip cache', '/opt/venv/bin/', '/opt/venv/lib/python3.12/site-packages/']
# Runtime stage : ['/opt/venv/bin/', '/opt/venv/lib/python3.12/site-packages/']
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Image အရွယ်အစားသည် deployment အမြန်နှုန်း၊ storage ကုန်ကျစရိတ်နှင့် attack surface အားလုံးကို သက်ရောက်သည်။ Production environment တွင် ပေါ့ပါး၍ မလိုအပ်သော ကိရိယာမပါဝင်သော image များကိုသာ အသုံးပြုသင့်သည်။ Multi-stage build သည် အဆင့်များကို Dockerfile တစ်ခုတည်းတွင် ထိန်းသိမ်းထားနိုင်သဖြင့် build logic ကိုလည်း ရိုးရှင်းစွာ စီမံခန့်ခွဲနိုင်သည်။

## 3. Compose wiring (api / worker / db / redis)

### ဘာကို ဆိုလိုတာလဲ
Docker Compose ဆိုသည်မှာ container များစွာကို တစ်ပြိုင်နက်တည်း ဖန်တီးပြီး တစ်ခုနှင့်တစ်ခု ချိတ်ဆက်ပေးသည့် ကိရိယာဖြစ်သည်။ API service၊ background worker၊ PostgreSQL ကဲ့သို့သော database၊ Redis ကဲ့သို့သော cache/message broker တို့ကို `compose.yaml` (ယခင် `docker-compose.yml`) ဖိုင်တစ်ခုတွင် သတ်မှတ်၍ တစ်ကြိမ်တည်း စတင်နိုင်သည်။

### ဘာကြောင့် လဲ
ခေတ်မီ AI backend တစ်ခုသည် service များစွာပါဝင်သည် — API က HTTP request လက်ခံသည်၊ worker က အချိန်ကြာရသော ML task များကို လုပ်ဆောင်သည်၊ db က data သိမ်းဆည်းသည်၊ redis က queue နှင့် cache အဖြစ် ဆောင်ရွက်သည်။ ၎င်းတို့အားလုံးကို လက်ဖြင့် `docker run` command တစ်ခုစီဖြင့် စတင်ခြင်းသည် အမှားအယွင်း များပြားသည်။ Compose ဖြင့် network နှင့် dependency များကို အလိုအလျောက် စီမံပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Compose သည် project တစ်ခုအတွက် default network တစ်ခု ဖန်တီးသည်။ Service တစ်ခုချင်းစီ၏ အမည်သည် network အတွင်းရှိ hostname အဖြစ် အသုံးပြုနိုင်သဖြင့် `postgres:5432` သို့မဟုတ် `redis:6379` ဟူ၍ service အမည်ဖြင့်ပင် ချိတ်ဆက်နိုင်သည်။ `depends_on` ဖြင့် စတင်ရမည့် အစဉ်အကိုလည်း ထိန်းနိုင်သည်။ `docker compose up` command တစ်ခုတည်းဖြင့် အားလုံးကို စတင်နိုင်သည်။

### ဥပမာ

```python
# Example: derive database and redis URLs from compose service names
services = {"api": None, "worker": None, "db": None, "redis": None}

# inside the compose network, service names act as hostnames
db_url = "postgresql://appuser:${DB_PASSWORD}@db:5432/appdb"
redis_url = "redis://redis:6379/0"

print("API container connects to DB at   :", db_url)
print("Worker container connects via     :", redis_url)
print("Services defined in compose.yaml  :", ", ".join(services.keys()))
# Expected output:
# API container connects to DB at   : postgresql://appuser:${DB_PASSWORD}@db:5432/appdb
# Worker container connects via     : redis://redis:6379/0
# Services defined in compose.yaml  : api, worker, db, redis
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Development အတွက်သာမက CI/CD pipeline တွင်လည်း integration test များ လုပ်ဆောင်ရန် service အားလုံးကို မိနစ်ပိုင်းအတွင်း ထိုးထွက်နိုင်ရန် လိုအပ်သည်။ Compose file တစ်ခုက စနစ်တစ်ခုလုံး၏ ဖွဲ့စည်းပုံကို code အဖြစ် မှတ်တမ်းတင်ပေးသဖြင့် အဖွဲ့ဝင်တိုင်း တူညီသော environment တွင် လုပ်ဆောင်နိုင်သည်။

## 4. Secrets များကို environment variable အဖြစ် ထည့်သွင်းခြင်း

### ဘာကို ဆိုလိုတာလဲ
Secret ဆိုသည်မှာ API key၊ database password၊ token ကဲ့သို့ လုံခြုံစွာ သိမ်းဆည်းရမည့် တန်ဖိုးများဖြစ်သည်။ ၎င်းတို့ကို code ထဲတွင် တိုက်ရိုက် ရေးသွင်းခြင်းအစား environment variable များမှတစ်ဆင့် container ထဲသို့ ပို့လွှတ်ခြင်းဖြစ်သည်။ Compose တွင် `.env` ဖိုင်များ သို့မဟုတ် secrets section ဖြင့် ထည့်သွင်းနိုင်သည်။

### ဘာကြောင့် လဲ
Password သို့မဟုတ် API key ကို code repository ထဲသို့ တိုက်ရိုက် ထည့်သွင်းပါက တစ်ကြိမ် leaked ဖြစ်ပါက ဆယ်နှစ်ပတ်လုံး အန္တရာယ်ရှိသည် — git history မှ ပျောက်ရခက်သည်။ Environment variable ဖြင့် ခွဲခြားထားပါက development နှင့် production အတွက် တူညီသော code၊ ကွဲပြားသော တန်ဖိုးများ အသုံးပြုနိုင်ပြီး secret များကို repo ထဲမှ ဖယ်ရှားနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Compose တွင် `environment:` key ဖြင့် variable များ သတ်မှတ်နိုင်ပြီး `${VAR_NAME}` ရေးပုံဖြင့် ဆိုက်ကြည့်ရှုနေရာမှ အလိုအလျောက် ဖတ်ယူသည်။ `.env` ဖိုင်ကို `.gitignore` တွင် ထည့်သွင်းထားရမည်။ Container အတွင်းရှိ Python code က `os.environ` ဖြင့် ဖတ်ယူနိုင်သည်။ Variable မရှိပါက fail-fast ဖြစ်စေရန် စစ်ဆေးခြင်းသည် နည်းလမ်းကောင်းဖြစ်သည်။

### ဥပမာ

```python
import os

# Read required secrets from environment variables; fail fast if missing
def get_required_env(name: str) -> str:
    value = os.environ.get(name)
    if not value:
        raise RuntimeError(f"Missing required environment variable: {name}")
    return value

# Simulate the container environment
os.environ["DATABASE_URL"] = "postgresql://appuser:secret123@db:5432/appdb"
os.environ["OPENAI_API_KEY"] = "sk-example-not-a-real-key"

print("DATABASE_URL host part:", get_required_env("DATABASE_URL").split("@")[-1])
print("API key loaded from env:", bool(get_required_env("OPENAI_API_KEY")))
# Expected output:
# DATABASE_URL host part: db:5432/appdb
# API key loaded from env: True
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production secret တစ်ခု leaked ဖြစ်ခြင်းသည် ကုမ္ပဏီတစ်ခုအတွက် စီးပွားရေးနှင့် ကိုယ်လက်ခံခြင်း (reputation) ဆုံးရှုံးမှု ဖြစ်စေနိုင်သည်။ MCP server တစ်ခုသည် အခြား service များနှင့် ချိတ်ဆက်ရန် API key များ လိုအပ်သဖြင့် ဤနည်းလမ်းသည် Docker-based MCP setup တွင် မဖြစ်မနေ လိုအပ်သည်။ အရေးကြီးသော အချက်များမှာ —

- **`.gitignore` တွင် `.env` ထည့်ရန်** — secret ပါသော ဖိုင်ကို repo ထဲ မတက်စေရန်။
- **Fail-fast စစ်ဆေးရန်** — server စတင်ချိန်မှာပဲ variable ရှိမရှိ စစ်ပြီး မရှိလျှင် ချက်ချင်း error ပြရန်၊ နောက်ပိုင်းမှာ ရှာပါ ခက်စေရန်။
- **Default value မထည့်ရန်** — `os.environ.get("KEY", "hardcoded-default")` ကဲ့သို့ ရေးပါက secret တစ်ခု မသတ်မှတ်ဘဲ server လည်နေနိုင်ပြီး အမှားကို မြင်ရခက်သည်။

### Compose ဖြင့် တွဲစပ်အသုံးပြုပုံ

`.env` ဖိုင် (repo ထဲ မထည့်ရ):

```text
OPENAI_API_KEY=sk-real-key-goes-here
DATABASE_URL=postgresql://appuser:strongpassword@db:5432/appdb
```

`docker-compose.yml` တွင် Compose က တူညီသော နာမည်ပေးထားသော `.env` ဖိုင်ကို အလိုအလျောက် ဖတ်ယူသည်:

```yaml
services:
    mcp-server:
        build: .
        environment:
            - OPENAI_API_KEY=${OPENAI_API_KEY}
            - DATABASE_URL=${DATABASE_URL}
```

ဤနည်းဖြင့် code တွင် secret တစ်ခုမျှ မရေးသွင်းဘဲ၊ development နှင့် production အတွက် `.env` ဖိုင် ကွဲပြားသော တန်ဖိုးများသာ ပြောင်းလဲပေးခြင်းဖြင့် တူညီသော image ကို အသုံးပြုနိုင်သည်။

### အနှစ်ချုပ်
Secret များကို code ထဲ မရေးဘဲ environment variable များဖြင့် container ထဲ ပို့ပါ။ `.env` ကို `.gitignore` ထည့်ပါ။ Server စတင်ချိန်တွင် fail-fast စစ်ဆေးခြင်းဖြင့် အမှားများကို စောစောသိရန် ပြုလုပ်ပါ။
