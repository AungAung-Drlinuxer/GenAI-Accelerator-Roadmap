## လေ့ကျင့်ခန်း ၁ — Git ထဲမှ Secret ရှာဖွေပြီး ဖယ်ရှားခြင်း

repository တစ်ခုထဲတွင် ရှိပြီးသား API key များကို ရှာဖွေပါ။ အောက်ပါ Python script ကို ရေးပြီး ဖိုင်တွေအတွင်းရှိ key pattern များကို စစ်ဆေးပါ။

```python
import re
import pathlib

# Patterns that look like common secrets
patterns = [
    r"sk-[A-Za-z0-9]{20,}",
    r"AKIA[0-9A-Z]{16}",
    r"ghp_[A-Za-z0-9]{36}",
]

for path in pathlib.Path(".").rglob("*"):
    if path.suffix in (".py", ".env", ".yml", ".md"):
        try:
            text = path.read_text(encoding="utf-8")
        except OSError:
            continue
        for pat in patterns:
            for match in re.finditer(pat, text):
                print(f"{path}: possible secret -> {match.group()[:12]}...")
```

တွေ့ရှိသော key များကို ဖိုင်မှ ဖျက်ပြီး `.env` ဖိုင်ကို `.gitignore` တွင် ထည့်သွင်းပါ။ key တစ်ခုလုံး ဖယ်ရှားရန် `git filter-repo` သို့မဟုတ် provider ၏ secret-scanning tool ကို သုံးပါ။

**Hints:** `git log -p | grep -i "key"` ဖြင့် history ထဲမှာ ရှိသလား စစ်နိုင်သည်။ commit history ထဲရှိ key ကို ဖျက်ပြီးပါက key အသစ် issue လုပ်ရန် မမေ့ပါနှင့်။
**Expected behavior:** ဖိုင်တွင် key မတွေ့တော့ပါက script က output မထုတ်ပါ။ `.gitignore` တွင် `.env` ပါဝင်ပြီး `git status` တွင် `.env` မပေါ်တော့ပါ။

## လေ့ကျင့်ခန်း ၂ — Python အက်ပ်တွင် Environment Variable မှ Secret ဖတ်ခြင်း

Hardcoded key အစား environment variable မှ ဖတ်သည့် configuration class တစ်ခုကို ရေးပါ။

```python
import os

class Settings:
    def __init__(self):
        self.api_key = os.environ.get("OPENAI_API_KEY")
        if not self.api_key:
            raise RuntimeError("OPENAI_API_KEY is not set")

settings = Settings()
# Never log the full key, only a short prefix
print("Loaded key:", settings.api_key[:6] + "...")
```

shell တွင် `export OPENAI_API_KEY=sk-xxxx` ဟု သတ်မှတ်ပြီး script ကို ဒါရိုက်တာအတွင်းမှ သက်သက် run ကြည့်ပါ။

**Hints:** `.env` ဖိုင်နှင့် `python-dotenv` package ကို local development အတွက် သုံးနိုင်သည် — သို့သော် `.env` ကို git ထဲ မတင်ပါနှင့်။ production တွင် vault ထံမှ inject လုပ်ပါ။
**Expected behavior:** Key ရှိပါက `Loaded key: sk-abc...` ကဲ့သို့ prefix တစ်ခုသာ ပေါ်ပြီး၊ key မရှိပါက `RuntimeError` တက်သည်။ Full key ဘယ်တုန်းကမှ log မထွက်ပါ။

## လေ့ကျင့်ခန်း ၃ — Least-Privilege Database Role တည်ဆောက်ခြင်း

PostgreSQL (သို့မဟုတ် SQLite ဖြင့် ပွင့်လင်းသော demo) တွင် app အတွက် read/write role နှင့် analyst အတွက် read-only role ခွဲပါ။ PostgreSQL ဥပမာ —

```sql
-- App role: can only touch its own table
CREATE ROLE app_user LOGIN PASSWORD 'from_vault';
GRANT SELECT, INSERT, UPDATE, DELETE ON messages TO app_user;

-- Analyst role: read only
CREATE ROLE analyst_ro LOGIN PASSWORD 'also_from_vault';
GRANT SELECT ON messages TO analyst_ro;
```

`analyst_ro` ဖြင့် login လုပ်ပြီး `UPDATE` ကို ကြိုးစားကြည့်ပါ။

**Hints:** `REVOKE ALL ON DATABASE mydb FROM PUBLIC;` ဖြင့် ပုံသေ privilege များ ဖြတ်တောက်ပါ။ လက်တွေ့ production တွင် password ကို vault မှ generate လုပ်ပါ။
**Expected behavior:** `analyst_ro` ၏ `UPDATE` သည် `permission denied` error တက်ပြီး `app_user` မှလည်း အခြား table များကို မမြင်ရပါ။

## လေ့ကျင့်ခန်း ၄ — Network Boundary စိတ်ကူးယဉ် ပုံဆွဲခြင်း

LLM application တစ်ခုအတွက် network zones များ ပါဝင်သော diagram (စာသားဖြင့် သို့မဟုတ် draw.io) ရေးဆွဲပါ — public ingress, app tier, database tier, external LLM API egress။ ပြီးပါက egress ကို စာရင်းပြုစုပါ —

```yaml
# Kubernetes NetworkPolicy: app pods may only reach the LLM API domain via egress proxy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-llm-egress-only
spec:
  podSelector:
    matchLabels:
      app: llm-backend
  policyTypes: ["Egress"]
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: egress-proxy
      ports:
        - protocol: TCP
          port: 8080
```

**Hints:** Pod တစ်ခုကို rule မသတ်မှတ်ထားပါက default မှာ အားလုံးကို ချိတ်နိုင်သည် (cluster ပေါ်မူတည်သည်)။ `defaultDeny` policy ဖြင့် စတင်ပါ။
**Expected behavior:** Diagram တွင် database သည် app tier မှလွဲပြီး လမ်းမရှိပါ။ NetworkPolicy အား apply လုပ်ပြီးပါက `llm-backend` pod သည် တိုက်ရိုက် internet ကို မရောက်နိုင်တော့ပါ။

## လေ့ကျင့်ခန်း ၅ — Dependency နှင့် Image Scanning

Python dependency များနှင့် Docker image ကို scan လုပ်ပါ။ ပထမဆုံး `requirements.txt` ဖိုင်ကို စစ်ပါ —

```bash
# Generate a lock file with hashes, then audit
pip install safety
safety check -r requirements.txt

# Scan the built container image
docker build -t llm-app:local .
trivy image llm-app:local
```

ရလဒ်ထဲက High/Critical finding တစ်ခုကို ရွေးပြီး package version တင်ခြင်း (သို့မဟုတ် pin ပြောင်းခြင်း) ဖြင့် ပြင်ဆင်ပြီး ပြန် scan လုပ်ပါ။

**Hints:** `pip list --outdated` ဖြင့် မီးမရှိး package များကို မြင်နိုင်သည်။ base image ကို `python:3.12-slim` ကဲ့သို့ သေးသွားသော official image သို့ ပြောင်းခြင်းက လမ်းကြောင်းအများအပြား လျှော့စေသည်။
**Expected behavior:** ဒုတိယအကြိမ် scan တွင် ရွေးချယ်ထားသော finding သည် ပျောက်ကွယ်သွားပြီး report ထဲတွင် ထပ်မပေါ်တော့ပါ (ကျန် finding များ အားလုံး မပြောပါနှင့် — တဖြည်းဖြည်း ပြင်သွားနိုင်သည်)။

## လေ့ကျင့်ခန်း ၆ — Audit Logging နှင့် Key Rotation Pipeline

Request တိုင်းကို log လုပ်ပြီး နောက်ဆုံးရ key version ကိုသာ သုံးသည့် rotation-aware client ရေးပါ။

```python
import time
import logging
import hashlib

logging.basicConfig(
    filename="audit.log",
    level=logging.INFO,
    format="%(asctime)s actor=%(name)s event=%(message)s",
)

class RotatingKeyClient:
    def __init__(self):
        # In production, fetch the current key from Vault API
        self.key = "v1-demo-key"
        self.key_version = 1

    def rotate(self, new_key):
        self.key = new_key
        self.key_version += 1
        logging.info(f"key_rotated version={self.key_version} "
                     f"fingerprint={hashlib.sha256(new_key.encode()).hexdigest()[:8]}")

    def call(self, actor, prompt):
        # Log metadata only - never the prompt content or the key
        logging.info(f"llm_call actor={actor} key_version={self.key_version} "
                     f"prompt_len={len(prompt)}")
        return "response"

client = RotatingKeyClient()
client.call("user123", "hello")
client.rotate("v2-demo-key")
client.call("user456", "another prompt")
```

Log ဖိုင်ကို ဖွင့်ကြည့်ပြီး rotation event မတိုင်မီ/နောက် key version ပြောင်းသွားပုံကို စစ်ဆေးပါ။

**Hints:** Vault ရဲ့ KV secrets engine တွင် `secret/data/llm-key` ကို version ခွဲ၍ သိမ်းနိုင်သည် — `X-Vault-Index` သို့မဟုတ် metadata `version` ဖြင့် လက်ရှိ version ကို သိနိုင်သည်။ Rotation ပြီးပါက ရက်ပေါင်း ၂၈ ကြာပါက အနည်းဆုံး အရင် key ကို revoke လုပ်ပါ။
**Expected behavior:** `audit.log` ထဲတွင် `key_rotated` event တစ်ကြိမ်နှင့် `llm_call` event နှစ်ကြိမ် ပေါ်ပြီး၊ key တစ်ခုလုံး သို့မဟုတ် prompt အပြည့်အစုံ မည်သည့် log line တွင်မှ မပါဝင်ပါ။
