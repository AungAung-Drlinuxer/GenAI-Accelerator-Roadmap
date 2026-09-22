# Week 6 — Security Hardening & Secrets Management

## ၁. Secrets ကိ Vault ထဲမှာ သိမ်းခြင်း (Git ထဲ မထည့်ရ)

### ဘာကို ဆိုလိုတာလဲ
Secrets ဆိုတာက API keys, database passwords, tokens စသည့့ လျှို့ဝှက်သတင်းအချက်အလက်များဖြစ်သည်။ ဒီအရာများကို source code repository (git) ထဲတွင် တိုက်ရိုက်ရေးသားထားခြင်း မပြုဘဲ HashiCorp Vault ကဲ့သို့သော secrets management system တစ်ခုထဲတွင် သိမ်းဆည်းထားရမည်။

### ဘာကြောင့် လဲ
Git history သည် ပျက်စီးရန် ခက်ခဲသည်။ Secret တစ်ခုကို commit လုပ်မိပါက commit history တစ်လျှောက် ကျန်ရစ်မည်ဖြစ်ပြီး repo ကို share လုပ်သည့်အခါ သူများ မြင်နိုင်သည်။ Public repo တစ်ခုတွင် secret  leaked ဖြစ်မှုက bot များက မိနစ်ပိုင်းအတွင်း ရှာတွေ့တတ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Vault သည် secrets များကို encrypted ဖြင့် သိမ်းဆည်းပေးပြီး application က authentication ပြုလုပ်ပြီးသောအခါ အချိန်ကန့်သတ်ချက်ဖြင့် secret ကို ယူနိုင်စေသည်။ Application ဘက်တွင် environment variable များမှတဆင့် သို့မဟုတ် Vault API မှတဆင့် လက်တွေ့ရှာယူသည်။ Code ထဲတွင် secret တစ်ခုမျှ ရေးထားစရာ မလိုပေ။

### ဥပမာ

```python
import os
import hvac

# Connect to Vault using a short-lived token from the environment
client = hvac.Client(
    url=os.environ["VAULT_ADDR"],
    token=os.environ["VAULT_TOKEN"],
)

# Read the database password from Vault instead of hardcoding it
secret = client.secrets.kv.v2.read_secret_version(
    path="prod/db", mount_point="secret"
)
db_password = secret["data"]["data"]["password"]

print("Password loaded from Vault:", bool(db_password))
print("Password length:", len(db_password))

# Expected output:
# Password loaded from Vault: True
# Password length: 24
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production AI application တစ်ခုတွင် OpenAI API key, database password, vector DB credentials များ လိုအပ်သည်။ Secret တစ်ခု leaked ဖြစ်ပါက ငွေကုန်ကျသည့် API billing များ သူများက အသုံးပြုနိုင်ပြီး data breach ဖြစ်စေနိုင်သည်။ OWASP LLM Top 10 တွင် LLM06 (Sensitive Information Disclosure) ကို အဓိက ခြိမ်းခြောက်မှုအဖြစ် ဖော်ပြထားသည်။

---

## ၂. Least-Privilege DB Roles (အနည်းဆုံး အခွင့်အရေးပေးခြင်း)

### ဘာကို ဆိုလိုတာလဲ
Application တစ်ခုက database တွင် လုပ်ဆောင်ခွင့်ရှိသမျှ လုံလောက်သလောက်သာ အခွင့်အရေးရရန် သတ်မှတ်ခြင်းဖြစ်သည်။ ဥပမာ — chatbot application က embedding ဖတ်ရုံသာ လိုပါက `SELECT` ခွင့်သာပေးပြီး `DROP TABLE` ခွင့် မပေးရ။

### ဘာကြောင့် လဲ
Application တစ်ခုတွင် SQL injection သို့မဟုတ် prompt injection မှတဆင့် တိုက်ခိုက်မှုဖြစ်ပါက attacker သည် admin account အခွင့်အရေးအားလုံး မရရှိစေရန်အတွက်ဖြစ်သည်။ ခွင့်ပြုချက်နည်းလျှင် ပျက်စီးမှုအန္တရယ်လည်း လျော့နည်းသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Database တွင် role များ ခွဲခြားသတ်မှတ်သည် — application role က လိုအပ်သည့် table များကိုသာ ဖတ်ခွင့်ရှိသည်။ Migration ကဲ့သို့ schema ပြောင်းလဲမှုများအတွက် သီးသန့် role သုံးသည်။ Code မှ မူလ password များ အစား Vault မှ ရရှိသော role-specific credentials များ သုံးသည်။

### ဥပမာ

```python
import psycopg

# Application connects with a read-only role, not as admin
conn = psycopg.connect(
    host="db.internal",
    dbname="chatbot",
    user="app_reader",  # role with only SELECT on documents table
    password=os.environ["DB_PASSWORD"],
)

try:
    # This read succeeds: the role has SELECT permission
    cur = conn.execute("SELECT id FROM documents LIMIT 1;")
    print("Read rows:", cur.fetchone() is not None)

    # This write fails: the role has no INSERT permission
    conn.execute("INSERT INTO documents (content) VALUES ('x');")
except psycopg.errors.InsufficientPrivilege:
    print("Write blocked: InsufficientPrivilege")

# Expected output:
# Read rows: True
# Write blocked: InsufficientPrivilege
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
LLM application များတွင် prompt injection မှတဆင့် generated SQL ကို တိုက်ခိုက်သူက ထိန်းချုပ်နိုင်ခြေရှိသည်။ Read-only role သုံးပါက အဆိုပါတိုက်ခိုက်မှုက data ကို ဖျက်ဆီး၍ မရနိုင်တော့ပေ။

---

## ၃. Network Boundaries (ကွန်ရက် နယ်နိမိတ်များ)

### ဘာကို ဆိုလိုတာလဲ
Internal service များ (database, vector store) ကို ပြင်ပ internet မှ မမြင်စေဘဲ private network ထဲတွင် ထားခြင်းဖြစ်သည်။ Application တွင် external traffic ဝင်လမ်းတစ်ခုတည်း (gateway/load balancer) သာ ဖွင့်ထားသည်။

### ဘာကြောင့် လဲ
Database တစ်ခုကို internet ပေါ်တွင် ဖွင့်ထားပါက အလိုအလျောက် စကanning လုပ်သော တိုက်ခိုက်သူများ ရောက်လာနိုင်သည်။ Private network ထဲတွင် ရှိပါက အတွင်းရှိ service များနှင့်သာ ဆက်သွယ်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Kubernetes တွင် NetworkPolicy ဖြင့် pod များအကြား traffic ကို ထိန်းချုပ်သည်။ Database pod က app pod မှလာသော connection များကိုသာ လက်ခံသည်။ Cloud တွင် security group / firewall rules များဖြင့် လည်း ဆက်သွယ်နိုင်သည်။

### ဥပမာ

```python
import socket

def check_service(host, port, name):
    # Try to open a TCP connection to a service
    try:
        with socket.create_connection((host, port), timeout=3):
            print(f"{name}: reachable at {host}:{port}")
    except (ConnectionRefusedError, socket.timeout, OSError):
        print(f"{name}: NOT reachable from this network")

# Internal service is reachable within the cluster network
check_service("postgres.chatbot.svc.cluster.local", 5432, "postgres")

# The same service is not exposed to the public internet
# (verified from an external host, e.g. by DNS/port not resolving)
check_service("chatbot.example.com", 5432, "postgres-via-public")

# Expected output (run inside the cluster):
# postgres: reachable at postgres.chatbot.svc.cluster.local:5432
# postgres-via-public: NOT reachable from this network
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI application များသည် model API, vector DB, cache စသည့် service များစွာနှင့် ဆက်သွယ်ရသည်။ တစ်ခုချင်းစီကို public မဖွင့်ဘဲ internal ထဲ ကျဉ်းစေခြင်းက တိုက်ခိုက်ခံရနိုင်သည့်အရာ အရေအတွက် (attack surface) ကို လျှော့ပေးသည်။

---

## ၄. Dependency & Image Scanning

### ဘာကို ဆိုလိုတာလဲ
Python package များနှင့် Docker image များတွင် လုံခြုံမှုအားနည်းသော (vulnerable) version များ ပါဝင်နေမှုကို အလိုအလျောက် စစ်ဆေးခြင်းဖြစ်သည်။ `pip-audit`, `safety`, `trivy`, `grype` ကဲ့သို့သော tool များကို အသုံးပြုသည်။

### ဘာကြောင့် လဲ
Dependency တစ်ခုတွင် CVE (Common Vulnerabilities and Exposures) ပွင့်နေပါက သင်၏ application တစ်ခုလုံး အန္တရယ်ရှိလာသည်။ CI pipeline တွင် အလိုအလျောက် စစ်ဆေးခြင်းဖြင့် တိုက်ခိုက်ခံရမီ ကြိုတင်ဖာဆီးနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Dependency စာရင်းကို `pip freeze` ဖြင့် ထုတ်ယူပြီး advisory database များနှင့် နှိုင်းယှဉ်စစ်ဆေးသည်။ Docker image အတွက် `trivy image <name>` ဖြင့် OS package များနှင့် Python package များကို တွဲဖက်စစ်ဆေးသည်။ Vulnerability တွေ့ပါက version တိုးမြှင့်၍ ပြင်ဆင်သည်။

### ဥပမာ

```python
import subprocess
import json

# Run pip-audit and capture its JSON report
result = subprocess.run(
    ["pip-audit", "-r", "requirements.txt", "--format", "json"],
    capture_output=True, text=True,
)

if result.returncode in (0, 1):
    findings = json.loads(result.stdout)
    print(f"Vulnerable dependencies found: {len(findings)}")
    for item in findings[:3]:
        pkg = item["name"]
        vuln = item["vulns"][0]["id"] if item["vulns"] else "n/a"
        print(f"  - {pkg}: {vuln}")
else:
    print("Scan failed:", result.stderr[:200])

# Expected output (depends on your requirements.txt):
# Vulnerable dependencies found: 1
#   - requests: PYSEC-2022-XXXXX
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI/LLM ecosystem တွင် `langchain`, `transformers` ကဲ့သို့ dependency များ များပြားပြီး မကြာခဏ ပြောင်းလဲလေ့ရှိသည်။ Supply chain တိုက်ခိုက်မှု ( dependency တစ်ခုကို hijack လုပ်ခြင်း) သည် တိုးတက်လာသော အန္တရယ်တစ်ခုဖြစ်သဖြင့် ပုံမှန်စစ်ဆေးခြင်းက မဖြစ်မနေ လိုအပ်သည်။

---

## ၅. Audit Logging

### ဘာကို ဆိုလိုတာလဲ
System အတွင်း ဖြစ်ပျက်သမျှ အရေးကြီး ဖြစ်ရပ်များ — ဘယ် user က ဘယ် action ကို ဘယ်အချိန်မှာ လုပ်ခဲ့လဲ — ကို tamper-resistant ဖြစ်စေရန် မှတ်တမ်းတင်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Incident (အန္တရယ်ဖြစ်ရပ်) ဖြစ်ပါက ဘာဖြစ်ခဲ့လဲ၊ ဘယ် data ထိခိုက်ခဲ့လဲကို ပြန်လည်စုံစမ်းရန် လိုအပ်သည်။ Compliance (ဥပဒေစည်းကမ်း) များအတွက် လည်း မှတ်တမ်းများ လိုအပ်သည်။ မှတ်တမ်းမရှိပါက ဖြစ်ပျက်မှုကို ရှင်းပြရန် မဖြစ်နိုင်ပေ။

### ဘယ်လို အလုပ်လုပ်လဲ
Structured logging (JSON format) ဖြင့် user ID, action, timestamp, request ID များ မှတ်တမ်းတင်သည်။ PII (ကိုယ်ရေးအချက်အလက်) များနှင့် user prompt အပြည့်အစုံကို မှတ်တမ်းထဲ ထည့်သွင်းခြင်း မပြုရ — IDs နှင့် metadata သာ ထည့်သင့်သည်။

### ဥပမာ

```python
import logging
import json
import time

logger = logging.getLogger("audit")
logger.setLevel(logging.INFO)

class JSONFormatter(logging.Formatter):
    def format(self, record):
        # Emit each log line as a single JSON object
        payload = {
            "ts": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
            "level": record.levelname,
            "msg": record.getMessage(),
        }
        return json.dumps(payload)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

def answer_question(user_id: str, question: str):
    # Log metadata only: never log the raw question text
    logger.info("llm_query", extra={
        "custom": {"user_id": user_id, "action": "llm_query", "model": "gpt-4o-mini"}
    })
    return "answer generated"

result = answer_question(user_id="u_12345", question="private question text")

# Expected output:
# {"ts": "2025-01-15T09:30:00Z", "level": "INFO", "msg": "llm_query", "custom": {"user_id": "u_12345", "action": "llm_query", "model": "gpt-4o-mini"}}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
OWASP LLM Top 10 ၏ LLM01 (Prompt Injection) တိုက်ခိုက်မှုကို ရှာဖွေရန် audit log များ အရေးကြီးသည်။ User prompt များကို မှတ်တမ်းတင်ပါက user privacy ကို ထိခိုက်နိုင်သောကြောင့် metadata သာ တင်သည့် ချိန်ညှိမှု မဖြစ်မနေ လိုအပ်သည်။

---

## ၆. Key Rotation (သော့များ လည်ပတ်ပြောင်းလဲခြင်း)

### ဘာကို ဆိုလိုတာလဲ
API keys နှင့် passwords များကို သတ်မှတ်ကာလအတိုင်း ပုံမှန်ပြောင်းလဲပေးခြင်းဖြစ်သည်။ Vault ကဲ့သို့သော system များက dynamic credentials များ အလိုအလျောက်ထုတ်ပေးပြီး TTL ကုန်ဆုံးလျှင် လည်ပတ်ပြောင်း
