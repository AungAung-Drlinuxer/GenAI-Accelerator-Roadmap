# Python & uv Project Workflow — ရှင်းလင်းချက်

## ၁။ pyproject.toml

### ဘာကို ဆိုလိုတာလဲ

`pyproject.toml` ဆိုသည်မှာ Python project တစ်ခု၏ metadata နှင့် dependency များကို သတ်မှတ်ထားသော configuration ဖိုင်ဖြစ်သည်။ project နာမည်၊ Python version requirement၊ နှင့် အသုံးပြုမည့် package များကို ဤဖိုင်တစ်ခုတည်းတွင် ရေးသားသည်။

### ဘာကြောင့် လဲ

အခြေခံကျသော Python project တိုင်းသည် ယနေ့ခေတ်တွင် `requirements.txt` ထက် `pyproject.toml` ကို ပိုမိုအသုံးပြုလာကြသည်။ အကြောင်းမှာ package metadata နှင့် dependency ကို ဖိုင်တစ်ခုတည်းတွင် စုစည်းထားနိုင်ပြီး၊ build tool အားလုံးက ဤ standard (PEP 621) ကို လိုက်နာသောကြောင့်ဖြစ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`uv init` command ကို run လိုက်လျှင် `pyproject.toml` ဖိုင်ကို အလိုအလျောက် ဖန်တီးပေးသည်။ `[project]` section တွင် နာမည်နှင့် version ရှိပြီး၊ `dependencies` list တွင် လိုအပ်သော package များကို ထည့်သွင်းရသည်။ `uv add <package>` command က ဒီ list ကို အလိုအလျောက် ပြင်ပေးသည်။

### ဥပမာ

```python
# read_config.py - demonstrates reading project configuration programmatically
import tomllib

# open pyproject.toml in binary mode (required by tomllib)
with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)

project = data["project"]
print("Project name:", project["name"])
print("Requires Python:", project.get("requires-python", "not specified"))

# Expected output:
# Project name: my-ai-app
# Requires Python: >=3.12
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

AI project တစ်ခုတွင် OpenAI SDK၊ Anthropic SDK ကဲ့သို့ package များ များစွာ သုံးရသည်။ ထို package များကို ဖိုင်တစ်ခုတည်းတွင် မှတ်တမ်းတိကျစွာ ထားရှိမှသာ တစ်ယောက်နှင့် တစ်ယောက် environment မတူညီသော ပြဿနာများ ရှောင်ရှင်းနိုင်သည်။

## ၂။ Lockfile (uv.lock) နှင့် Deterministic Environment

### ဘာကို ဆိုလိုတာလဲ

Lockfile ဆိုသည်မှာ project တွင် အသုံးပြုမည့် package တိုင်း၏ **အတိအကျ version နံပါတ်** (ဥပမာ `httpx==0.27.2`) ကို မှတ်တမ်းတင်ထားသော `uv.lock` ဖိုင်ဖြစ်သည်။

### ဘာကြောင့် လဲ

`pyproject.toml` တွင် `httpx>=0.27` ဟူ၍ ရေးပါက အချိန်ကြာသည်နှင့်အမျှ အသစ်ထွက် version များ အလိုအလျောက် ပြောင်းလဲသွားနိုင်သည်။ ထိုအခါ ယနေ့ run သော result နှင့် လမ်းသစ် run သော result များ ကွဲပြားနိုင်သည်။ Lockfile က ဤအရာကို တားဆီးပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`uv add` သို့မဟုတ် `uv lock` command ကို run လိုက်သောအခါ uv က dependency tree အပြည့်အစုံကို ဖြေရှင်းပြီး `uv.lock` ဖိုင်ကို ရေးသည်။ နောက်ပိုင်းတွင် `uv sync` command က ဒီ lockfile အတိုင်း environment ကို ပြန်လည် တည်ဆောက်ပေးသည်။ Team member အားလုံး ဤ lockfile ကို share လုပ်ထားပါက အားလုံး၏ environment သည် တစ်ပြားမကွာ တူညီနေမည်။

### ဥပမာ

```python
# check_lock.py - simulate verifying a pinned dependency version
import subprocess

# run 'uv export --format requirements-txt' to see exact pinned versions
result = subprocess.run(
    ["uv", "export", "--format", "requirements-txt"],
    capture_output=True,
    text=True,
)

# print only lines containing 'httpx' to keep output short
for line in result.stdout.splitlines():
    if "httpx" in line:
        print(line)

# Expected output:
# httpx==0.27.2     (exact version depends on your uv.lock)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

LLM library များသည် API ပြောင်းလဲမှုများ မကြာခဏ ရှိသည်။ Version တစ်ခုနှင့် တစ်ခု ကွာခြားလျှင် function signature ပြောင်းသွားပြီး ကုဒ်အလုပ်မလုပ်တော့နိုင်သည်။ Production တွင် ဒီကဲ့သို့ အာမခံချက်မရှိသော ပြဿနာများသည် အလွန်ဆိုးရွားသည်။

## ၃။ uv run နှင့် နေ့စဉ်အသုံး

### ဘာကို ဆိုလိုတာလဲ

`uv run <script.py>` ဆိုသည်မှာ project ၏ environment ကို အလိုအလျောက် စစ်ဆေးပြီး script ကို run ပေးသော command ဖြစ်သည်။

### ဘာကြောင့် လဲ

Traditional workflow တွင် `python -m venv .venv`၊ `source .venv/bin/activate`၊ `pip install -r requirements.txt` စသည့် အဆင့်များစွာ လိုအပ်သည်။ `uv run` က ဤအဆင့်များကို တစ်ခုတည်းသို့ ခြုံငုံပေးပြီး၊ environment မှန်ကန်ကြောင်း သေချာမှသာ script ကို run ပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

`uv run` ကို run လိုက်သောအခါ uv က `pyproject.toml` နှင့် `uv.lock` ကို ဖတ်ပြီး၊ `.venv` ထဲရှိ package များ လိုအပ်ချက်နှင့် ကိုက်ညီမှုရှိမရှိ စစ်ဆေးသည်။ မကိုက်ပါက အလိုအလျောက် `uv sync` ကို ဆောင်ရွက်ပေးသည်။ ထို့နောက် script ကို `.venv` အတွင်းရှိ Python interpreter ဖြင့် run သည်။

### ဥပမာ

```python
# check_env.py - show which Python and packages the script is running with
import sys
import importlib.metadata

print("Python version:", sys.version.split()[0])
print("Executable path:", sys.executable)

# show the exact installed version of the openai package, if present
try:
    version = importlib.metadata.version("openai")
    print("openai version:", version)
except importlib.metadata.PackageNotFoundError:
    print("openai package is not installed")

# Expected output:
# Python version: 3.12.x
# Executable path: <your-project>/.venv/bin/python
# openai version: 1.x.x
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

CI pipeline များ၊ Docker container များ၊ သို့မဟုတ် teammate တစ်ယောက်၏ စက်ထဲတွင် မည်သူကမျှ `activate` လုပ်ရန် မမှတ်မိနိုင်ပါ။ `uv run` ကိုသာ အသုံးပြုပါက ထို အလေးအနက်များ အားလုံး ပျောက်ကွယ်သွားပြီး တူညီသော environment အာမခံသည်။

## ၄။ Reproducibility နှင့် LLM Application

### ဘာကို ဆိုလိုတာလဲ

Reproducibility (ပြန်လည်ထုတ်ပေးနိုင်မှု) ဆိုသည်မှာ တစ်နေရာတွင် အလုပ်လုပ်သော ကုဒ်သည် အခြားနေရာ၊ အခြားအချိန်တွင်လည်း အတူတူပင် အလုပ်လုပ်ရမည်ဟူသော သဘောတရားဖြစ်သည်။

### ဘာကြောင့် လဲ

LLM သို့ request ပို့သော ကုဒ်တစ်ခုသည် အလွန်အရေးပါသော dependency များစွာပေါ်တွင် မှီခိုသည် — HTTP client library၊ SDK၊ encoding library များ စသည်ဖြင့်။ ဤ library များ၏ version များ ကွဲပြားပါက model သို့ပို့လိုက်သော request ၏ ပုံစံပင် ကွဲပြားနိုင်ပြီး၊ ရလဒ်များလည်း ကွဲပြားနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Dependency version များကို lockfile တွင် တိကျစွာ သတ်မှတ်ခြင်းအားဖြင့် ကုဒ်၊ environment၊ နှင့် run လုပ်သည့်အချိန် သုံးခုလုံးကို ချိတ်ဆက်ပေးသည်။ Git တွင် `pyproject.toml` နှင့် `uv.lock` နှစ်ခုလုံးကို commit လုပ်ထားရန် အရေးကြီးသည်။

### ဥပမာ

```python
# repro_check.py - verify that the active environment matches the lockfile
import subprocess
import sys

# check that the current interpreter is inside the project venv
if ".venv" not in sys.executable:
    print("WARNING: not running inside the project venv!")
    print("Use: uv run repro_check.py")
else:
    print("OK: running inside the project venv")

# verify the environment matches uv.lock exactly
result = subprocess.run(
    ["uv", "sync", "--check"],
    capture_output=True,
    text=True,
)
print("Lockfile check:", "PASSED" if result.returncode == 0 else "FAILED")

# Expected output:
# OK: running inside the project venv
# Lockfile check: PASSED
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Bug report တစ်ခုကို စုံစမ်းရာတွင် "ဒီ version နဲ့ ပြန် run ကြည့်ပါ" ဟု တိကျစွာ ပြောနိုင်ရန်အတွက် reproducibility သည် မရှိမဖြစ် လိုအပ်သည်။ Production LLM application များတွင် version drift ကြောင့် ဖြစ်သော အမှားများသည် ရှာဖွေရန် အလွန်ခက်ခဲပြီး၊ lockfile က ၎င်းကို အစအဦးတွင်ပင် တားဆီးပေးသည်။

## အနှစ်ချုပ်

- `pyproject.toml` က project metadata နှင့် dependency requirement များကို စုစည်းသတ်မှတ်ပေးသည်။
- `uv.lock` က package တိုင်း၏ အတိအကျ version ကို မှတ်တမ်းတင်ပြီး deterministic environment ကို အာမခံပေးသည်။
- `uv run` က environment စစ်ဆေးမှုနှင့် script run ခြင်းကို တစ်ခုတည်းသော command ဖြင့် ပြုလုပ်ပေးသည်။
- LLM application များတွင် version မတူညီမှုက request format ပြောင်းလိုက်ပြီး ရလဒ်များပါ ပြောင်းသွားနိုင်သောကြောင့် reproducibility သည် အလွန်အရေးကြီးသည်။
- `pyproject.toml` နှင့် `uv.lock` နှစ်ခုလုံးကို Git တွင် commit လုပ်ထားရမည်။
