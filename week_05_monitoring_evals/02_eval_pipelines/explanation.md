# Week 5 — LLM Monitoring & Evaluations
## Evaluation Pipelines & Regression Gates

အီးလ်အယ်အမ် (LLM) system တွေကို production မှာ သုံးတဲ့အခါ "ဒီ version က အရင် version ထက် ပိုကောင်းလား၊ ပိုဆိုးလား" ဆိုတာကို အချက်အလက်နဲ့ ဖြေရှင်းရပါတယ်။ ဒီ module မှာ dataset ဒီဇိုင်း၊ deterministic scoring၊ LLM-judge scoring၊ CI integration နဲ့ regression gate တွေကို လေ့လာမယ်။

## 1. Dataset Design

### ဘာကို ဆိုလိုတာလဲ

Evaluation dataset ဆိုတာက prompt-version တစ်ခုချင်းစီရဲ့ အရည်အသွေးကို တိုင်းတာဖို့ သီးခြားပြင်ဆင်ထားတဲ့ test case စု ဖြစ်ပါတယ်။ Test case တစ်ခုမှာ input (user ရဲ့မေးခွန်း) နဲ့ မျှော်မှန်းရမယ့် ရလဒ် (expected behavior) ပါဝင်ပါတယ်။ Software testing မှာ unit test သုံးသလို ML system အတွက် eval dataset ကို သုံးတာပါ။

### ဘာကြောင့် လဲ

LLM output က deterministic မဟုတ်ပါ။ Model တစ်ခုကို တစ်ခါတည်း မေးကြည့်ရုံနဲ့ အရည်အသွေးကို မသိနိုင်ပါ။ Dataset က အားနည်းချက်ကို ဖုံးကွယ်ပေးနိုင်ပြီး prompt ပြောင်းတဲ့အခါ ဘယ် behavior တွေ ပြန်ဆိုးသွားလဲ (regression) ဆိုတာကို မြင်ရစေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

ကောင်းတဲ့ eval dataset တစ်ခုမှာ ဒီအချက်တွေ ပါဝင်သင့်ပါတယ်:

- **Real production inputs** — စစ်မှန်တဲ့ user question တွေကနေ ထုတ်ယူထားတာ (PII တွေကို anonymize လုပ်ဖို့ မမေ့ပါနဲ့)
- **Edge cases** — ရှားပါတဲ့ input တွေ၊ adversarial input တွေ
- **Label ရှင်းလင်းမှု** — "ကောင်းတယ်/မကောင်းဘူး" ဆိုတဲ့ သတ်မှတ်ချက်ကို လူနှစ်ယောက် အထက် တူညီအောင် စာရင်းအင်းနည်းနဲ့ စစ်ဆေးခြင်း (inter-annotator agreement)
- **Coverage** — system ကလုပ်ပေးရမယ့် task အမျိုးအစားအားလုံးကို ကုံစွာခြုံခြင်း

### ဥပမာ

```python
# Example: building an eval dataset as a simple list of dicts
eval_dataset = [
    {
        "id": "qa_001",
        "input": "What is the return policy for damaged items?",
        "expected_keywords": ["return", "damaged", "refund"],
        "category": "policy_question",
    },
    {
        "id": "qa_002",
        "input": "ygybuhijnlkoiuhgf",  # adversarial: keyboard mash
        "expected_behavior": "refuse_or_clarify",
        "category": "edge_case",
    },
]

# Check dataset balance: how many cases per category?
from collections import Counter
counts = Counter(case["category"] for case in eval_dataset)
print(counts)
# Expected output: Counter({'policy_question': 1, 'edge_case': 1})
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Dataset ညံ့ရင် score တွေက မှားယွင်းတဲ့ ယုံကြည်မှု ပေးပါတယ်။ အလွယ်တွေချည်း ပါဝင်နေရင် system က ပြဿနာ ရှုပ်ထွေးတဲ့ ကိစ္စတွေမှာ မအောင်မြင်ပဲ ကောင်းနေသလို မြင်ရမှာပါ။ Production ကနေ sample ယူပြီး dataset ကို အမြဲ update လုပ်သင့်ပါတယ်။

## 2. Deterministic vs LLM-Judge Scoring

### ဘာကို ဆိုလိုတာလဲ

Deterministic scoring က rule-based ဖြစ်ပါတယ် — exact match, keyword ပါမပါ၊ JSON schema မှန်မမှန်၊ length check တွေကို Python နဲ့ တိုက်ရိုက်စစ်တာပါ။ LLM-judge scoring က အခြား model (သို့) တူညီတဲ့ model ကို grader အဖြစ် သုံးပြီး output ရဲ့ အရည်အသွေးကို rubric အတိုင်း အမှတ်ပေးခိုင်းတာပါ။

### ဘာကြောင့် လဲ

Deterministic check တွေက မြန်ပြီး၊ ဈေးကျပြီး၊ အကြိမ်ရာပေး တူညီပါတယ်။ ဒါပေမယ့် "ဒီအဖြေက မှန်လား၊ ထိုင်ဖက်သလား" ဆိုတာကို စစ်လို့ မရပါ။ LLM-judge က အဲဒီမျိုး ပွင့်လင်းတဲ့ မေးခွန်းတွေအတွက် သင့်လျော်ပေမယ့် score က တစ်ရက်ချင်း မတူနိုင်ပါ (temperature, model version ပေါ်မူတည်ပြီး)။ နှစ်မျိုးလုံးကို တွဲသုံးတာက အကောင်းဆုံးပါ။

### ဘယ်လို အလုပ်လုပ်လဲ

လက်တွေ့အားဖြင့် — format နဲ့ structure အတွက် deterministic check၊ semantic အရမှန်ကန်မှုအတွက် LLM-judge ကို သုံးပါတယ်။ LLM-judge prompt မှာ rubric ရှင်းလင်းစွာ ရေးပေးရပြီး 1-5 လိုမျိုး ကျဉ်းကျဉ်း score ထုတ်ခိုင်းတာက ရိုးရိုးရှင်းရှင်း "ကောင်း/မကောင်း" ထုတ်ခိုင်းတာထက် ပိုတည်ငြာပါတယ်။ Judge model ရဲ့ မှန်ကန်မှုကို လူက စစ်ထားတဲ့ golden answers နဲ့ ယှဉ်ပြီး calibration လုပ်သင့်ပါတယ်။

### ဥပမာ

```python
# Example: combining a deterministic check with a simple judge function
def deterministic_check(output, expected_keywords):
    # Rule-based: every expected keyword must appear in the output
    lowered = output.lower()
    missing = [kw for kw in expected_keywords if kw not in lowered]
    return len(missing) == 0, missing

output = "You can request a refund for damaged items within 30 days."
passed, missing = deterministic_check(output, ["return", "damaged", "refund"])
print(passed, missing)
# Expected output: False ['return']

def llm_judge_prompt(question, answer, rubric):
    # Build a judge prompt with a fixed rubric to reduce variance
    return (
        "You are grading a customer-support answer.\n"
        f"Question: {question}\n"
        f"Answer: {answer}\n"
        f"Score 1-5 based on this rubric: {rubric}\n"
        "Reply with only the integer score."
    )

rubric = "5 = fully correct and safe; 1 = wrong or harmful"
print(llm_judge_prompt("What is the return policy?", output, rubric)[:80])
# Expected output: You are grading a customer-support answer.
# Question: What is the return policy?
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Score မတိကျရင် regression gate တွေက အဓိပ္ပာယ်မရှိပါ။ Deterministic က အနည်းဆုံး format regression တွေကို မြန်မြန် ဖမ်းပေးပြီး၊ LLM-judge က ဆိုင်းငန်းတဲ့ အရည်အသွေး ကျဆင်းမှုကို ဖမ်းပေးပါတယ်။ Judge ကိုပဲ အားကိုးရင် judge model တစ်ခုတည်း ပြောင်းသွားတာနဲ့ score တွေ အားလုံး လှိမ့်သွားနိုင်လို့ အန္တရာယ် ရှိပါတယ်။

## 3. Running Evals in CI

### ဘာကို ဆိုလိုတာလဲ

CI (Continuous Integration) ဆိုတာက pull request တိုင်း၊ commit တိုင်းမှာ test တွေ အလိုအလျောက် ပြေးခိုင်းတဲ့ စနစ်ပါ (GitHub Actions လို)။ LLM eval တွေကိုပါ အဲဒီ pipeline ထဲ ထည့်ခြင်းကို "running evals in CI" လို့ ခေါ်ပါတယ်။ Pytest ရဲ့ `parametrize` နဲ့ exit code တွေကို သုံးပြီး LLM test တွေကို ထုံးစံအတိုင်း ပြေးအောင် လုပ်နိုင်ပါတယ်။

### ဘာကြောင့် လဲ

ပုံမှန် software မှာ test တွေ ပျက်ရင် merge မလုပ်ပါနဲ့ ဆိုတဲ့ စည်းမျဉ်း ရှိပေမယ့် LLM app တွေက test မပါပဲ ထွက်နေတတ်ပါတယ်။ CI ထဲ ထည့်မှပို့ရင် prompt တစ်ကြောင်း ပြောင်းတာ၊ model တစ်ခု upgrade လုပ်တာ၊ retrieval config ပြောင်းတာတွေက production quality ကို သက်ရောက်မှုရှိမရှိကို အလိုအလျောက် သိရမှာပါ။

### ဘယ်လို အလုပ်လုပ်လဲ

အဓိက အဆင့်တွေက — (၁) eval တွေကို pytest test အဖြစ် ရေး၊ (၂) expensive LLM-judge eval တွေကို nightly တွေမှာ ပြေးပြီး deterministic တွေကို PR တိုင်းမှာ ပြေး၊ (၃) eval ရလဒ်တွေကို Langfuse လို tracing platform ထဲ ပို့၊ (၄) exit code မတူရင် CI က fail အဖြစ် သတ်မှတ်။

### ဥပမာ

```python
# File: test_evals.py
# Run with: pytest test_evals.py -v
import pytest

eval_cases = [
    {"id": "qa_001", "input": "What is the return policy?", "must_contain": "return"},
    {"id": "qa_002", "input": "Talk to a human agent.", "must_contain": "support"},
]

@pytest.mark.parametrize("case", eval_cases, ids=[c["id"] for c in eval_cases])
def test_response_contains_key_fact(case):
    # Deterministic check that runs fast on every pull request
    response = get_bot_response(case["input"])  # your app under test
    assert case["must_contain"] in response.lower(), (
        f"Case {case['id']} regression: keyword missing"
    )
# Expected output: 2 passed in 0.34s  (all keywords present)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Manual eval လုပ်တာက မေ့တတ်ပြီး တစ်ခါပြီးရင် ရပ်တတ်ပါတယ်။ CI ထဲမှာ ပြေးနေတဲ့ eval က မမေ့နိုင်ပါ — တစ်ခါတစ်ရံ ဖွင့်ရုံပဲ အလုပ်လုပ်နေမယ်။ Expensive eval တွေကို schedule ခွဲခြင်းက cost ကို ထိန်းပေးပြီး deterministic တွေက fast feedback ပေးပါတယ်။

## 4. Regression Gates: Thresholds That Block a Deploy

### ဘာကို ဆိုလိုတာလဲ

Regression gate ဆိုတာက eval score တွေကို ကြိုတင်သတ်မှတ်ထားတဲ့ threshold နဲ့ ယှဉ်ပြီး မမှီရင် deploy ကို အလိုအလျောက် ရပ်ဆိုင်းတဲ့ စည်းမျဉ်းပါ။ ဥပမာ — pass rate ၉၅ ရာခိုင်ႏှုံးအောက် ကျရင် merge မလုပ်ပါနဲ့ ဆိုတာမျိုးပါ။

### ဘာကြောင့် လဲ

အဖွဲ့အစည်းတွေက regression ကို မျက်နှာဖြင့် သိတတ်ပြီး အဲဒီအချိန်မှာ user တွေက နစ်နာပြီးသား ဖြစ်နေတတ်ပါတယ်။ Threshold တစ်ခု သတ်မှတ်ပြီး fail ရင် pipeline ကို ရပ်ခိုင်းတာက ကျန်းမာတဲ့ default ဖြစ်စေပါတယ် — "ကျော်လို့ရတယ်" ဆိုတာကို ရည်ရွယ်ချက်ရှိရှိ ရွေးရမှ ဖြစ်ပါမယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

(၁) လက်ရှိ version ရဲ့ baseline score ကို တိုင်းပါ။ (၂) threshold ကို baseline ထက် အနည်းငယ် နိမ့်တဲ့ နေရာမှာ ထားပါ — ဘာကြောင့်ဆို LLM score တွေက noise ရှိလို့ အတိအကျ တူညီအောင် မထားသင့်ပါ။ (၃) CI script မှာ threshold fail ရင် non-zero exit code ထုတ်ပါ။ (၄) Critical category (safety, policy) တွေအတွက် threshold ကို ပိုတောင်းပါ။

### ဥပမာ

```python
# Example: a simple regression gate function used at the end of an eval run
def evaluate_run(results, baseline_pass_rate, baseline_judge_score):
    # results: list of dicts, each with "passed" (bool) and "judge_score" (1-5)
    pass_rate = sum(1 for r in results if r["passed"]) / len(results)
    avg_judge = sum(r["judge_score"] for r in results) / len(results)

    gates = {
        "pass_rate": pass_rate >= baseline_pass_rate,      # e.g. 0.95
        "judge_score": avg_judge >= baseline_judge_score,  # e.g. 4.0
    }
    return pass_rate, avg_judge, gates

sample_results = [
    {"passed": True, "judge_score": 5},
    {"passed": True, "judge_score": 4},
    {"passed": False, "judge_score": 3},
]
pass_rate, avg_judge, gates = evaluate_run(sample_results, 0.95, 4.0)
print(f"pass_rate={pass_rate:.2f}, judge={avg_judge:.2f}, gates={gates}")

if not all(gates.values()):
    raise SystemExit(1)  # CI treats exit code 1 as a blocking failure
# Expected output: pass_rate=0.67, judge=4.00, gates={'pass_rate': False, 'judge_score': True}
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Gate မရှိရင် eval ရလဒ်တွေက info သက်သက် ဖြစ်နေပြီး ဘယ်သူမှ တာဝန် မယူပါ။ Threshold တွေက "ဒီပြောင်းလဲမှုက accept လုပ်မလုပ်" ဆိုတဲ့ ဆုံးဖြတ်ချက်ကို ရှင်းလင်းစေပြီး product decision ကို engineering discussion အဖြစ် ပြောင်းပေးပါတယ်။

## 5. Tracking Scores Over Prompt Versions

### ဘာကို ဆိုလိုတာ
