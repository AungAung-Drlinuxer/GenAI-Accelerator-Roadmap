# Quality Toolchain: ruff, pytest, secrets hygiene

AI engineering project များအတွက် code အရည်အသွေးကို အလိုအလျောက်စစ်ဆေးပေးမည့် toolchain သုံးမျိုး — ruff, pytest နှင့် secrets hygiene — ကို လက်တွေ့အသုံးချတတ်စေရန် သင်ကြားပေးသည်။

## ဒီ module မှာ ဘာသင်မလဲ

- ruff ဖြင့် Python code ကို lint လုပ်ခြင်းနှင့် format လုပ်ခြင်း
- pytest ဖြင့် test ဖိုင်ဖွဲ့စည်းပုံနှင့် test ရေးနည်း
- pytest fixtures သုံးပြီး test data ကို ပြန်လည်အသုံးချခြင်း
- API keys များကို environment variables များထဲ သိမ်းဆည်းခြင်း
- `.env` ဖိုင်ကို git မှ ဖယ်ရှားခြင်း (`.gitignore`)
- pre-commit hook များဖြင့် code တင်မခင် အရည်အသွေးစစ်ကြေးခံခြင်း

## သင်ခန်းစာများ

1. **ruff basics** — `ruff check` နှင့် `ruff format` ကို တပြိုင်နက်သုံးပြီး lint + format လုပ်နည်း။
2. **ruff configuration** — `pyproject.toml` ထဲမှာ rule selection နှင့် per-file ignores သတ်မှတ်နည်း။
3. **pytest fundamentals** — test ဖိုင် naming convention, `assert`, နှင့် test discovery အလုပ်လုပ်ပုံ။
4. **Fixtures** — `@pytest.fixture` ဖြင့် shared test data နှင့် setup/teardown စီမံနည်း၊ `conftest.py`။
5. **Parameterized tests** — `@pytest.mark.parametrize` ဖြင့် ဖြစ်နိုင်ခြေများ ကွဲပြားသော input များကို တစ်ပိတ် test လုပ်နည်း။
6. **Secrets hygiene** — API key များကို `os.environ` / `python-dotenv` ဖြင့် ဖတ်ယူပြီး source code ထဲ မရေးသွင်းရခြင်းအကြောင်း။
7. **.env ကို git က ကာကွယ်ခြင်း** — `.gitignore` ထည့်သွင်းခြင်း၊ မသတိထားမိဘူး key ကို history ထဲပါသွားပါက လုပ်ရမည့်အရာ။
8. **Pre-commit gates** — `pre-commit` framework သုံးပြီး commit မတင်ခင် ruff နှင့် secret scanning ကို အလိုအလျောက် လည်ပတ်စေနည်း။

## လိုအပ်ချက်များ (Prerequisites)

- Python 3.10 နှင့်အထက် installed ဖြစ်ရမည်
- pip ဖြင့် package install လုပ်နိုင်စွမ်း၊ virtual environment ဖန်တီးနိုင်စွမ်း
- Git အခြေခံ command များ (`init`, `add`, `commit`) သိရမည်
- Python function, dict, module အခြေခံများ နားလည်ရမည်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- AI model API ချိတ်ဆက်သည့် project တစ်ခု စတင်ထူထောင်ချိန်၊ lint/test setup ကို ပထမဆုံး လုပ်သင့်သည်
- LLM provider များ၏ API keys ကို အသုံးပြုရသည့်အခါ key ဖမ်းမိမှု (leak) မှ ကာကွယ်ရန်
- အဖွဲ့လိုက် အလုပ်လုပ်သည့်အခါ code style တူညီစေရန်နှင့် broken code ကို main branch ထဲ မရောက်စေရန်
- Production တင်မည့် code တွင် regression များ ကြိုသိနိုင်ရန် automated tests ထားလိုသည့်အခါ

## ကိုးကား

- ruff official documentation — <https://docs.astral.sh/ruff/>
- pytest official documentation — <https://docs.pytest.org/>
- pre-commit framework — <https://pre-commit.com/>
- python-dotenv (PyPI) — <https://pypi.org/project/python-dotenv/>
