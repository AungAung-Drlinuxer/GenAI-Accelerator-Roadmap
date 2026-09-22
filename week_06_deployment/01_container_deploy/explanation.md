# Container Deploy & Reverse Proxy with TLS — ရှင်းလင်းချက်

ဒီသင်ခန်းစာတွင် production deployment အတွက် အခြေခံကျသော အစိတ်အပိုင်း ငါးမျိုးကို အဆင့်ဆင့် လေ့လာမည်။

## ၁။ Container Image Build နှင့် Ship လုပ်ခြင်း

### ဘာကို ဆိုလိုတာလဲ

Docker image ဆိုသည်မှာ application ၏ code၊ dependencies နှင့် runtime settings အားလုံးကို ထည့်သွင်းထားသော အသင့်အသုံးပြုနိုင်သည့် package တစ်ခုဖြစ်သည်။ Image ကို build ပြီး registry (ဥပမာ Docker Hub) သို့ push လုပ်ခြင်းကို "shipping" ဟု ခေါ်သည်။

### ဘာကြောင့် လဲ

"ကျွန်တော့် machine မှာ အလုပ်လုပ်တယ်" ဟူသော ပြဿနာကို Docker က ဖြေရှင်းပေးသည်။ Image ထဲတွင် လိုအပ်ချက်အားလုံး ပါဝင်သောကြောင့် မည်သည့် server ပေါ်တွင်မဆို တူညီသော ရလဒ် ရရှိသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

ပထမဆုံး application အတွက် `Dockerfile` ရေးသည်၊ ၎င်းမှ `docker build` ဖြင့် image ဖန်တီးသည်၊ ထို့နောက် `docker push` ဖြင့် registry သို့ တင်သည်၊ နောက်ဆုံး server ပေါ်တွင် `docker pull` ဖြင့် ဆွဲချ ဖွင့်လှစ်သည်။

### ဥပမာ

```python
# demo_version_check.py
# Simulate the version info that a containerized app exposes.
import platform

APP_VERSION = "1.0.0"

print(f"App version : {APP_VERSION}")
print(f"Python     : {platform.python_version()}")
# Expected output:
# App version : 1.0.0
# Python     : 3.11.8
```

`Dockerfile` ဥပမာ (reference only):

```text
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Registry ထဲသို့ တင်ထားသော image တစ်ခုကို မည်သည့်အခါမျှ ပြန်၍ မွေးဖွင့်စရာ မလိုပဲ version tag အလိုက် ရယူနိုင်သဖြင့် rollback လုပ်ခြင်း၊ စမ်းသပ်ခြင်းနှင့် scaling လုပ်ခြင်းတို့ကို လွယ်ကူစေသည်။

## ၂။ Environment နှင့် Config Separation

### ဘာကို ဆိုလိုတာလဲ

Application code နှင့် ပတ်ဝန်းကျင်အလိုက် ပြောင်းလဲနိုင်သော configuration (API keys၊ database URLs၊ model paths) တို့ကို သီးခြားခွဲခြားထားခြင်းဖြစ်သည်။ ပုံမှန်အားဖြင့် environment variables များကို အသုံးပြုသည်။

### ဘာကြောင့် လဲ

Secret keys များကို code repository ထဲတွင် မထည့်သွင်းရပါ။ Environment တစ်ခုစီ (development၊ staging၊ production) အတွက် တူညီသော image ကိုပင် ကွဲပြားသော config ဖြင့် ဖွင့်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Python တွင် `os.environ` သို့မဟုတ် `os.getenv` ဖြင့် environment variable များကို ဖတ်ယူသည်။ Docker တွင် `--env-file` ဖြင့် `.env` file မှ တန်ဖိုးများကို container ထဲသို့ ထည့်ပေးသည်။

### ဥပမာ

```python
# demo_config.py
import os

def get_config():
    # Read settings from environment variables with safe defaults.
    return {
        "app_env": os.getenv("APP_ENV", "development"),
        "api_key": os.getenv("API_KEY", "not-set"),
        "model_path": os.getenv("MODEL_PATH", "/models/default"),
    }

if __name__ == "__main__":
    cfg = get_config()
    for key, value in cfg.items():
        print(f"{key} = {value}")
# Expected output:
# app_env = development
# api_key = not-set
# model_path = /models/default
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production API key တစ်ခု အလွဲသော GitHub repository ထဲ ရောက်သွားခြင်းက လုံခြုံရေး ထိခိုက်မှု အကြီးစားဖြစ်သည်။ Config ခွဲထားခြင်းဖြင့် ဤအန္တရာယ်ကို ရှောင်ရှားနိုင်ပြီး image တစ်ခုတည်းကို နေရာအများစုတွင် အသုံးပြုနိုင်သည်။

## ၃။ Caddy Reverse Proxy နှင့် Automatic HTTPS

### ဘာကို ဆိုလိုတာလဲ

Reverse proxy ဆိုသည်မှာ client များ၏ request များကို လက်ခံပြီး နောက်ကျောရှိ application server များဆီ ထပ်မံ ပို့ဆောင်ပေးသည့် ဝန်ဆောင်မှုတစ်ခုဖြစ်သည်။ Caddy သည် Let's Encrypt မှ certificate များကို အလိုအလျောက် ရယူပြီး ပြန်လည် အသစ်ပြုလုပ်ပေးသည်။

### ဘာကြောင့် လဲ

HTTPS ကို လက်ဖြင့် စီမံရန်မှာ certificate request၊ renewal၊ reload စသည့် အဆင့်များစွာ ပါဝင်သည်။ Caddy ၏ Automatic HTTPS က ထိုအလုပ်အားလုံးကို အလိုအလျောက် လုပ်ဆောင်ပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`Caddyfile` တွင် domain name နှင့် ထို domain မှ ပို့ဆောင်ရမည့် internal service address ကို သတ်မှတ်သည်။ Caddy စတင်သောအခါ Let's Encrypt ဆီမှ certificate တောင်းခံပြီး port 80/443 တွင် TLS traffic ကို လက်ခံသည်။

`Caddyfile` ဥပမာ (reference only):

```text
api.example.com {
    reverse_proxy app:8000
}
```

### ဥပမာ

```python
# demo_proxy_route.py
# Simple simulation of how a reverse proxy maps host rules to backends.
routes = {
    "api.example.com/health": "app:8000/health",
    "api.example.com/predict": "app:8000/predict",
}

for public_path, backend_path in routes.items():
    print(f"{public_path}  ->  {backend_path}")
# Expected output:
# api.example.com/health  ->  app:8000/health
# api.example.com/predict  ->  app:8000/predict
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Browser များနှင့် mobile clients များက HTTPS မရှိသော API ကို တဖြည်းဖြည်း ပယ်ရှားလာကြသည်။ Caddy က config စာလုံး နှစ်ကြောင်းဖြင့် ထိုပြဿနာကို ဖြေရှင်းပေးသဖြင့် အချိန်ကုန်သက်သာပြီး လုံခြုံရေးလည်း တိုးတက်သည်။

## ၄။ Health Checks

### ဘာကို ဆိုလိုတာလဲ

Health check ဆိုသည်မှာ application က မိမိကိုယ်ကို "ရှင်သန်နေသည်၊ ဝန်ဆောင်မှုပေးနိုင်သည်" ဟု ဖြေဆိုသည့် endpoint တစ်ခုဖြစ်သည်။ Docker သည် ၎င်းကို အချိန်အခါအလိုက် စစ်ဆေးပြီး container ကို ပြန်လည်စတင်နိုင်သည်။

### ဘာကြောင့် လဲ

Process တစ်ခု ရှင်သန်နေသော်လည်း dependency (database၊ model file) နှင့် ချိတ်ဆက်မှု ပျက်နေနိုင်သည်။ Health check က ဤကွာခြားချက်ကို ဖမ်းဆုပ်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

FastAPI (သို့မဟုတ် တခြား framework) တွင် `/health` route တစ်ခု ရေးသည်၊ Dockerfile တွင် `HEALTHCHECK` directive ဖြင့် ၎င်းကို စစ်ရမည့် command သတ်မှတ်သည်။ Docker က `starting`၊ `healthy`၊ `unhealthy` ဟူ၍ အခြေအနေ သုံးမျိုး မှတ်ယူသည်။

### ဥပမာ

```python
# demo_health.py
# A tiny HTTP health server using only the standard library.
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import datetime

class HealthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            body = json.dumps({
                "status": "ok",
                "time": datetime.datetime.utcnow().isoformat() + "Z",
            }).encode()
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
        else:
            self.send_response(404)
            self.end_headers()

if __name__ == "__main__":
    # Run with a real server; visiting /health returns status ok.
    print("Starting health server on :8080 ... try GET http://localhost:8080/health")
    HTTPServer(("0.0.0.0", 8080), HealthHandler).serve_forever()
# Expected output (when run):
# Starting health server on :8080 ... try GET http://localhost:8080/health
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Load balancer များနှင့် orchestrator များက health status အပေါ် မူတည်၍ traffic ပို့ကြသည်။ Health check မရှိပါက ပျက်နေသော container ထံသို့ user request များ ဆက်လက် ရောက်ရှိမည်ဖြစ်သည်။

## ၅။ Zero-Downtime Restarts

### ဘာကို ဆိုလိုတာလဲ

Application ကို update လုပ်နေစဉ် ဝန်ဆောင်မှု ရပ်တန့်မှု မရှိစေဘဲ deployment ပြောင်းလဲနိုင်ခြင်းကို ဆိုလိုသည်။

### ဘာကြောင့် လဲ

AI inference API ကို အသုံးပြုနေသော user များအတွက် စက္ကန့်အနည်းငယ် ပိတ်သွားခြင်းပင် error များ ဖြစ်ပေါ်စေနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

နည်းလမ်း အခြေခံမှာ — container အသစ်ကို စတင်ပြီး healthy ဖြစ်မှ၊ proxy က traffic ကို အသစ်ဆီ လွှဲပြောင်းပြီးနောက်၊ container အရင်ကို ပိတ်ခြင်းဖြစ်သည်။ Caddy compose setup တွင် `docker compose up -d` ဖြင့် app container ကို အစားထိုးလျှင် proxy က အလိုအလျောက် ချိတ်ဆက်ပေးသည်။

### ဥပမာ

```python
# demo_blue_green.py
# Simulate a blue/green switch without dropping any request slot.
class Deployment:
    def __init__(self):
        self.active = "blue"
        self.running = {"blue": True, "green": False}

    def deploy_new_version(self):
        # Start green, verify health, then switch traffic.
        self.running["green"] = True
        if self.running["green"]:
            self.active = "green"
            self.running["blue"] = False  # old version stopped later
        return self.active

d = Deployment()
print("active before:", d.active)
print("active after :", d.deploy_new_version())
# Expected output:
# active before: blue
# active after : green
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Model update လုပ်ရန် API ကို အပြင်းအထန် ပိတ်ရန် မလိုအပ်တော့ပါ။ အသစ် version ကို စမ်းသပ်ပြီးမှ အလုပ်လုပ်နေသော traffic ကို ရုတ်တရက် ရွှေ့နိုင်သဖြင့် user များ သို့မသိ ဖြစ်နေမည်ဖြစ်သည်။

## အနှစ်ချုပ်

ဤ module တွင် image build/shipping၊ config separation၊ Caddy reverse proxy ၏ automatic HTTPS၊ health check များနှင့် zero-downtime restart pattern တို့ကို လေ့လာခဲ့သည်။ ဤအစိတ်အပိုင်းများ ပေါင်းစည်းမှုဖြင့် AI application တစ်ခုကို လုံခြုံစွာ၊ ယုံကြည်စွာ production ပတ်ဝန်းကျင်သို့ တင်နိုင်မည်ဖြစ်သည်။
