## လေ့ကျင့်ခန်း ၁ — Git ထဲမှ Secrets ရှာဖွေခြင်းနှင့် Vault သို့ရွှေ့ခြင်း

```python
# Scan a text line for high-risk secret patterns before committing
import re

SECRET_PATTERNS = {
    "AWS Access Key": r"AKIA[0-9A-Z]{16}",
    "OpenAI-style API key": r"sk-[A-Za-z0-9]{20,}",
    "Generic password assignment": r"(?i)(password|passwd|secret)\s*=\s*['\"][^'\"]{6,}['\"]",
}

def find_secrets(text):
    # Return a list of (pattern_name, matched_string) tuples
    findings = []
    for name, pattern in SECRET_PATTERNS.items():
        match = re.search(pattern, text)
        if match:
            findings.append((name, match.group(0)))
    return findings

sample_line = "client = OpenAI(api_key='sk-abc123def456ghi789jklmnop')"
for name, value in find_secrets(sample_line):
    print(f"DANGER: {name} found -> {value[:12]}...")

# Correct approach: read the key from an environment variable at runtime
import os
api_key = os.environ.get("OPENAI_API_KEY")
print("Loaded from environment:", "OK" if api_key else "MISSING")
```

**အဓိကအယူအဆ** — API key များကို git repository ထဲတွင် လုံးဝမထည့်သင့်ဘဲ environment variable သို့မဟုတ် HashiCorp Vault ကဲ့သို့သော vault မှ runtime တွင်သာ ဖတ်ယူသင့်သည်။

## လေ့ကျင့်ခန်း ၂ — Least-Privilege Database Role ဖန်တီးခြင်း

```python
# Print SQL statements that create a read-only role for an app
# Run these statements once with a DBA account (e.g., psql)
SQL_STATEMENTS = [
    # A role that cannot write, only SELECT on specific tables
    "CREATE ROLE app_reader NOLOGIN;",
    "GRANT CONNECT ON DATABASE chatbot TO app_reader;",
    "GRANT USAGE ON SCHEMA public TO app_reader;",
    "GRANT SELECT ON public.conversations, public.feedback TO app_reader;",
    # A login role for the app, inheriting only the read role
    "CREATE ROLE app_service LOGIN PASSWORD :'app_password';",
    "GRANT app_reader TO app_service;",
    # Revoke dangerous defaults from PUBLIC
    "REVOKE ALL ON SCHEMA public FROM PUBLIC;",
]

for stmt in SQL_STATEMENTS:
    print(stmt + ";")

# Verification query the app can run, but writes must fail
VERIFY = "SELECT current_user, has_table_privilege('app_service', 'conversations', 'SELECT');"
print("\nVerify with:", VERIFY)
print("Then confirm an INSERT is rejected with 'permission denied'.")
```

**အဓိကအယူအဆ** — Application တစ်ခုသည် လိုအပ်သည့် table များကို ဖတ်ရုံသာ ခွင့်ပြုသော least-privilege role ဖြင့် အသုံးပြုသင့်ပြီး owner သို့မဟုတ် superuser account ကို မည်သည့်အခါမျှ တိုက်ရိုက်မသုံးသင့်ပါ။

## လေ့ကျင့်ခန်း ၃ — Network Boundary စစ်ဆေးမှုငယ် (Port Allow-List)

```python
# Simulate an egress firewall check for an AI inference service
ALLOWED_EGRESS = {
    ("app-server", 443),   # TLS to the LLM API
    ("app-server", 5432),  # internal database only
}
INTERNAL_SERVICES = {"app-server", "db-internal", "vector-db"}

def check_connection(src, dst, port):
    # Deny anything not in the explicit allow-list
    if (src, port) not in ALLOWED_EGRESS:
        return f"DENY {src} -> {dst}:{port}"
    if dst not in INTERNAL_SERVICES and port == 5432:
        return f"DENY {src} -> {dst}:{port} (DB must stay internal)"
    return f"ALLOW {src} -> {dst}:{port}"

print(check_connection("app-server", "db-internal", 5432))  # allowed
print(check_connection("app-server", "api.openai.com", 443))  # allowed
print(check_connection("app-server", "evil.example.com", 8080))  # denied
print(check_connection("app-server", "outside-host", 5432))  # denied
```

**အဓိကအယူအဆ** — Database နှင့် internal service များကို private network အတွင်း၌သာ ထားရှိပြီး outbound ချိတ်ဆက်မှုများကို 443/TLS ကဲ့သို့သော တိကျသော allow-list ဖြင့် ကန့်သတ်သင့်သည်။

## လေ့ကျင့်ခန်း ၄ — Dependency စစ်ဆေးခြင်း (pip-audit အသုံးပြုပုံ)

```python
# Generate a requirements file and show how to audit it
import subprocess
import sys

REQUIREMENTS = """fastapi==0.115.0
uvicorn==0.30.6
openai==1.40.0
pyjwt==2.9.0
"""

def write_requirements(path="requirements.txt"):
    with open(path, "w", encoding="utf-8") as f:
        f.write(REQUIREMENTS)
    print(f"Wrote {path}")

def audit_dependencies():
    # pip-audit checks installed packages against the OSV/advisory databases
    cmd = [sys.executable, "-m", "pip_audit", "-r", "requirements.txt"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True)
        print(result.stdout or result.stderr)
    except FileNotFoundError:
        print("Install first: pip install pip-audit")

write_requirements()
audit_dependencies()
```

**အဓိကအယူအဆ** — Third-party package များတွင် ထိုးဖောက်နိုင်သော vulnerability များ ရှိနိုင်သဖြင့် build မတင်မီ `pip-audit` သို့မဟုတ် `safety` ဖြင့် ပုံမှန်စစ်ဆေးပြီး lock လုပ်ထားသော version များကိုသာ အသုံးပြုသင့်သည်။

## လေ့ကျင့်ခန်း ၅ — Audit Log မှတ်တမ်းရေးခြင်း

```python
# Minimal structured audit logger for security-relevant events
import json
import logging
import time

logger = logging.getLogger("audit")

def audit_event(event_type, actor, detail):
    # Structured JSON lines are easy to search and forward to a SIEM
    record = {
        "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "event": event_type,
        "actor": actor,
        "detail": detail,
    }
    logger.warning(json.dumps(record, ensure_ascii=False))

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    audit_event("secret_read", "app-service", {"key": "OPENAI_API_KEY", "vault_path": "secret/data/ai"})
    audit_event("role_grant", "admin", {"role": "app_reader", "to": "app_service"})
    audit_event("login_failed", "unknown", {"ip": "203.0.113.7"})
```

**အဓိကအယူအဆ** — Secret ဖတ်ခြင်း၊ role ပြောင်းခြင်းနှင့် login ကျိုးစားမှုများကို structured audit log ဖြင့် မှတ်တမ်းတင်ထားခြင်းသည် ဖြစ်ပျက်မှုကို နောက်မှ စစ်ဆေးနိုင်ရန် မရှိမဖြစ်လိုအပ်သည်။

## လေ့ကျင့်ခန်း ၆ — Key Rotation အလိုအလျောက်လုပ်ဆောင်ခြင်း

```python
# Check whether a stored key is older than the rotation period
from datetime import datetime, timedelta, timezone

ROTATION_DAYS = 90

def needs_rotation(created_at_iso, now=None):
    # Compare key age against the policy window
    now = now or datetime.now(timezone.utc)
    created = datetime.fromisoformat(created_at_iso)
    age = now - created
    return age > timedelta(days=ROTATION_DAYS), age.days

key_metadata = {
    "openai-api-key": "2025-01-10T08:00:00+00:00",
    "db-app-password": "2025-05-20T08:00:00+00:00",
}

for name, created in key_metadata.items():
    rotate, age_days = needs_rotation(created)
    status = "ROTATE NOW" if rotate else "OK"
    print(f"{name}: age={age_days}d -> {status}")

# Dual-key pattern: keep old key valid briefly so deploys never break
print("Rotation procedure: create new key -> deploy apps with new key -> revoke old key")
```

**အဓိကအယူအဆ** — API key နှင့် password များကို သတ်မှတ်ထားသော ကာလအတွင်း အလိုအလျောက် လှည့်ပတ်ပြောင်းရွှေ့ပြီး လှည့်ပတ်စဉ်တွင် ဝန်ဆောင်မှု မရပ်နားစေရန် dual-key နည်းလမ်းကို အသုံးပြုသင့်သည်။

## လေ့ကျင့်ခန်း ၇ — Container Image စစ်ဆေးမှု Pipeline ပြင်ဆင်ခြင်း

```python
# Generate a CI step file that scans the Docker image before pushing
SCAN_STEP = """name: scan-image
on: push
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t ai-app:${{ github.sha }} .
      - name: Scan with Trivy
        run: |
          docker run --rm \\
            -v /var/run/docker.sock:/var/run/docker.sock \\
            aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL ai-app:${{ github.sha }}
"""

def write_workflow(path=".github/workflows/scan-image.yml"):
    import os
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "w", encoding="utf-8") as f:
        f.write(SCAN_STEP)
    print(f"Wrote {path} - HIGH/CRITICAL findings will fail the build")

write_workflow()
```

**အဓိကအယူအဆ** — Container image တွင် ပါဝင်သော OS package များကို Trivy သို့မဟုတ် Grype ဖြင့် စကင်ဖြင့် ရုပ်ရှင်ရှာပြီး HIGH နှင့် CRITICAL အဆင့် ပြဿနာများ တွေ့ပါက build ကို ရပ်တန့်စေသင့်သည်။
