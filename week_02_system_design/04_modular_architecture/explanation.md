# Modular Architecture: Workflows, Nodes, DI

## Orchestration နှင့် Steps များကို ခွဲခြားခြင်း

### ဘာကို ဆိုလိုတာလဲ
Workflow တစ်ခုဆိုတာ အဆင့် (step) ငယ်များစွာကို အစီအစဉ်ရှိရှိ ခေါ်တွေ့ပေးသော "orchestration layer" ဖြစ်သည်။ Node တစ်ခုစီက အလုပ်တစ်ခုကိုသာ လုပ်ပြီး၊ workflow က ဘယ် node ကို ဘယ်အစီအစဉ်နှင့် ဘယ် data နှင့် ခေါ်မလဲဆိုတာကို သီးခြား ဆုံးဖြတ်သည်။

### ဘာကြောင့် လဲ
Logic အားလုံးကို function ကြီးတစ်ခုထဲ ရေးထားပါက အစိတ်အပိုင်းတစ်ခု ပြင်လိုက်ရင် တစ်ခုလုံး ပြိုမိနိုင်သည်။ Orchestration နှင့် steps ကို ခွဲလိုက်ခြင်းအားဖြင့် node တစ်ခုစီကို သီးခြား စမ်းသပ်နိုင်၊ ပြန်အသုံးပြုနိုင်ပြီး workflow အစီအစဉ်ကိုပါ လွယ်လင့်တကူ ပြင်ဆင်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Node တစ်ခုကို `run(input) -> output` ပုံစံနှင့် သီးခြား ဖန်တီးသည်။ Workflow class တစ်ခုက node စာရင်းကို ထိန်းပြီး အစီအစဉ်အတိုင်း data ကို တစ် node မှ တစ် node လွှဲပေးသည်။

### ဥပမာ
```python
# Define small, independent nodes
def load_text(source: str) -> str:
    return source.strip()

def count_words(text: str) -> int:
    return len(text.split())

# Orchestration layer: decides order and data flow only
class Workflow:
    def __init__(self, steps):
        self.steps = steps

    def run(self, source: str):
        data = source
        for step in self.steps:
            data = step(data)
        return data

wf = Workflow(steps=[load_text, count_words])
print(wf.run("  modular design is powerful  "))
# Expected output: 4
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI application များတွင် prompt building, model call, parsing, validation စသည့် အဆင့်များရှိသည်။ ၎င်းတို့ကို ခွဲထားပါက model ကိုပြောင်းလိုသောအခါ node တစ်ခုတည်းကိုသာ ပြင်ရုံဖြင့် ရှင်းရှင်းလင်းလင်း ဖြေရှင်းနိုင်သည်။

## Typed Node Inputs/Outputs

### ဘာကို ဆိုလိုတာလဲ
Node တိုင်း၏ input နှင့် output type ကို dataclass သို့မဟုတ် type hints ဖြင့် ကြေညာခြင်းဖြစ်သည်။ ဥပမာ — `LoadResult`, `ParseResult` ကဲ့သို့ အဆင့်စီတွင် သီးခြား type သတ်မှတ်ခြင်း။

### ဘာကြောင့် လဲ
String သို့မဟုတ် dict ယေဘုယျ type ချည်း အလယ်အလတ် data အဖြစ် သုံးပါက အဆင့်တစ်ခုချင်းစီက ဘယ် field လိုအပ်သည်ကို မသိရဘဲ  run-time မှာမှ အမှားများ ပေါ်လာတတ်သည်။ Type များ သတ်မှတ်ခြင်းအားဖြင့် အမှားကို အစောပိုင်းမှာ ဖမ်းနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Python `dataclass` များဖြင့် node အဆင့်စီ၏ output ကို ကိုယ်စားပြုပြီး function signature တွင် type hints ထည့်သည်။ `mypy` ကဲ့သို့ static checker များဖြင့် စစ်နိုင်သည်။

### ဥပမာ
```python
from dataclasses import dataclass

@dataclass
class RetrievedDocs:
    query: str
    docs: list[str]

@dataclass
class Answer:
    text: str

def retrieve(query: str) -> RetrievedDocs:
    # Simulated retrieval step
    return RetrievedDocs(query=query, docs=["doc one", "doc two"])

def generate(res: RetrievedDocs) -> Answer:
    # Simulated generation step based on docs
    return Answer(text=f"Answer using {len(res.docs)} docs for '{res.query}'")

result = generate(retrieve("what is RAG?"))
print(result.text)
# Expected output: Answer using 2 docs for 'what is RAG?'
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Type safety ရှိပါက workflow ကို ပြင်ဆင်သည့်အခါ node များကြား မလိုက်ဖက်မှုကို တွေ့ရန် လွယ်သည်။ IDE များကလည်း autocomplete နှင့် error warning ပေးနိုင်၍ အဖွဲ့လိုက် ဖန်တီးသည့်အခါ ဆက်သွယ်မှု ပိုတိကျသည်။

## Dependency Injection (DI) ကို Testability အတွက် အသုံးပြုခြင်း

### ဘာကို ဆိုလိုတာလဲ
Node ၏အပြင်ဘက် အမျိုးအစားများ (model client, database, config) ကို node အတွင်းမှ တိုက်ရိုက် ဖန်တီးခြင်းအစား၊ အပြင်မှ လက်ဆင်းပေးလိုက်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Node အတွင်းမှာ client ကို တိုက်ရိုက် ဖန်တီးပါက စမ်းသပ်တဲ့အခါ real API ကိုပဲ ခေါ်မိမည်။ DI သုံးပါက စမ်းသပ်ချိန်တွင် fake/mock client ထည့်ပေးနိုင်၍ မကြာခဏ၊ အခမဲ့၊ တည်ငြိမ်စွာ test ရေးနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Constructor သို့မဟုတ် function parameter များမှတစ်ဆင့် dependency ကို လက်ခံသည်။ Production တွင် real object၊ test တွင် fake object ပေးလိုက်သည်။

### ဥပမာ
```python
from dataclasses import dataclass

@dataclass
class Answer:
    text: str

class FakeModelClient:
    # Test double: returns a fixed response without network calls
    def complete(self, prompt: str) -> str:
        return f"FAKE:{prompt}"

class AnswerNode:
    def __init__(self, model_client):
        self.model = model_client  # dependency injected from outside

    def run(self, question: str) -> Answer:
        prompt = f"Question: {question}\nAnswer:"
        return Answer(text=self.model.complete(prompt).strip())

node = AnswerNode(model_client=FakeModelClient())
print(node.run("what is DI?").text)
# Expected output: FAKE:Question: what is DI?
# Answer:
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
AI model call များသည် ကုန်ကျစရိတ်ရှိပြီး ရလဒ်များ မတည်ငြိုးနိုင်။ Fake dependency များဖြင့် business logic ကိုသာ အသေအချာ စစ်ဆေးနိုင်ပြီး၊ model ကိုယ်တိုင်ကို နှောင့်နှေးမှု သီးခြား စမ်းသပ်နိုင်သည်။

## Failure Isolation (အမှား အပိုင်းခွဲ ထိန်းချုပ်ခြင်း)

### ဘာကို ဆိုလိုတာလဲ
Node တစ်ခု ပျက်စီးသည့်အခါ တစ်ခုလုံး system ပျက်သွားခြင်းအစား အမှားကို ထို node အတွင်းမှာသာ ကန့်သတ်ခြင်း၊ retry သို့ fallback လုပ်ခြင်းဖြင့် ကျန် nodes များ ဆက်လက်အလုပ်လုပ်နိုင်ရန် ဆောင်ရွက်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
External service (API, vector store) များကို မှီခိုသော AI pipeline များတွင် တစ်ခုခု နှောင့်နှေးခြင်း၊ ပျက်စီးခြင်းသည် သာမန်အဖြစ်အပျက်ဖြစ်သည်။ Failure ကို ကန့်သတ်နိုင်ပါက system တစ်ခုလုံး၏ တည်ငြိမ်မှု တိုးလာသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Node တစ်ခုစီကို try/except ဖြင့် ရစ်ပတ်ထားပြီး၊ မှားလျှင် ရှင်းရှင်းလင်းလင်း error message တစ်ခု အပေါ်ဆင့် တင်ပေးသည်။ Workflow က error အမျိုးအစားအရ retry လည်းလုပ်နိုင်၊ fallback node ကိုလည်း ခေါ်နိုင်သည်။

### ဥပမာ
```python
class NodeFailed(Exception):
    pass

def risky_node(value: int) -> int:
    if value < 0:
        raise NodeFailed(f"invalid input: {value}")
    return value * 2

def safe_run(node, value, fallback_value: int) -> int:
    # Isolate failure: catch, report, and fall back
    try:
        return node(value)
    except NodeFailed as err:
        print(f"[warn] node failed: {err}; using fallback")
        return fallback_value

print(safe_run(risky_node, 5, fallback_value=0))
print(safe_run(risky_node, -1, fallback_value=0))
# Expected output:
# 10
# [warn] node failed: invalid input: -1; using fallback
# 0
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production တွင် အသုံးပြုသူသည် နောက်ခံ error များ မမြင်လိုပါ။ Failure isolation ရှိပါက တစ်စိတ်တစ်ဒေသ ပျက်နိုင်စဉ်ကျန် feature များ အလုပ်လုပ်နေဆဲဖြစ်ပြီး၊ ပြဿနာရှိသော node ကို log အရ လွယ်ကူစွာ ရှာဖွာနိုင်သည်။

## Observability Hooks

### ဘာကို ဆိုလိုတာလဲ
Workflow အလယ်တွင် log, metric သို့ timing ထုတ်ယူနိုင်ရန် ထည့်သွင်းထားသော အမှတ်အသားများ (hooks) ဖြစ်သည် — node စတင်ခြင်း၊ ပြီးဆုံးခြင်း၊ ပျက်စီးခြင်းတိုင်းကို မှတ်တမ်းတင်ခြင်း။

### ဘာကြောင့် လဲ
AI pipeline များသည် ရလဒ် မတည်ငြိုးနိုင်၍ ဘယ် node က အမှားရှိသည် သို့မဟုတ် နှောင့်နှေးသည်ကို မသိပါက debug လုပ်ရန် ခက်ခဲသည်။ Hooks များက အဆင့်စီ၏ အခြေအနေကို မြင်သာစေသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Workflow က node တိုင်း စတင်/ပြီးဆုံးချိန်တွင် hook function များကို ခေါ်သည်။ Hook က log ရေးနိုင်၊ duration တိုင်းနိုင်၊ metric ပို့နိုင်သည်။ Business logic နှင့် logging ကို ခွဲထားသောကြောင့် node များကို မညှူနှောဘဲ ရေးနိုင်သည်။

### ဥပမာ
```python
import time

def observe(event: str, node_name: str, **info):
    # Simple observability hook: could log to a real backend
    print(f"[trace] {event} node={node_name} {info}")

class Workflow:
    def __init__(self, steps):
        self.steps = steps

    def run(self, data):
        for step in self.steps:
            name = getattr(step, "__name__", str(step))
            observe("start", name)
            t0 = time.perf_counter()
            data = step(data)
            observe("end", name, seconds=round(time.perf_counter() - t0, 4))
        return data

def upper(text: str) -> str:
    return text.upper()

result = Workflow(steps=[upper]).run("hello")
print(result)
# Expected output:
# [trace] start node=upper
# [trace] end node=upper seconds=0.0
# HELLO
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production တွင် "ဘယ်အဆင့်က နှေးနေလဲ"၊ "ဘယ် prompt က ရလဒ်ညံ့နေလဲ" ကို သိရန် observability မရှိမဖြစ် လိုအပ်သည်။ Hooks များကို အလုပ်လုပ်ချက် နှောင့်နှေးစဉ် troubleshooting၊ cost tracking နှင့် quality monitoring အတွက် အခြေခံအသုံးပြုကြသည်။

## အနှစ်ချုပ်
- Orchestration နှင့် steps ကို ခွဲခြားခြင်းအားဖြင့် workflow တစ်ခုလုံးကို စီမံရလွယ်ကူပြီး node များကို ပြန်အသုံးပြုနိုင်သည်။
- Typed inputs/outputs က node များကြား စာချုပ် (contract) ရှင်းလင်းစေပြီး အမှားများကို အစောပိုင်း ဖမ်းနိုင်သည်။
- Dependency injection က node များကို test လုပ်ရလွယ်စေပြီး fake/mock dependency များဖြင့် အမြန်၊ စရိတ်မကုန် test ရေးနိုင်သည်။
- Failure isolation က တစ် node ပျက်လျှင် တစ်ခုလုံး မပျက်စေဘဲ retry/fallback ဖြင့် system တည်ငြိုမှု တိုးစေသည်။
- Observability hooks က အဆင့်တိုင်း၏ log၊ duration နှင့် အခြေအနေကို မြင်သာစေ၍ debugging နှင့် monitoring ကို လွယ်ကူစေသည်။
