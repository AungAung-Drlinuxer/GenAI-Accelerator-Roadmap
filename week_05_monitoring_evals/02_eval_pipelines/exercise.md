## လေ့ကျင့်ခန်း ၁ — Evaluation Dataset ဖန်တီးခြင်း

`eval_dataset.jsonl` ဆိုသော file တစ်ခုကို JSON Lines format ဖြင့် ဖန်တီးပါ။ ၁၀ ခုသော test case ပါဝင်ရမည် — ၄ ခုမှာ factual QA၊ ၃ ခုမှာ summarization၊ ၃ ခုမှာ code explanation task ဖြစ်ရမည်။ တစ်ကြောင်းလျှင် `id`၊ `category`၊ `input`၊ `expected_keywords` (list of strings) ဟူ၍ field များ ပါဝင်ရမည်။

**Hints:** JSONL ဆိုသည်မှာ တစ်ကြောင်းလျှင် JSON object တစ်ခုပါဝင်သော format ဖြစ်သည်။ `expected_keywords` ထဲ့မှာ answer တွင် မဖြစ်မနေ ပါသင့်သော စကားလုံးများကို ထည့်ပါ။

**Expected behavior:** `wc -l eval_dataset.jsonl` ပြေးလျှင် `10` ရရှိပြီး Python `json.loads()` ဖြင့် တစ်ကြောင်းချင်းစီကို အောင်မြင်စွာ parse လုပ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၂ — Deterministic Keyword Scoring Function

Python function `score_keywords(response: str, expected_keywords: list[str]) -> float` ကိုရေးပါ။ Response ထဲတွင် keyword တိုင်း lowercase ဖြင့် ပါဝင်လျှင် အမှတ်ပေးပြီး ၀ မှ ၁.၀ အတွင်း score ဖြင့် return လုပ်ရမည်။ ထို့နောက် လေ့ကျင့်ခန်း ၁ မှ dataset ကို ဖတ်၍ စမ်းသပ်ပါ။

```python
def score_keywords(response: str, expected_keywords: list[str]) -> float:
    # Return fraction of expected keywords found (case-insensitive)
    if not expected_keywords:
        return 1.0
    found = sum(1 for kw in expected_keywords if kw.lower() in response.lower())
    return found / len(expected_keywords)
```

**Hints:** `open(..., encoding="utf-8")` ဖြင့် file ကိုဖွင့်ပြီး `for line in f:` ဖြင့် တစ်ကြောင်းချင်း process လုပ်ပါ။ Dataset ထဲက `expected_keywords` ကို တိုက်ရိုက် အသုံးပြုနိုင်သည်။

**Expected behavior:** Function သည် keyword များအားလုံးပါလျှင် `1.0`၊ တစ်ခုမှမပါလျှင် `0.0` return လုပ်သည်။ Dataset ရှိ case ၁၀ ခုလုံးအတွက် score များ print ထုတ်ပြနိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — pytest ဖြင့် Golden-Output Regression Test

လေ့ကျင့်ခန်း ၂ မှ scorer ကို အသုံးပြု၍ `test_eval.py` ထဲတွင် pytest test များ ရေးပါ။ အနည်းဆုံး ၃ ခု — (a) score အတွက် unit test၊ (b) dataset file တွင် id များ ထပ်နေခြင်း မရှိစေရန် test၊ (c) သတ်မှတ်ထားသော fixed response string တစ်ခုက score threshold `0.8` ကို ကျော်ရန် test။

**Hints:** `pip install pytest` ဖြင့် install လုပ်ပါ။ Test function များကို `test_` ဖြင့် စတင်ရေးပြီး `pytest test_eval.py -v` ဖြင့် ပြေးပါ။ `assert` statement များကို အသုံးပြုပါ။

**Expected behavior:** `pytest test_eval.py -v` ပြေးလျှင် test ၃ ခုလုံး `PASSED` ဖြစ်သည်။ Threshold ကို `1.2` ဟု ပြောင်းလျှင် (c) test သည် `FAILED` ဖြစ်သည်။

## လေ့ကျင့်ခန်း ၄ — LLM-as-Judge 评分 Function

`score_with_judge(question: str, response: str, criteria: str) -> float` ဆိုသော function ရေးပါ။ ယခုအဆင့်တွင် judge အဖြစ် မိမိကိုယ်တိုင် လက်ဖြင့် အမှတ်ပေးမည့် mock version ဖြစ်သော `mock_judge.py` module ကို ရေးပြီး၊ နောက်ပိုင်းတွင် API call နှင့် လွယ်ကူစွာ လဲလှယ်နိုင်ရန် interface ကို ဒီဇိုင်းပါ။ Judge prompt တွင် question, response၊ criteria၊ ပြီးလျှင် "0.0 မှ 1.0 အတွင်း မှတ်ချက်တစ်ခုသာ ထုတ်ပေးပါ" ဟူသော ညွှန်ကြားချက် ပါဝင်ရမည်။

**Hints:** Mock judge သည် response ရှည်လျှင် အမှတ်နည်း၊ criteria ထဲက စကားလုံး ပါဝင်လျှင် အမှတ်တိုးသကဲ့သို့ ရိုးရှင်းသော rule များဖြင့် စမ်းသပ်နိုင်သည်။ Function signature ကိုမပြောင်းဘဲ implementation ကိုသာ နောက်မှ လဲလှယ်နိုင်ရန် စဉ်းစားပါ။

**Expected behavior:** `score_with_judge()` ကို မတူသော response ၂ ခုဖြင့် ခေါ်လျှင် မတူသော score များ ရရှိသည်။ Judge implementation ကို လဲလှယ်ဖို့ calling code ကို မထိပါ။

## လေ့ကျင့်ခန်း ၅ — Regression Gate: Threshold ဖြင့် Deploy Block

`run_evals.py` script ရေးပါ။ Dataset ရှိ case များအားလုံးကို run ပြီး average score တွက်ရမည်။ `--threshold` argument ကိုလက်ခံပြီး average score သည် threshold အောက်နိမ့်လျှင် exit code `1` ဖြင့် ပြေးပြီး "REGRESSION DETECTED" ဟု print ရမည်။ Threshold ကျော်လျှင် exit code `0` ဖြင့် အောင်မြင်စွာ ပြီးမြောက်ရမည်။

**Hints:** `sys.argv` သို့မဟုတ် `argparse` module ဖြင့် threshold ကို လက်ခံပါ။ Exit code အတွက် `sys.exit(1)` ကို အသုံးပြုပါ။ Response များကို ယခုအဆင့်တွင် ကြိုတင်သိမ်းထားသော `responses.jsonl` (သို့) dataset ၏ expected content များဖြင့် အစားထိုးနိုင်သည်။

**Expected behavior:** `python run_evals.py --threshold 0.3` ပြေးလျှင် exit code `0`၊ `--threshold 0.95` ပြေးလျှင် exit code `1` ရရှိသည်။ `echo $?` ဖြင့် စစ်ဆေးနိုင်သည်။

## လေ့ကျင့်ခန်း ၆ — CI Pipeline တွင် Eval ထည့်သွင်းခြင်းး

GitHub Actions workflow file `.github/workflows/eval.yml` ကို ရေးပါ။ Python ၃.၁၁ setup လုပ်ပြီး dependencies install ၍ `pytest test_eval.py` ကို run ၍ ထို့နောက် `python run_evals.py --threshold 0.7` ကို run ရမည်။ Eval တစ်ခုမှ ပျက်လျှင် pipeline က တစ်ပြိုင်နက် ရပ်တန့် (fail) သွားရမည်။ README တစ်လက်ဖြင့် prompt version တစ်ခု တစ်ခေါက deploy ချင်း threshold ကို မည်သို့ တစ်ဆင့်တိုး မြှင့်နိုင်/လျှော့နိုင်ကြောင်း မှတ်တမ်းတင်ပါ။

**Hints:** Workflow တွင် `on: push` trigger ထည့်ပါ။ pytest သို့မဟုတ် eval script မှ non-zero exit code ထွက်လျှင် Actions runner က job ကို automatically fail အဖြစ် မှတ်ယူသည်။ Threshold သမိုင်းကို Git commit message များတွင် မှတ်တမ်းတင်ခြင်းက ရိုးရှင်းသော version tracking နည်းတစ်ခု ဖြစ်သည်။

**Expected behavior:** Commit ပြီး push လျှင် Actions tab တွင် eval job ပြေးသည်ကို မြင်ရမည်။ Threshold ကို ကြီးမားစွာ မြှင့်လျှင် job အနီရောင် (fail) ဖြစ်ပြီး deploy မပြုလုပ်ရန် အချက်ပြုသကဲ့သို့ အလုပ်လုပ်သည်။
