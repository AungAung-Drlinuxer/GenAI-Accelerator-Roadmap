# Week 2 — Modular Architecture: Workflows, Nodes, DI (solution.md)

## လေ့ကျင့်ခန်း ၁ — Orchestration နှင့် Steps ခွဲခြားခြင်း

Orchestration (workflow) ဆိုသည်မှာ steps များကို အစဉ်အတန်း သတ်မှတ်ပေးသော logic သာဖြစ်ပြီး၊ တစ်ခုချင်းစီ၏ အလုပ်ကို node တွေအတွင်းမှာ သီးသန့်ထားရမည်။

```python
# orchestration.py - separate the workflow from the steps

def load_data(source: str) -> dict:
    """A single step: knows only its own job."""
    # pretend we read from a file or API
    return {"rows": source.split(",")}

def clean_data(data: dict) -> dict:
    """Another step, independent of the workflow above."""
    cleaned = [r.strip() for r in data["rows"]]
    return {"rows": cleaned}

def run_workflow(source: str) -> dict:
    """The orchestrator: only decides the ORDER of steps."""
    step1 = load_data(source)
    step2 = clean_data(step1)
    return step2

if __name__ == "__main__":
    result = run_workflow("apple , banana , cherry")
    print(result)  # {'rows': ['apple', 'banana', 'cherry']}
```

**အဓိကအယူအဆ** — Orchestration layer သည် steps များ၏ အစဉ်အတန်းကိုသာ ဆုံးဖြတ်ပြီး အလုပ်အစစ်ကို node တစ်ခုစီက ကိုယ့်ဘာသာ တာဝန်ယူဆောင်ရွက်သင့်သည်။

## လေ့ကျင့်ခန်း ၂ — Typed Node Inputs/Outputs

Node တစ်ခုစီ၏ input/output ကို type များဖြင့် ရှင်းလင်းစွာ သတ်မှတ်ခြင်းက data contract တစ်ခု ဖြစ်စေသည်။

```python
# typed_nodes.py - use dataclasses as node contracts

from dataclasses import dataclass

@dataclass
class RawText:
    text: str

@dataclass
class Tokens:
    words: list

@dataclass
class WordCount:
    count: int

def tokenize(node_input: RawText) -> Tokens:
    """Node with typed input and typed output."""
    return Tokens(words=node_input.text.lower().split())

def count_words(node_input: Tokens) -> WordCount:
    """Next node accepts exactly the previous output type."""
    return WordCount(count=len(set(node_input.words)))

if __name__ == "__main__":
    raw = RawText(text="Hello world hello again")
    tokens = tokenize(raw)
    result = count_words(tokens)
    print(result.count)  # 3 unique words
```

**အဓိကအယူအဆ** — Typed inputs/outputs ရှိလျှင် node များကြား မတူညီသော data ပုံစံများ မှားယွင်းစီးဆင်းမှုကို ကြိုတင်ဖမ်းဆုပ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — Dependency Injection ဖြင့် Testability

Node အတွင်းရှိ external dependencies (model client, database, ...) ကို အပြင်ကနှ ထိုးသွင်းပေးခြင်းအားဖြင့် test ရလွယ်သော ပုံစံပြောင်းနိုင်သည်။

```python
# di_nodes.py - inject dependencies instead of creating them inside

class FakeModelClient:
    """A lightweight stand-in for the real model client."""
    def complete(self, prompt: str) -> str:
        return "FAKE: " + prompt

class SummarizerNode:
    def __init__(self, model_client):
        # dependency is injected, not created here
        self.model_client = model_client

    def run(self, text: str) -> str:
        prompt = f"Summarize: {text}"
        return self.model_client.complete(prompt)

if __name__ == "__main__":
    # in production we would pass a real client
    node = SummarizerNode(model_client=FakeModelClient())
    print(node.run("Long document text"))
    # test without a real API call or cost
```

**အဓိကအယူအဆ** — Dependency Injection သည် node ကို concrete client တစ်ခုတည်းနှင့် တွယ်အားမခံစေဘဲ fake objects ဖြင့် လွယ်ကူစွာ စမ်းသပ်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၄ — Failure Isolation

Workflow တွင် node တစ်ခု ပျက်စီးလျှင် အခြား nodes များကို ထိခိုက်မစေရန် error handling ကို node အဆင့်တွင် ထားရှိရမည်။

```python
# failure_isolation.py - catch per-node errors, keep the pipeline alive

def risky_step(value: int) -> int:
    """This step may fail for bad input."""
    if value == 0:
        raise ValueError("value must not be zero")
    return 10 // value

def safe_step(value: int) -> int:
    return value + 1

def run_workflow(values: list) -> list:
    results = []
    for v in values:
        try:
            # failure is contained to this single item
            out = safe_step(risky_step(v))
            results.append({"input": v, "output": out, "status": "ok"})
        except Exception as exc:
            results.append({"input": v, "error": str(exc), "status": "failed"})
    return results

if __name__ == "__main__":
    for r in run_workflow([2, 0, 5]):
        print(r)
```

**အဓိကအယူအဆ** — Failure isolation ဆိုသည်မှာ node တစ်ခု သို့မဟုတ် item တစ်ခု ပျက်စီးသည့်တိုင် workflow တစ်ခုလုံး ရပ်တန့်မသွားစေဘဲ ကျန်အစိတ်အပိုင်းများ ဆက်လက်လည်ပတ်နိုင်စေခြင်းဖြစ်သည်။

## လေ့ကျင့်ခန်း ၅ — Observability Hooks

Workflow အတွင်း ဖြစ်ပျက်ချက်များကို စောင့်ကြည့်နိုင်ရန် node များတွင် logging/tracing hooks ထည့်သွင်းပေးရမည်။

```python
# observability.py - simple hooks around every node execution

import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("workflow")

def with_hooks(node_name: str, func, payload):
    """Wrapper that records start, end and timing of each node."""
    logger.info("node=%s status=started", node_name)
    try:
        result = func(payload)
        logger.info("node=%s status=finished", node_name)
        return result
    except Exception:
        logger.exception("node=%s status=error", node_name)
        raise

def uppercase_step(text: str) -> str:
    return text.upper()

if __name__ == "__main__":
    output = with_hooks("uppercase_step", uppercase_step, "hello hooks")
    print(output)  # HELLO HOOKS
```

**အဓိကအယူအဆ** — Observability hooks များက workflow ၏ အချိန်ကုန်မှု၊ အမှားများနှင့် အဆင့်တစ်ခုချင်းစီ၏ အခြေအနေကို အချိန်နှင့်တပြိုင်နက် မြင်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၆ — ပေါင်းစပ် Mini-Workflow

အထက်ဖော်ပြပါ သဘောတရားများအားလုံး (orchestration, typing, DI, isolation, observability) ကို ပေါင်းစပ်ထားသော workflow အသေးစားတစ်ခု ရေးကြည့်ပါ။

```python
# mini_workflow.py - all principles combined in one small pipeline

import logging
from dataclasses import dataclass

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("mini")

@dataclass
class PipelineInput:
    text: str

@dataclass
class PipelineOutput:
    summary: str

class SimpleClient:
    def complete(self, prompt: str) -> str:
        return prompt[:20] + "..."

def build_pipeline(client):
    """Orchestrator builder with an injected dependency."""
    def run(node_input: PipelineInput) -> PipelineOutput:
        # typed input goes in, typed output comes out
        try:
            raw = client.complete(node_input.text)
            logger.info("pipeline finished")
            return PipelineOutput(summary=raw)
        except Exception:
            # isolate the failure and report it instead of crashing
            logger.exception("pipeline failed")
            return PipelineOutput(summary="ERROR")
    return run

if __name__ == "__main__":
    pipeline = build_pipeline(SimpleClient())
    print(pipeline(PipelineInput(text="A very long document that needs a summary")))
```

**အဓိကအယူအဆ** — Modularity ၏ အခြေခံများဖြစ်သော orchestration/step ခွဲခြားမှု၊ typed contracts, dependency injection, failure isolation နှင့် observability တို့ကို တွဲဖက်အသုံးချလျှင် စမ်းသပ်ရလွယ်ကူပြီး ပြုပြင်စရာနည်းသော AI system တစ်ခု ရရှိသည်။
