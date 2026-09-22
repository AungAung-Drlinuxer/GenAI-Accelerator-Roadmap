# လေ့ကျင့်ခန်းများ — Container Deploy & Reverse Proxy with TLS

## လေ့ကျင့်ခန်း ၁ — Version info response ရေးခြင်း

Application ၏ version နှင့် environment ကို JSON format ဖြင့် ပြန်ပေးမည့် Python function `version_info()` ကို ရေးပါ။ `APP_VERSION` နှင့် `APP_ENV` environment variables များကို ဖတ်ယူပြီး default တန်ဖိုးများ (`"0.1.0"`၊ `"development"`) အသုံးပြုပါ။

**Hints:** `os.getenv(name, default)` function က default တန်ဖိုးဖြင့် ဖတ်ယူခြင်းကို ထောက်ပံ့သည်။

**Expected behavior:** Function က `{"version": ..., "env": ...}` ပါဝင်သော dict ကို ပြန်ပြီး environment variables များ မရှိပါက default တန်ဖိုးများ အသုံးပြုသည်။

## လေ့ကျင့်ခန်း ၂ — Config loader ခွဲခြားခြင်း

`load_config()` function ကို ရေးပါ။ ၎င်းသည် `DATABASE_URL`၊ `API_KEY` နှင့် `MODEL_PATH` variables များကို ဖတ်ယူပြီး missing key များအတွက် error message list တစ်ခုကို စုဆောင်းပေးရမည်။ အားလုံး ရှိပါက errors list က ဗလာ ဖြစ်ရမည်။

**Hints:** Errors များကို return ပြုလုပ်မည့် dict ထဲတွင် `"errors"` key အဖြစ် ထည့်ပါ။

**Expected behavior:** `API_KEY` မသတ်မှတ်ထားပါက errors list တွင် `"API_KEY is missing"` ပါဝင်သည်။

## လေ့ကျင့်ခန်း ၃ — Health status စစ်ဆေးခြင်း

`check_health(components)` function ကို ရေးပါ။ `components` က dict ဖြစ်ပြီး ဥပမာ `{"database": True, "model": False}` ကဲ့သို့ ဖြစ်သည်။ Function က status နှင့် failing components list ပါသော dict ပြန်ရမည် — အားလုံး OK ဖြစ်လျှင် `"ok"`၊ တစ်ခုမှ မအောင်မြင်လျှင် `"degraded"` ပြန်ရမည်။

**Hints:** `any(value is False for value in components.values())` က failing check အတွက် အသုံးဝင်သည်။

**Expected behavior:** `{"database": True, "model": False}` ထည့်လျှင် `{"status": "degraded", "failing": ["model"]}` ရရှိသည်။

## လေ့ကျင့်ခန်း ၄ — Docker HEALTHCHECK command ဖန်တီးခြင်း

`docker inspect` မှ ရရှိလာမည့် health log lines (ဥပမာ `"healthy"`၊ `"unhealthy"`၊ `"starting"`) များကို စစ်ဆောင်း၍ `summarize_health(log)` function ရေးပါ။ အရေအတွက်အလိုက် status တစ်ခုစီ count ပြုလုပ်ပြီး dict ပြန်ရမည်။

**Hints:** `dict.get(key, 0) + 1` pattern က counting အတွက် ရိုးရိုးရှင်းရှင်း ဖြေရှင်းနည်းဖြစ်သည်။

**Expected behavior:** `["healthy", "healthy", "unhealthy"]` input က `{"healthy": 2, "unhealthy": 1}` ပြန်သည်။

## လေ့ကျင့်ခန်း ၅ — Reverse proxy route mapping

`parse_caddyfile(lines)` function ကို ရေးပါ။ Caddyfile ပုံစံ `"api.example.com {"` နှင့် `"    reverse_proxy app:8000"` ကဲ့သို့သော lines များကို လက်ခံပြီး `{"host": ..., "backend": ...}` dict list တစ်ခု ပြန်ရမည်။ Comment lines (`#` စသည်များ) ကို ချန်လှန်ရမည်။

**Hints:** Line တစ်ခုစီကို `.strip()` ပြုလုပ်ပြီး `startswith("{")` ဟုတ်မဟုတ် စစ်ပါ။

**Expected behavior:** Input နှစ် block ပါဝင်လျှင် output list တွင် route dicts နှစ်ခု ပါဝင်သည်။

## လေ့ကျင့်ခန်း ၆ — Zero-downtime switch simulation

`BlueGreen` class ကို ရေးပါ။ `switch()` method ရှိရမည် — ၎င်းက လက်ရှိ inactive slot အသစ်ကို healthy ဟု သတ်မှတ်ပြီး active traffic ကို ထို slot သို့ ရွှေ့ပြီး အရင် slot ကို shutdown ပြုလုပ်ရမည်။ `active` attribute က လက်ရှိ slot name ကို ဖော်ပြရမည်။

**Hints:** Blue slot မှစတင်ပြီး `self.active == "blue"` ဖြစ်ပါက green သို့ ရွှေ့ပါ၊ ပြီးလျှင် ဆန့်ကျင်ဘက် ဖြစ်သည်။

**Expected behavior:** `switch()` နှစ်ကြိမ်ခေါ်လျှင် active က `"green"` မှ `"blue"` သို့ ပြန်ပြောင်းသည်၊ ဝန်ဆောင်မှု ရပ်တန့်မှု မရှိပါ။
