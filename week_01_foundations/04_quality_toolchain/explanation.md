# Week 1 — Quality Toolchain: ruff, pytest, secrets hygiene

AI engineering project တစ်ခုကို စတင်ချိန်မှာ ကုဒ်အရည်အသွေး (code quality) နဲ့ လုံခြုံမှု (security) အခြေခံကို အရင်ချမှတ်သင့်ပါတယ်။ ဒီ module မှာ ruff နဲ့ lint/format လုပ်နည်း၊ pytest နဲ့ test ရေးနည်း၊ API keys တွေကို env var မှာ သိမ်းနည်းနဲ့ pre-commit gate တပ်နည်းတွေကို လေ့လာပါမယ်။

---

## Linting & Formatting with ruff

### ဘာကို ဆိုလိုတာလဲ
Linting ဆိုတာ ကုဒ်ထဲမှာ ရှိနေတဲ့ အမှားအယွင်းတွေ၊ စံမမှန်တဲ့ style တွေကို အလိုအလျောက် ရှာဖွေပြပေးတဲ့ နည်းပညာပါ။ Formatting ကတော့ ကုဒ်ရဲ့ ပုံစံ (indentation, quote, line length) ကို အလိုအလျောက် ညီညွတ်အောင် ပြင်ပေးတာပါ။ ruff က Rust နဲ့ ရေးထားတဲ့ Python linter/formatter ဖြစ်ပြီး အလွန်မြန်ပါတယ်။

### ဘာကြောင့် လဲ
Python ကုဒ်တွေက flexible ဖြစ်တာကြောင့် အလုပ်လုပ်ပေမယ့် ဖတ်ရခက်တဲ့၊ အမှားဖြစ်စေနိုင်တဲ့ ပုံစံတွေ ရေးလို့ရပါတယ်။ ဥပမာ — import လုပ်ပြီးသား module တွေကို မသုံးဘဲ ထားခဲ့ခြင်း၊ မသတ်မှတ်ထားတဲ့ variable ကို ခေါ်သုံးခြင်း (F821) တို့က bug ဖြစ်စေနိုင်ပါတယ်။ Linter က ဒီလိုပြဿနာတွေကို run ခါစကနဲ့ ဖမ်းပေးလို့ အချိန်ကုန်သက်သere သက်သာပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
ruff က source file တွေကို parse လုပ်ပြီး rule စုံတွေနဲ့ စစ်ဆေးပါတယ်။ `pip install ruff` လုပ်ပြီး `ruff check .` နဲ့ lint စစ်နိုင်ပြီး `ruff format .` နဲ့ အလိုအလျောက် format လုပ်နိုင်ပါတယ်။ Rule တွေကို `pyproject.toml` မှာ သတ်မှတ်နိုင်ပါတယ်။

### ဥပမာ
```python
# lint_example.py -- a file with several lint issues
import os          # unused import (F401)
import json        # unused import (F401)

def add_numbers(a, b):
    return a + b   # fine, but style rules may flag spacing elsewhere

result = add_numbers(2, 3)
print(reslt)       # typo: undefined name (F821)

# Run: ruff check lint_example.py
# Expected output:
# lint_example.py:2:8: F401 [*] `os` imported but unused
# lint_example.py:3:8: F401 [*] `json` imported but unused
# lint_example.py:9:6: F821 Undefined name `reslt`
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI project တွေမှာ prototype ရေးနေရင်းနဲ့ code တွေ အမြန်ကြီးလာပါတယ်။ ruff က typos, unused imports, unreachable code တွေကို အလိုအလျောက် ဖမ်းပေးတာကြောင့် review ချိန်၊ debug ချိန်တွေ သက်သာစေပြီး အဖွဲ့တွေကို code style တစ်ခုတည်း လိုက်နာစေပါတယ်။ `ruff format` က formatting အငြင်းပွားမှုတွေကိုပါ ပျောက်သွားစေပါတယ်။

---

## pytest Structure & Fixtures

### ဘာကို ဆိုလိုတာလဲ
pytest က Python အတွက် test framework တစ်ခုပါ။ Test file တွေကို `test_` နဲ့ စတဲ့ ဖိုင်နာမည်တွေမှာ ရေးပြီး `assert` statement နဲ့ စစ်ဆေးပါတယ်။ Fixture ကတော့ test တွေ မရေးခင် ပြင်ဆင်ပေးရမဲ့ data/object တွေကို နေရာတူမှ ပြန်သုံးနိုင်အောင် ပေးတဲ့ pytest feature ပါ။

### ဘာကြောင့် လဲ
Prompt template တစ်ခု၊ data parsing function တစ်ခု ပြောင်းလိုက်တဲ့အခါ အခြားနေရာတွေ ပျက်သွားမလား ဆိုတာ သိချင်ပါတယ်။ Test တွေ ရှိရင် `pytest` command တစ်ခုတည်းနဲ့ စစ်လို့ရပါတယ်။ Fixture က ပြင်ဆင်ချက် code တွေကို ထပ်ရေးနေရမှုကနေ ကင်းစေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
pytest က current folder ထဲမှာ `test_*.py` ဟု အမည်တပ်ထားတဲ့ ဖိုင်တွေကို ရှာပြီး function အမည် `test_` နဲ့ စတဲ့ function တွေကို run ပါတယ်။ `@pytest.fixture` decorator တပ်ထားတဲ့ function က test function ရဲ့ parameter နာမည်နဲ့ ကိုက်ညီရင် အလိုအလျောက် value ထိုးပေးပါတယ်။

### ဥပမာ
```python
# test_cleaner.py -- structure: file starts with test_, functions start with test_
import pytest

def clean_text(text: str) -> str:
    # strip whitespace and lowercase the input
    return text.strip().lower()

@pytest.fixture
def sample_prompt() -> str:
    # reusable prepared data for every test that requests this fixture
    return "  Hello AI Engineering  "

def test_clean_text_strips_and_lowercases(sample_prompt):
    assert clean_text(sample_prompt) == "hello ai engineering"

def test_clean_text_empty_string():
    assert clean_text("   ") == ""

# Run: pytest -q
# Expected output:
# 2 passed
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI application တွေမှာ prompt ပြောင်း၊ model ပြောင်းတိုင်း output ပြောင်းသွားတတ်ပါတယ်။ Preprocessing logic တွေ၊ response parsing တွေအတွက် test ရှိထားရင် ပြောင်းလဲမှုတွေက ဘယ်နေရာအထိ သက်ရောက်မလဲ ဆိုတာ မြန်မြန်သိနိုင်ပါတယ်။ Fixture တွေက test data တူညီမှု သေချာစေလို့ test တွေ ယုံကြည်စိတ်ချရပါတယ်။

---

## Secrets Hygiene — env vars & .env out of git

### ဘာကို ဆိုလိုတာလဲ
Secrets ဆိုတာ API keys, tokens, passwords တွေ ဖြစ်ပါတယ်။ ဒီတွေကို ကုဒ်ထဲ တိုက်ရိုက် မရေးဘဲ environment variable တွေထဲ သိမ်းပြီး Python ကနေ `os.environ` နဲ့ ဖတ်သင့်ပါတယ်။ `.env` ဖိုင်ထဲ သိမ်းတဲ့အခါ ဒို့ကို git ထဲ တင်တာကနေ ကာကွယ်ဖို့ `.gitignore` မှာ ထည့်ရပါတယ်။

### ဘာကြောင့် လဲ
OpenAI API key တစ်ခု ကုဒ်ထဲ ရေးသွင်းပြီး GitHub public repo တင်လိုက်ရင် မိနစ်အနည်းငယ်အတွင်း bots တွေ ဖမ်းပြီး အသုံးမှာ ငွေကုန်စေနိုင်ပါတယ်။ Secret တစ်ခြမ်းကို git history ထဲ ဝင်သွားရင် ဖျက်လို့ရှင်းရခက်ပြီး key အသစ် ထုတ်ပြောင်းရပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Terminal မှာ `export OPENAI_API_KEY="..."` လိုမျိုး သတ်မှတ်ခြင်း (Linux/macOS) ဒါမှမဟုတ် `.env` ဖိုင်မှာ `KEY=value` ပုံစံနဲ့ ရေးပြီး `python-dotenv` package နဲ့ load လုပ်တာ ဖြစ်ပါတယ်။ Project root မှာ `.env.example` (value အစား placeholder) ကို commit လုပ်ပြီး `.env` ကို `.gitignore` ထည့်ပါတယ်။

### ဥပမာ
```python
# config.py -- read a secret from the environment, never hardcode it
import os

def get_api_key() -> str:
    # read from environment variable; raise early if missing
    key = os.environ.get("OPENAI_API_KEY", "")
    if not key:
        raise RuntimeError("OPENAI_API_KEY is not set. Add it to .env or export it.")
    return key

# .env file (NOT committed to git):
# OPENAI_API_KEY=sk-your-key-here

# .gitignore should contain:
# .env

# Run: python -c "import config; print(config.get_api_key()[:3] + '...')"
# Expected output:
# sk-...
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI engineering မှာ external API တွေ သုံးရမှု များပါတယ်။ အဲဒီ API တွေရဲ့ keys တွေက ငွေနဲ့ တိုက်ရိုက်ဆက်နွှယ်ပါတယ်။ Secret hygiene က ဖောက်သည်ရဲ့ data တွေပါ ကာကွယ်ပေးတဲ့ အခန်းကနေပါ ပါဝင်ပါတယ်။ အဖွဲ့နဲ့ အလုပ်လုပ်တဲ့အခါ `.env.example` ရှိထားရင် အချင်းချင်း setup လုပ်ရလွယ်ပါတယ်။

---

## Pre-commit Style Gates

### ဘာကို ဆိုလိုတာလဲ
Pre-commit hook က git commit လုပ်ခါစမှာ အလိုအလျောက် check တွေ run ပြီး မပြည့်စုံရင် commit ကို ရပ်တန့်စေတဲ့ gate ပါ။ `pre-commit` tool က `.pre-commit-config.yaml` ဖိုင်ကနေ hook တွေကို စီမံပါတယ်။

### ဘာကြောင့် လဲ
သူများတွေ review မလုပ်ခင် ကိုယ့်ကုဒ်က စံနဲ့ ကိုက်နေအောင် အလိုအလျောက် စစ်ပေးလို့ ဖြစ်ပါတယ်။ ဒီလို gate ရှိရင် lint မှား၊ secret ထဲ ဝင်နေတဲ့ ဖိုင် တွေကို repository ထဲ မရောက်ခင် ဖမ်းနိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
`.pre-commit-config.yaml` မှာ hook list သတ်မှတ်ပြီး `pre-commit install` တစ်ခါ run ထားရင် commit တိုင်းမှာ အလိုအလျောက် run ပါတယ်။ ruff hook နဲ့ `detect-secrets` သို့မဟုတ် `gitleaks` တို့ကို ထည့်လေ့ရှိပါတယ်။

### ဥပမာ
```python
# .pre-commit-config.yaml content (YAML, shown here for reference)
# repos:
#   - repo: https://github.com/astral-sh/ruff-pre-commit
#     rev: v0.4.4
#     hooks:
#       - id: ruff
#       - id: ruff-format
#   - repo: https://github.com/gitleaks/gitleaks
#     rev: v8.18.0
#     hooks:
#       - id: gitleaks
#
# Setup:
#   pip install pre-commit
#   pre-commit install
#
# Expected output:
# pre-commit installed at .git/hooks/pre-commit
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Quality gate တွေက လူအမှားကို machine နဲ့ ဖမ်းပေးလို့ ဖြစ်ပါတယ်။ Developer တွေ အားလုံး အတူတူ pattern နဲ့ ရေးအောင်၊ secret တွေ repository ထဲ မရောက်အောင် အလိုအလျောက် သေချာစေပါတယ်။ အထူးသဖြင့် CI pipeline ထိ မရောက်ခင် ပြဿနာ ဖမ်းနိုင်တာက အချိန်ကုန် သက်သာစေပါတယ်။

---

## အနှစ်ချုပ်
- **ruff** က Python code တွေကို မြန်မြန် lint နဲ့ format လုပ်ပေးပြီး typos, unused imports တွေကို အလိုအလျောက် ဖမ်းပေးပါတယ်။
- **pytest** မှာ test file တွေက `test_` နဲ့ စရပြီး `assert` နဲ့ စစ်ပါတယ်၊ fixture တွေက တူညီတဲ့ test data ကို နေရာတူ သုံးစေပါတယ်။
- API keys တွေကို **environment variable ဒါမှမဟုတ် `.env`** ထဲ သိမ်းပြီး `.gitignore` နဲ့ git က ကာကွယ်ပါ။
- `.env.example` ကို commit လုပ်ပြီး `.env` ကို commit မလုပ်ဘဲ အဖွဲ့ setup လွယ်အောင် လုပ်ပါ။
- **pre-commit** hooks က commit ခါစမှာ ruff နဲ့ secret detection တွေ အလိုအလျောက် run ပေးပြီး ပြဿနာတွေ repo ထဲ မရောက်ခင် ဖမ်းပါတယ်။
- ဒီ toolchain သုံးခုက Week 1 အတွက် ယုံကြည်စိတ်ချရတဲ့ AI engineering အခြေခံ ဖြစ်ပါတယ်။
