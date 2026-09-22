## လေ့ကျင့်ခန်း ၁ — Workflow နှင့် Node ခွဲခြားခြင်း

အောက်ပီ `load_text` နှင့် `count_words` ဆိုသော function နှစ်ခုကို ကြည့်ပါ။ ယခုအခါ ၄င်းတို့ကို workflow ထဲတွင် တိုက်ရိုက်ခေါ်နေသော `run_pipeline` ကို **step များကို list တစ်ခုအဖြစ် သီးသန့်ဖော်ထားသော** pipeline အဖြစ် ပြန်ရေးပါ။ orchestration logic (data ကို step တစ်ခုဆီ တစ်ဆင့်ချတစ်ဆင့် ပို့ခြင်း) နှင့် step logic ကို ကွဲပြားအောင် လုပ်ပေးပါ။

```python
def load_text(path):
    with open(path, "r", encoding="utf-8") as f:
        return f.read()

def count_words(text):
    return len(text.split())

# TODO: rewrite as a workflow of separated steps
def run_pipeline(path):
    text = load_text(path)
    return count_words(text)
```

**Hints:** steps ကို `[load_text, count_words]` ကဲ့သို့ list တစ်ခုအဖြစ် သတ်မှတ်ပြီး `run_workflow(data, steps)` ဆိုသော generic orchestrator တစ်ခု ရေးပါ။ orchestrator သည် step တစ်ခုချင်းစီ၏ output ကို နောက် step ၏ input အဖြစ် ပေးပါ။

**Expected behavior:** `run_workflow` ကို ခေါ်လျှင် အတွားတစ်ခုပေးလျှင် စာလုံးအရေအတွက် ရရှိပြီး၊ `load_text` ကို workflow list ထဲမှ ဖယ်ရှားလျှင် pipeline ကို အခြား combination ဖြင့် အလွယ်ပြန်တည်ဆောက်နိုင်သည်။

## လေ့ကျင့်ခန်း ၂ — Typed Node Input/Output များ

Python `dataclass` များဖြင့် node input/output အတွက် type များ သတ်မှတ်ပါ။ `FetchResult` (text ပါဝင်) နှင့် `AnalysisResult` (word_count, char_count ပါဝင်) ဆိုသော dataclass နှစ်ခု ဖန်တီးပြီး `fetch_node(path: str) -> FetchResult` နှင့် `analyze_node(result: FetchResult) -> AnalysisResult` ကို ရေးပါ။ ထို့နောက် မှားသော type ကို ပေးလျှင် `TypeError` တက်စေရန် `analyze_node` ထဲတွင် type check ထည့်ပါ။

```python
from dataclasses import dataclass

# TODO: define FetchResult and AnalysisResult dataclasses
```

**Hints:** `dataclasses.dataclass` decorator ကို အသုံးပြုပြီး `isinstance(result, FetchResult)` စစ်ဆေးမှု ထည့်သွင်းပါ။

**Expected behavior:** `analyze_node(FetchResult(text="hi"))` ကို ခေါ်လျှင် `AnalysisResult` တစ်ခု ရရှိသည်။ `analyze_node("raw string")` ခေါ်လျှင် `TypeError` တက်သည်။

## လေ့ကျင့်ခန်း ၃ — Dependency Injection ဖြင့် Test လွယ်အောင် လုပ်ခြင်း

အောက်ပီ `summarize_service` သည် network call ကို အလုပ်အတွက် တိုက်ရိုက်ခေါ်နေသဖြင့် test လုပ်ရခက်သည်။ external dependency ကို constructor (သို့) parameter မှ inject လုပ်စေပြီး fake version တစ်ခုဖြင့် test ရေးနိုင်အောင် ပြန်ရေးပါ။

```python
def summarize_service(text):
    # hard to test: real network call inside the function
    import requests
    resp = requests.post("https://example.invalid/summarize", json={"text": text})
    return resp.json()["summary"]

# TODO: refactor so the HTTP client is injected
```

**Hints:** `SummarizeService(http_client)` ဆိုသော class တစ်ခု ဖန်တီးပြီး `http_client.post(url, json=payload)` ကို ခေါ်စေပါ။ test အတွက် `FakeClient` class တစ်ခု ရေးပြီး `post` method မှ ကြိုတင်သတ်မှတ်ထားသော result ပြန်စေပါ။

**Expected behavior:** fake client ဖြင့် `SummarizeService` ကို စမ်းသပ်လျှင် network မလိုဘဲ အမှန်ဆိုင်ရာ logic (payload ဖွဲ့ခြင်း၊ result ဖြတ်ယူခြင်း) ကို စစ်ဆေးနိုင်သည်။

## လေ့ကျင့်ခန်း ၄ — Failure Isolation အတွက် Step Retry နှင့် Graceful Skip

လေ့ကျင့်ခန်း ၁ ၏ orchestrator ကို တိုးချဲ့ပါ — step တစ်ခုတွင် exception တက်လျှင် pipeline တစ်ခုလုံး မကျရန်၊ `retry_count` (default 1) အတိုင်း ထပ်မံခေါ်ပြီး နောက်ဆုံးတွင် `None` ဖြင့် ခဏ skip လုပ်စေပါ။ အဆင့်တိုင်း၏ အောင်မြင်/ကျရှုံးမှု အခြေအနေကို `dict` တစ်ခုတွင် မှတ်တမ်းတင်ပါ။

**Hints:** `try/except` ကို step loop ထဲတွင် အသုံးပြုပြီး `status[node_name]` ကို `"ok"` သို့မဟုတ် `"failed"` အဖြစ် မှတ်ပါ။ node များကို `(name, function)` tuple list အဖြစ် ပြောင်းလျှင် အမည်ခေါ်ရလွယ်သည်။

**Expected behavior:** တစ်ခါတမေးပဲ အလုပ်လုပ်သော flaky node တစ်ခုပါသော pipeline သည် retry ကြောင့် အဆင်ပြေစွာ အပြီးသတ်နိုင်ပြီး၊ အမြဲမှားသော node ပါလျှင် pipeline ကျရှုံးစေကာ `status` dict တွင် `"failed"` ဟု မှတ်ပြသည်။

## လေ့ကျင့်ခန်း ၅ — Observability Hooks ထည့်သွင်းခြင်း

လေ့ကျင့်ခန်း ၄ ၏ orchestrator တွင် observability hooks ထည့်ပါ — တစ်ခုမက်ခင် `on_step_start(name, data)`, ပြီးလျှင် `on_step_end(name, result)`, ကျရှုံးလျှင် `on_step_error(name, exception)` ကို ခေါ်ပါ။ hooks များကို inject လုပ်နိုင်ရန် orchestrator သည် `hooks` parameter လက်ခံရမည်။ default hooks များက `print` ဖြင့် log ထုတ်ပါကောင်း၊ no-op ဖြစ်စေပါ။

**Hints:** hooks ကို class `Hooks` တစ်ခုအဖြစ် ဖန်တီးပြီး subclass လုပ်၍ custom logging (ဥပမာ — `logging` module သို့မဟုတ် file ထဲ ရေးခြင်း) စမ်းကြည့်ပါ။

**Expected behavior:** pipeline လည်နေစဉ် တစ်ခုမက်ခင်/ပြီးလျှင် console တွင် log စာကြောင်းများ ထွက်ပေါ်ပြီး၊ step တစ်ခုကျရှုံးလျှင် error hook က exception message ကို ဖော်ပြသည်။ hooks inject မလုပ်လျှင် pipeline သည် ပုံမှန်အတိုင်း အလုပ်လုပ်သည်။

## လေ့ကျင့်ခန်း ၆ — ပေါင်းစပ် Mini Framework တည်ဆောက်ခြင်း

လေ့ကျင့်ခန်း ၁ မှ ၅ အထိ သင်ယူခဲ့သည်များကို ပေါင်းစပ်၍ mini workflow framework တစ်ခု တည်ဆောက်ပါ — `WorkflowBuilder` class (steps ထည့်ရန် `add_step(name, fn)`), `Workflow.run(input_data, hooks=None)`, typed dataclass inputs/outputs, retry ဖြင့် failure isolation နှင့် observability hooks အားလုံး ပါဝင်ရမည်။ ဆုံးတွင် sample pipeline (load → clean → analyze) တစ်ခုနှင့် fake dependency အသုံးပြုထားသော test function တစ်ခု ရေးပြီး စမ်းသပ်ပါ။

**Hints:** steps ကို internal list ထဲသိမ်းပြီး `run` ထဲတွင် loop, retry, hooks, status dict တို့ကို ပေါင်းစပ်ပါ။ clean step အတွက် ဥပမာ — `text.strip().lower()` ဟု မူဝါဒတစ်ခု သတ်မှတ်နိုင်သည်။

**Expected behavior:** `WorkflowBuilder().add_step("load", load_step).add_step("analyze", analyze_step)` ကဲ့သို့ chain လုပ်၍ pipeline တည်ဆောက်နိုင်ပြီး၊ `run` ခေါ်လျှင် hooks log များ၊ status dict တစ်ခုနှင့် နောက်ဆုံး output တို့ ရရှိသည်။ flaky step တစ်ခု ထည့်စမ်းပါက retry အလုပ်လုပ်သည်ကို တွေ့ရမည်။
