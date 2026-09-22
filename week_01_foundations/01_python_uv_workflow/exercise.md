# လေ့ကျင့်ခန်းများ — Python & uv Project Workflow

ဒီလေ့ကျင့်ခန်းများကို တစ်ခုပြီးတစ်ခု အစဉ်လိုက် လုပ်ဆောင်ပါ။ Exercise ၁ မှ ၃ အထိ command line နဲ့ လုပ်ရမှာဖြစ်ပြီး၊ ၄ မှ ၆ အထိ Python script ရေးရမှာဖြစ်ပါတယ်။

## လေ့ကျင့်ခန်း ၁ — uv project အသစ် တည်ဆောက်ခြင်း

`uv init my-first-ai-app` command ကို အသုံးပြု၍ project အသစ်တစ်ခု တည်ဆောက်ပါ။ ဖန်တီးလိုက်သော ဖိုင်များကို စာရင်းကြည့်ပြီး၊ `pyproject.toml` ဖိုင်ထဲမှာ ဘယ် field တွေ ပါဝင်လဲ ဆိုတာကို မှတ်သားပါ။

**Hints:** `ls -la my-first-ai-app` နဲ့ ဖိုင်တွေကို ကြည့်နိုင်ပါတယ်။ `cat pyproject.toml` နဲ့ အကြောင်းအရာကို ဖတ်နိုင်ပါတယ်။

**Expected behavior:** `pyproject.toml`၊ `main.py` (သို့မဟုတ် `hello.py`) နှင့် `.python-version` ဖိုင်များ ပေါ်ပါသည်။

## လေ့ကျင့်ခန်း ၂ — Dependency ထည့်သွင်းခြင်း

`uv add httpx` command ကို run လုပ်ပြီး၊ `pyproject.toml` နှင့် `uv.lock` ဖိုင်နှစ်ခုလုံး ပြောင်းလဲသွားပုံကို လေ့လာပါ။ ဘယ်ဖိုင်မှာ ဘာ ပေါင်းထည့်သွားလဲ ဆိုတာကို ဖော်ပြပါ။

**Hints:** `uv add` မ run ခင်နဲ့ run ပြီးခါမှာ `pyproject.toml` ကို `cat` လုပ်၍ နှိုင်းယှဉ်ပါ။ `uv.lock` ထဲတွင် httpx နှင့် ၎င်း၏ dependency များ ပါဝင်သည်ကို မြင်ရမည်။

**Expected behavior:** `pyproject.toml` ထဲ `dependencies` list တွင် `httpx` ပေါ်လာပြီး၊ `uv.lock` ထဲတွင် version အတိအကျ မှတ်တမ်း ဝင်သည်။

## လေ့ကျင့်ခန်း ၃ — uv run ဖြင့် script run ခြင်း

`main.py` (သို့မဟုတ် သင်၏ script) ထဲမှာ `import httpx` လုပ်၍ httpx version ကို print လုပ်စေပြီး `uv run main.py` ဖြင့် run ပါ။ activate လုပ်ခြင်း မရှိဘဲ run လို့ရမှုကို သတိပြုပါ။

**Hints:** `python main.py` ကို တိုက်ရိုက် run လုပ်၍ မတူညီမှုကို နှိုင်းယှဉ်ကြည့်ပါ — တချို့စက်တွင် httpx မရှောင်းဘဲ error ပေးနိုင်သည်။

**Expected behavior:** `uv run` က အလိုအလျောက် environment ကို ပြင်ဆင်ပေးပြီး script ကို အောင်မြင်စွာ run ပေးသည်။

## လေ့ကျင့်ခန်း ၄ — pyproject.toml ဖတ်ခြင်း (Python)

Python code ဖြင့် `pyproject.toml` ဖိုင်ကို `tomllib` module သုံး၍ ဖတ်ပြီး project name၊ version နှင့် dependencies list ကို print လုပ်ပါ။

**Hints:** `tomllib.load()` က binary mode ဖြင့် ဖွင့်ရမည် (`"rb"`)။ Python 3.11 နှင့်အထက်တွင် `tomllib` built-in ဖြစ်သည်။

**Expected behavior:** Project name၊ version နှင့် dependency string များ ထင်ရှားစွာ ပေါ်သည်။

## လေ့ကျင့်ခန်း ၅ — Installed version စစ်ဆေးခြင်း

`importlib.metadata` module ဖြင့် `httpx` package ၏ installed version ကို ရယူပြီး print လုပ်ပါ။ Package မရှိပါက လှမ်း၍ ရှင်းလင်းသော error message ထုတ်ပါ။

**Hints:** `importlib.metadata.version("httpx")` က version string ပြန်ပေးသည်။ `PackageNotFoundError` exception ကို catch လုပ်ပါ။

**Expected behavior:** ဒီ script ကို `uv run` ဖြင့် run လျှင် httpx version ပေါ်ပြီး၊ venv မပါသော Python ဖြင့် run လျှင် error message ပေါ်သည်။

## လေ့ကျင့်ခန်း ၆ — Environment တူညီမှု စစ်ဆေးခြင်း

Script တစ်ခုရေး၍ လက်ရှိ Python interpreter သည် project ၏ `.venv` အတွင်းတွင် ရှိမရှိ (string `"venv"` ကို `sys.executable` ထဲ ရှာခြင်းအားဖြင့်) စစ်ဆေးပြီး၊ မရှိပါက `uv run <script>` ကို အသုံးပြုရန် ညွှန်ကြားသော message ထုတ်ပါ။

**Hints:** `sys.executable` က Python interpreter ၏ full path ပေးသည်။ `.venv` path က အနည်းငယ် OS အလိုက် ကွဲပြားနိုင်သည်။

**Expected behavior:** `uv run` ဖြင့် run လျှင် "OK" message ပေါ်ပြီး၊ system Python ဖြင့် run လျှင် warning နှင့် ညွှန်ကြားချက် ပေါ်သည်။
