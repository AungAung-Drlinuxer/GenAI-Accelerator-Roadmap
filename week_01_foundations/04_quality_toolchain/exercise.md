## လေ့ကျင့်ခန်း ၁ — ruff ဖြင့် formatting စတင်အသုံးပြုခြင်း

ပေးထားသော `greet.py` ဖိုင်ကို ဖန်တီးပါ။

```python
def greet( name ):
    message=f"Hello, {name}!"
    return message
print(greet( "AI engineer" ))
```

terminal တွင် `ruff format greet.py` ကို လည်ပတ်ပြီး ဖိုင်ကို ဖွင့်ကြည့်ပါ။ ရလဒ်နှင့် မူလကို နှိုင်းယှဉ်ပါ။ ထို့နောက် `ruff check greet.py` ဖြင့် lint issues များ ရှိ၊ မရှိ စစ်ကြည့်ပါ။

**Hints:** `pip install ruff` ဖြင့် အရင် install လုပ်ပါ။ Formatting သည် whitespace၊ quotes၊ line breaks များကိုသာ ပြင်ဆင်ပေးပြီး logic ကို မပြောင်းလဲပါ။

**Expected behavior:** `ruff format` က `message = f"Hello, {name}!"` ကဲ့သို့ spaces များ ပြောင်းလဲပေးပြီး `ruff check` က ဤရိုးရိုးဖိုင်အတွက် violations မတွေ့ပါက exit code 0 ဖြင့် "All checks passed!" ပြသပါမည်။

## လေ့ကျင့်ခန်း ၂ — ruff lint rules ကို ရွေးချယ်ခြင်း

`pyproject.toml` ဖိုင်တွင် ruff configuration ထည့်ပါ။

```toml
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I"]
```

အောက်ပါ `models.py` ကို စစ်ကြည့်ပါ။

```python
import os
import sys
from pathlib import Path


def predict(x):
    unused = 42
    return x * 2
```

`ruff check models.py` လည်ပတ်ပြီး warnings များကို `--fix` option ဖြင့် အလိုအလျောက် ပြင်ဆင်ပါ။

**Hints:** `"I"` သည် import sorting rule အုပ်စုဖြစ်သည်။ `unused` variable ကို `F841` rule က ဖမ်းပါမည်။

**Expected behavior:** `--fix` အပြီးတွင် `from pathlib import Path` သည် third-party block အဖြစ် စီစဉ်ခံရပြီး unused variable နှင့် မလိုအပ်သော `sys` import ကို ဖျက်ပြီး "All checks passed!" ရရှိပါမည်။

## လေ့ကျင့်ခန်း ၃ — pytest test ပထမဆုံးရေးခြင်း

`math_utils.py` နှင့် `test_math_utils.py` ကို အောက်ပါအတိုင်း ဖန်တီးပါ။

```python
# math_utils.py
def safe_divide(a, b):
    if b == 0:
        raise ValueError("division by zero")
    return a / b
```

```python
# test_math_utils.py
import pytest
from math_utils import safe_divide

def test_normal_division():
    assert safe_divide(10, 2) == 5

def test_zero_division():
    with pytest.raises(ValueError):
        safe_divide(10, 0)
```

`pytest` ကို လည်ပတ်ပါ။

**Hints:** test files နှင့် functions သည် `test_` prefix ဖြင့် စရမည်ဖြစ်သည်။ `pytest.raises` က exception ဖြစ်မှုကို စစ်ဆေးပါသည်။

**Expected behavior:** `pytest` က test ၂ ခုလုံး pass ဖြစ်ကြောင်း `2 passed` ဟု အစိမ်းရောင်ဖြင့် ပြသပါမည်။

## လေ့ကျင့်ခန်း ၄ — fixture များဖြင့် test data ပြင်ဆင်ခြင်း

`test_dataset.py` တွင် fixture သုံး၍ sample data များကို စီမံပါ။

```python
import pytest

@pytest.fixture
def sample_records():
    # Provide a small reusable data set for tests
    return [
        {"id": 1, "score": 0.9},
        {"id": 2, "score": 0.4},
        {"id": 3, "score": 0.75},
    ]

def test_count(sample_records):
    assert len(sample_records) == 3

def test_high_scores(sample_records):
    high = [r for r in sample_records if r["score"] > 0.5]
    assert len(high) == 2
```

fixture ကို တစ်ကြိမ်သာ ဖန်တီးပြီး test များစွာတွင် မျှဝေသုံးနိုင်ရန် `scope="module"` ကို စမှတ်ကြည့်ပါ။

**Hints:** Fixture parameter name သည် function နာမည်နှင့် တူရမည်။ `pytest --setup-show` ဖြင့် fixture lifecycle ကို ကြည့်နိုင်သည်။

**Expected behavior:** Test ၂ ခုလုံး pass ဖြစ်ပြီး `scope="module"` ထည့်ပါက `SETUP    M sample_records` ကို module တစ်ခုတွင် တစ်ကြိမ်သာ ဖန်တီးသည်အဖြစ် တွေ့ရမည်။

## လေ့ကျင့်ခန်း ၅ — API key ကို .env ဖြင့် စီမံခြင်း

`.env` ဖိုင်တွင် `API_KEY=sk-demo-12345` ဟု ထည့်ပြီး `.gitignore` တွင် `.env` ကို ထည့်ပါ။ အောက်ပါ code ဖြင့် ဖတ်ယူပါ။

```python
import os

def get_api_key():
    # Read the key from an environment variable, never hard-code it
    key = os.getenv("API_KEY")
    if not key:
        raise RuntimeError("API_KEY is not set")
    return key

if __name__ == "__main__":
    print("key loaded:", bool(get_api_key()))
```

`git status` ဖြင့် `.env` ကို git က မမြင်တော့စေရန် အတည်ပြုပါ။ Test တစ်ခုတွင် `monkeypatch.setenv("API_KEY", "test-key")` သုံး၍ စမှတ်ပါ။

**Hints:** `pip install python-dotenv` လိုပါက `load_dotenv()` ဖြင့် `.env` ကို ဖွင့်နိုင်သည်။ `monkeypatch` သည် pytest built-in fixture ဖြစ်သည်။

**Expected behavior:** `get_api_key()` က hard-coded key မပါဘဲ environment မှ ဖတ်ယူပြီး၊ `git status` တွင် `.env` မပေါ်ဘဲ monkeypatch test က pass ဖြစ်ပါမည်။

## လေ့ကျင့်ခန်း ၆ — pre-commit gate တပ်ဆင်ခြင်း

`.pre-commit-config.yaml` ကို ဖန်တီးပါ။

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
        args: ["--fix"]
      - id: ruff-format
```

Project root တွင် `pre-commit install` လုပ်ပါ။ ဒီဇိုင်းမညီသော code တစ်ခုကို commit ကြို့ကြည့်ပါ။

**Hints:** ပထမအကြိမ် လည်ပတ်ပါက `pre-commit` က hook environments များကို download လုပ်သဖြင့် အချိန်အနည်းငယ် ကြာမည်။ `rev` ကို လက်ရှိ release ဖြင့် ပြောင်းနိုင်သည်။

**Expected behavior:** Formatting မညီသော file ကို commit ရန် ကြိုးစားပါက hook က ဖိုင်ကို အလိုအလျောက် ပြင်ပြီး commit ကို ရပ်တန့်၍ ထပ်မံ stage လုပ်ရန် တောင်းဆိုပါမည်။
