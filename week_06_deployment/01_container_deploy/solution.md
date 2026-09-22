# ဖြေရှင်းချက်များ — Container Deploy & Reverse Proxy with TLS

## လေ့ကျင့်ခန်း ၁ — Version info response ရေးခြင်း

```python
# solution_1_version_info.py
import os

def version_info():
    # Read version and environment from env vars with safe defaults.
    return {
        "version": os.getenv("APP_VERSION", "0.1.0"),
        "env": os.getenv("APP_ENV", "development"),
    }

if __name__ == "__main__":
    print(version_info())
    os.environ["APP_VERSION"] = "1.2.3"
    os.environ["APP_ENV"] = "production"
    print(version_info())
# Expected output:
# {'version': '0.1.0', 'env': 'development'}
# {'version': '1.2.3', 'env': 'production'}
```

**အဓိကအယူအဆ** — Environment variables ဖြင့် version info ကို code မှ ခွဲထုတ်ခြင်းက image တစ်ခုတည်းကို နေရာများစွာတွင် အသုံးပြုနိုင်စေသည်။

## လေ့ကျင့်ခန်း ၂ — Config loader ခွဲခြားခြင်း

```python
# solution_2_load_config.py
import os

REQUIRED_KEYS = ["DATABASE_URL", "API_KEY", "MODEL_PATH"]

def load_config():
    # Collect values for required keys, tracking any that are missing.
    config = {}
    errors = []
    for key in REQUIRED_KEYS:
        value = os.environ.get(key)
        if value is None:
            errors.append(f"{key} is missing")
        else:
            config[key] = value
    config["errors"] = errors
    return config

if __name__ == "__main__":
    os.environ["DATABASE_URL"] = "postgres://localhost/app"
    os.environ["MODEL_PATH"] = "/models/latest"
    print(load_config())
# Expected output:
# {'DATABASE_URL': 'postgres://localhost/app', 'MODEL_PATH': '/models/latest', 'errors': ['API_KEY is missing']}
```

**အဓိကအယူအဆ** — Startup ချိန်တွင် missing config များကို စောစွာ ဖမ်းဆုပ်ခြင်းက production တွင် နောက်ပိုင်း ပြဿနာများ ဖြစ်ပွားမှုကို လျှော့ချ်ပေးသည်။

## လေ့ကျင့်ခန်း ၃ — Health status စစ်ဆေးခြင်း

```python
# solution_3_check_health.py

def check_health(components):
    # Report overall status plus a list of failing component names.
    failing = [name for name, ok in components.items() if not ok]
    if failing:
        status = "degraded"
    else:
        status = "ok"
    return {"status": status, "failing": failing}

if __name__ == "__main__":
    print(check_health({"database": True, "model": False}))
    print(check_health({"database": True, "model": True}))
# Expected output:
# {'status': 'degraded', 'failing': ['model']}
# {'status': 'ok', 'failing': []}
```

**အဓိကအယူအဆ** — Health endpoint တစ်ခုက dependency တစ်ခုချင်းစီ၏ အခြေအနေကို အသေးစိတ် ဖော်ပြပေးရမည်ဖြစ်သည်။

## လေ့ကျင့်ခန်း ၄ — Docker HEALTHCHECK command ဖန်တီးခြင်း

```python
# solution_4_summarize_health.py

def summarize_health(log):
    # Count occurrences of each health status in the log lines.
    counts = {}
    for entry in log:
        counts[entry] = counts.get(entry, 0) + 1
    return counts

if __name__ == "__main__":
    log = ["starting", "healthy", "healthy", "unhealthy", "healthy"]
    print(summarize_health(log))
# Expected output:
# {'starting': 1, 'healthy': 3, 'unhealthy': 1}
```

**အဓိကအယူအဆ** — Health log history ကို ခြုံငုံသုံးသပ်ခြင်းဖြင့် container တစ်ခု တည်ငြိမ်စွာ လည်ပတ်နေမှု ရှိမရှိကို သိနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Reverse proxy route mapping

```python
# solution_5_parse_caddyfile.py

def parse_caddyfile(lines):
    # Parse a simplified Caddyfile into a list of host-to-backend routes.
    routes = []
    host = None
    for line in lines:
        text = line.strip()
        if not text or text.startswith("#"):
            continue  # skip blank lines and comments
        if text.endswith("{"):
            host = text[:-1].strip()
        elif text.startswith("reverse_proxy") and host is not None:
            backend = text.split(maxsplit=1)[1].strip()
            routes.append({"host": host, "backend": backend})
    return routes

if __name__ == "__main__":
    sample = [
        "# main site",
        "api.example.com {",
        "    reverse_proxy app:8000",
        "}",
        "admin.example.com {",
        "    reverse_proxy admin:9000",
        "}",
    ]
    for route in parse_caddyfile(sample):
        print(route)
# Expected output:
# {'host': 'api.example.com', 'backend': 'app:8000'}
# {'host': 'admin.example.com', 'backend': 'admin:9000'}
```

**အဓိကအယူအဆ** — Caddyfile ၏ host-to-backend mapping က reverse proxy စိတ်ကူးကို Python data structure ဖြင့် နားလည်လွယ်စေသည်။

## လေ့ကျင့်ခန်း ၆ — Zero-downtime switch simulation

```python
# solution_6_blue_green.py

class BlueGreen:
    def __init__(self):
        # Start with blue active and green idle.
        self.active = "blue"
        self.healthy = True

    def _other_slot(self):
        # Return the name of the slot that is not currently active.
        return "green" if self.active == "blue" else "blue"

    def switch(self):
        # Bring up the new slot, verify health, move traffic, stop the old slot.
        new_slot = self._other_slot()
        if not self.healthy:
            raise RuntimeError("new slot is not healthy; refusing to switch")
        old_slot = self.active
        self.active = new_slot  # traffic now goes to the new slot
        print(f"traffic moved {old_slot} -> {new_slot}; {old_slot} shutting down")
        return self.active

if __name__ == "__main__":
    bg = BlueGreen()
    print("active:", bg.active)
    print("after switch 1:", bg.switch())
    print("after switch 2:", bg.switch())
# Expected output:
# active: blue
# traffic moved blue -> green; blue shutting down
# after switch 1: green
# traffic moved green -> blue; green shutting down
# after switch 2: blue
```

**အဓိကအယူအဆ** — အသစ်ကို အရင်စတင်ပြီး healthy ဖြစ်မှ traffic ရွှေ့ကာ အရင်ကို ပိတ်ခြင်းက zero-downtime deployment ၏ အခြေခံ စည်းမျဉ်းဖြစ်သည်။
