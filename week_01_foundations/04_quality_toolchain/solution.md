# Week 1 — Quality Toolchain: ruff, pytest, secrets hygiene (Solution)

## လေ့ကျင့်ခန်း ၁ — ruff ဖြင့် lint လုပ်ခြင်းနှင့် format လုပ်ခြင်း

`ruff` သည် Python code ရှိ style အမှားများနှင့် အလွန်အကျွံ import များကို ရှာဖွေပေးသည့် အလွန်မြန်ဆန်သော lint tool ဖြစ်သည်။ `pyproject.toml` ထဲတွင် စည်းမျဉ်းများ သတ်မှတ်ပြီး `ruff check` ဖြင့် စစ်ဆေးနိုင်သည်။

```python
# file: check_with_ruff.py
# Run: pip install ruff
# Then: ruff check . --fix   (auto-fix safe issues)
# Then: ruff format .       (format like black)

import os          # unused import -> ruff flags this (F401)
import json        # unused import -> ruff flags this (F401)


def add_numbers(a, b):
    # ruff format will normalize spacing and quotes
    result = a + b
    return result


def divide(a, b):
    # ruff rule B902/B (bugbear) style checks catch risky code patterns
    if b == 0:
        raise ValueError("b must not be zero")
    return a / b


if __name__ == "__main__":
    print(add_numbers(2, 3))
```

`pyproject.toml` တွင် အောက်ပါအတိုင်း စည်းမျဉ်း ရွေးနိုင်သည် —

```toml
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "B"]  # pycodestyle, pyflakes, isort, bugbear
```

**အဓိကအယူအဆ** — lint သည် code ပေးရန်မလိုဘဲ ဖတ်ရလွယ်ပြီး အမှားနည်းသော style ကို အလိုအလျောက် အာမခံပေးသဖြင့် reviewer အချိန်ကို သိမ်းဆည်းပေးသည်။

## လေ့ကျင့်ခန်း ၂ — pytest ဖွဲ့စည်းပုံနှင့် fixtures

`pytest` သည် `test_` နှင့် စတင်သော function များကို အလိုအလျောက် ရှာဖွေ၍ run ပေးသည်။ `fixture` များကို အသုံးပြု၍ စမ်းသပ်မှုတစ်ခုစီအတွက် ကြိုတင်ပြင်ဆင်ရမည့် data များကို နေရာချထားနိုင်သည်။

```python
# file: test_math_utils.py
# Run: pip install pytest
# Then: pytest -v

import pytest


# A fixture provides reusable setup data for tests.
# pytest injects it by matching the parameter name.
@pytest.fixture
def sample_numbers():
    # This dict is created fresh for each test that requests it
    return {"a": 10, "b": 5}


def add(a, b):
    return a + b


def divide(a, b):
    if b == 0:
        raise ValueError("b must not be zero")
    return a / b


def test_add(sample_numbers):
    # fixture values are injected as the argument
    result = add(sample_numbers["a"], sample_numbers["b"])
    assert result == 15


def test_divide(sample_numbers):
    assert divide(sample_numbers["a"], sample_numbers["b"]) == 2.0


def test_divide_by_zero_raises():
    # pytest.raises verifies that the expected exception occurs
    with pytest.raises(ValueError):
        divide(1, 0)


@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),      # simple case
    (-1, 1, 0),     # negative numbers
    (0, 0, 0),      # zero edge case
])
def test_add_many_cases(a, b, expected):
    assert add(a, b) == expected
```

project တစ်ခု၏ သာမန် ဖွဲ့စည်းပုံမှာ —

```
project/
  src/
    math_utils.py
  tests/
    test_math_utils.py
  pyproject.toml
```

**အဓိကအယူအဆ** — fixture များနှင့် `parametrize` ကို အသုံးပြုခြင်းအားဖြင့် setup code ကို ထပ်မရေးဘဲ စမ်းသပ်မှုများစွာကို တစ်နေရာတည်းမှ စီမံနိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — API key များကို env var တွင်သိမ်းခြင်း၊ .env ကို git မှ ဖယ်ရှားခြင်း

API key များကို code ထဲ တိုက်ရိုက် ရေးသွင်းခြင်းသည် key ယိုစိမ့်မှု၏ အဓိကအကြောင်းရင်း ဖြစ်သည်။ key များကို environment variable များတွင် သိမ်းဆည်းပြီး `.env` file ကို `.gitignore` ထဲ ထည့်သွင်းရမည်။

```python
# file: load_secret.py
# Run: pip install python-dotenv
# Create a .env file (NEVER commit it) with:
#   OPENAI_API_KEY=sk-your-key-here

import os

from dotenv import load_dotenv


def get_api_key() -> str:
    # Load key-value pairs from a local .env file into os.environ
    load_dotenv()

    # Read the key from the environment, fail loudly if missing
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise RuntimeError(
            "OPENAI_API_KEY is not set. "
            "Add it to your .env file or export it in your shell."
        )
    return api_key


def mask_secret(secret: str, visible: int = 4) -> str:
    # Show only the last few characters when logging
    if len(secret) <= visible:
        return "****"
    return "*" * (len(secret) - visible) + secret[-visible:]


if __name__ == "__main__":
    key = get_api_key()
    # Never print the full key; always mask it
    print("Loaded key:", mask_secret(key))
```

`.gitignore` file တွင် အောက်ပါများ ထည့်သွင်းရမည် —

```
.env
.env.*
```

key တစ်ခု မတော်တဆ commit လုပ်မိပါက `git rm --cached .env` ဖြင့် remove လုပ်ပြီး key ကို ချက်ချင်း rotate (အသစ်ပြောင်း) လုပ်ရမည်။

**အဓိကအယူအဆ** — secret များသည် code မဟုတ်ဘဲ environment မှ လာရမြဲဖြစ်ပြီး `.env` ကို git မှ လုံးဝ ကင်းရှင်းစေခြင်းသည် key ယိုစိမ့်မှုကို အခြေခံကျိုးဆိုင်းပေးသည်။

## လေ့ကျင့်ခန်း ၄ — pre-commit gate တပ်ဆင်ခြင်း

`pre-commit` သည် `git commit` မတင်မီ အလိုအလျောက် စစ်ဆေးမှုများ run ပေးသည့် tool ဖြစ်သည်။ အောက်ပါ config file ကို project root တွင် ထားပြီး `pre-commit install` ဟု အသုံးပြုလျှင် git hook အဖြစ် တပ်ဆင်ပြီးဖြစ်သည်။

```yaml
# file: .pre-commit-config.yaml
# Run: pip install pre-commit
# Then: pre-commit install
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff          # lint and auto-fix issues
        args: [--fix]
      - id: ruff-format   # format the code
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: check-added-large-files  # block huge files
      - id: detect-private-key       # block committed keys
      - id: end-of-file-fixer        # ensure trailing newline
      - id: trailing-whitespace      # strip trailing spaces
```

တပ်ဆင်ပြီးနောက် `git commit` တိုင်းတွင် — lint မှားရှိလျှင် commit ကို ဆိုင်းငံ့၍ အလိုအလျောက် ပြင်ဆင်ပေးမည်။ ပြင်ပြီးပါက `git add` ပြန်လုပ်ကာ commit ပြန်တင်ရမည်။

**အဓိကအယူအဆ** — စစ်ဆေးမှုများအားလုံးကို commit မတင်ခင် အလိုအလျောက် တပ်ဆင်ထားခြင်းအားဖြင့် အဖွဲ့တစ်ခုလုံး တစ်သားတည်းကျသော code standard ကို စက်ဖြင့် အာမခံနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — API key ကို .env ဖြင့် စီမံခြင်း

အရင်ဆုံး ပရောဂက်တ် folder အတွင်း `.env` ဖိုင်ကို ဖန်တီးပြီး `API_KEY=sk-demo-12345` ဟု ရိုက်ထည့်ပါ။ ထို့နောက် `.gitignore` ဖိုင်တွင် `.env` ဆိုသည့် စာကြောင်းကို ထည့်သွင်းပါ။ ဤသို့ ပြုလုပ်ခြင်းအားဖြင့် secret key များကို git repository ထဲသို့ မတင်ဘဲ လုံခြုံစွာ စီမံနိုင်ပါသည်။ `python-dotenv` package ကို သုံးပါက `.env` ဖိုင်ထဲရှိ value များကို environment variable အဖြစ် အလိုအလျောက် ဖွင့်နိုင်ပါသည်။ `monkeypatch.setenv()` သည် pytest ၏ built-in fixture ဖြစ်ပြီး test အတွင်း၌ environment variable ကို ယာယီ သတ်မှတ်ပေးသဖြင့် တကယ့် key ကို မသုံးဘဲ test ကို အောင်မြင်စေနိုင်ပါသည်။ Test ပြီးဆုံးသည်နှင့် monkeypatch က မူလ environment ကို အလိုအလျောက် ပြန်ပြင်ပေးသဖြင့် အခြား test များပေါ် သက်ရောက်မှု မရှိပါ။

```python
import os
from dotenv import load_dotenv


def get_api_key():
    # Read the key from an environment variable, never hard-code it
    key = os.getenv("API_KEY")
    if not key:
        raise RuntimeError("API_KEY is not set")
    return key


if __name__ == "__main__":
    # Load variables from the .env file into the environment
    load_dotenv()
    print("key loaded:", bool(get_api_key()))
```

Test ဖိုင် (ဥပမာ `test_key.py`) ကို အောက်ပါအတိုင်း ရေးပါ။

```python
from mymodule import get_api_key


def test_get_api_key_returns_test_value(monkeypatch):
    # Set a temporary environment variable only for this test
    monkeypatch.setenv("API_KEY", "test-key")
    assert get_api_key() == "test-key"


def test_get_api_key_raises_when_missing(monkeypatch):
    # Remove the variable to simulate an unset key
    monkeypatch.delenv("API_KEY", raising=False)
    try:
        get_api_key()
        assert False, "expected RuntimeError"
    except RuntimeError:
        pass
```

`pytest` ဖြင့်  run ပြီးနောက် `git status` ကို ထုတ်ကြည့်ပါက `.env` ဖိုင်ကို git က ထည့်သွင်းမမြင်တော့သည်ကို တွေ့ရပါမည်။

**အဓိကအယူအဆ** — Secret key များကို `.env` ဖိုင်တွင် သိမ်းဆည်းပြီး `.gitignore` ဖြင့် git မှ ကာကွယ်၍၊ code က environment variable မှတစ်ဆင့် ဖတ်ယူပြီး test များတွင် `monkeypatch.setenv()` ဖြင့် ယာယီ value သတ်မှတ် စမှတ်သင့်သည်။

## လေ့ကျင့်ခန်း ၆ — pre-commit gate တပ်ဆင်ခြင်း

pre-commit သည် commit မလုပ်မီအခြေအနေ၌ code ကို စစ်ဆေးပြီး formatting နှင့် lint ပြဿနာများကို ဖမ်းဆီးပေးသည့် gate တစ်ခုဖြစ်သည်။ ပထမဦးစွာ project root တွင် `.pre-commit-config.yaml` ဖိုင်ကို ဖန်တီးရမည်။ ဖိုင်အတွင်း၌ `ruff-pre-commit` repository ကို `v0.4.4` version ဖြင့် ညွှန်းဆိုပြီး `ruff` hook (auto-fix ပါဝင်သော `--fix` argument ဖြင့်) နှင့် `ruff-format` hook နှစ်ခုကို မှတ်ပုံတင်ရမည်။

Config ဖိုင် ပြင်ဆင်ပြီးပါက `pre-commit install` command ဖြင့် local Git repository ၌ hook များကို တပ်ဆင်ရမည်။ ထို့နောက် ဒီဇိုင်းမညီသော Python code တစ်ခုကို ဖန်တီး၍ commit ကြိုးစားကြည့်ရမည် — hook က ဖိုင်ကို အလိုအလျောက် ပြင်ဆင်ပြီး commit ကို ရပ်တန့်သွားမည်ဖြစ်သည်။

အောက်ပါ script သည် ဆိုင်ရာ config ဖိုင်ကို ရေးသွင်းပြီး hook installation နှင့် မညီသော code ဖိုင် ဖန်တီးခြင်း အဆင့်များကို အလိုအလျောက် လုပ်ဆောင်ပေးသည်။

```python
import subprocess
from pathlib import Path

# Define the pre-commit configuration content
config_content = """repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
        args: ["--fix"]
      - id: ruff-format
"""

# Write the config file at the project root
config_path = Path(".pre-commit-config.yaml")
config_path.write_text(config_content, encoding="utf-8")
print(f"Created {config_path}")

# Install the pre-commit hooks into the local Git repository
result = subprocess.run(
    ["pre-commit", "install"],
    capture_output=True,
    text=True,
)
print(result.stdout)
if result.returncode != 0:
    print(result.stderr)
    raise SystemExit("pre-commit install failed")

# Create a badly formatted Python file to test the gate
bad_code = """import os,sys
def  greeting( name ):
    x=f"Hello, {name}"
    return x
"""
bad_file = Path("bad_style.py")
bad_file.write_text(bad_code, encoding="utf-8")
print(f"Created {bad_file} with intentionally bad formatting")

# Stage the file and attempt a commit to observe hook behavior
subprocess.run(["git", "add", str(bad_file)], check=True)
commit_result = subprocess.run(
    ["git", "commit", "-m", "test badly formatted code"],
    capture_output=True,
    text=True,
)
print("Commit output:")
print(commit_result.stdout)
print(commit_result.stderr)
```

Script ကို လည်ပတ်ပြီးနောက် commit ကြိုးစားမှုက ရပ်တန့်သွားပြီး hook output အတွင်း၌ `ruff-format` က ဖိုင်ကို ပြင်ဆင်ပြီးဖြစ်ကြောင်း ပြသမည်။ ထိုအခါ `git diff` ဖြင့် ပြင်ပြီးသော ပြောင်းလည်းမှုများကို စစ်ဆေးနိုင်ပြီး `git add` ဖြင့် ထပ်မံ stage လုပ်ကာ commit ကို ထပ်မံကြိုးစားနိုင်သည်။ ပထမအကြိမ် လည်ပတ်ခြင်း၌ hook environments များကို download လုပ်ရသဖြင့် အချိန်အနည်းငယ် ကြာမည်ကို မှတ်ထားရမည်။

**အဓိကအယူအဆ** — pre-commit hook များသည် ဒီဇိုင်းမညီသော code များကို commit မလုပ်မီအခြေအနေ၌ အလိုအလျောက် ပြင်ဆင်ပြီး commit ကို ရပ်တန့်စေခြင်းဖြင့် repository အတွင်း ဝင်ရောက်လာမည့် code ၏ အရည်အသွေးကို အမြဲတမ်း အာမခံပေးသည်။
