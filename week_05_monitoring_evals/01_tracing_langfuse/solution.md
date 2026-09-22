# Solutions — Tracing with Langfuse

```python
# Shared setup for all solutions (run once)
from langfuse import Langfuse

langfuse = Langfuse()
```

## လေ့ကျင့်ခန်း ၁ — ပထမ trace ဖန်တီးပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

# Create the first trace with a name and input
trace = langfuse.trace(
    name="exercise-1-first-trace",
    input={"message": "Hello Langfuse!"},
)

url = trace.get_trace_url()
print(url)
# Expected output: a Langfuse UI URL for this trace
```

**အဓိကအယူအဆ** — Trace ဆိုတာ user request တစ်ခုရဲ့ ပြင်ပအမှတ်အသားဖြစ်ပြီး URL တစ်ခုနဲ့ ဝင်ကြည့်နိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Span တွေ nested ပုံစံ ထည့်ပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

trace = langfuse.trace(name="exercise-2-spans", input={"q": "order status"})

# Step 1: retrieval span
retrieve_span = trace.span(name="retrieve")
docs = ["doc1: orders ship in 2 days."]
retrieve_span.end(output={"docs": docs})

# Step 2: formatting span
format_span = trace.span(name="format")
formatted = "Context: orders ship in 2 days."
format_span.end(output={"formatted": formatted})

print(trace.get_trace_url())
# Expected output: trace URL; two spans appear inside the trace tree
```

**အဓိကအယူအဆ** — Span တွေက trace ထဲရဲ့ အဆင့်တစ်ခုချင်းစီကို အချိန်နဲ့တွဲပြီး မှတ်တမ်းတင်ပေးပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Generation capture လုပ်ပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

trace = langfuse.trace(name="exercise-3-generation")

# Record a full LLM call with prompt, completion and usage
generation = trace.generation(
    name="llm-answer",
    model="gpt-4o-mini",
    input=[
        {"role": "system", "content": "You are a support agent."},
        {"role": "user", "content": "How do I reset my password?"},
    ],
    output="Go to Settings > Security and click Reset Password.",
    usage={"input": 25, "output": 14, "total": 39},
)
print(generation.name)
# Expected output: llm-answer
```

**အဓိကအယူအဆ** — Generation ဆိုတာ LLM ခေါ်ဆိုမှုရဲ့ prompt, completion, token usage သုံးခုလုံးကို တစ်နေရာတည်း သိမ်းထားတဲ့ span အထူး ဖြစ်ပါတယ်။

## လေ့ကျင့်ခန်း ၄ — Session နဲ့ user တွဲပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

# Two turns of the same user in the same session
trace_a = langfuse.trace(
    name="exercise-4-turn-1",
    sessionId="session-ex4",
    userId="user-ex4",
    input={"message": "Hi"},
)
trace_b = langfuse.trace(
    name="exercise-4-turn-2",
    sessionId="session-ex4",
    userId="user-ex4",
    input={"message": "What can you do?"},
)
print(trace_a.get_trace_url())
print(trace_b.get_trace_url())
# Expected output: two trace URLs; both grouped under session-ex4 / user-ex4
```

**အဓိကအယူအဆ** — sessionId နဲ့ userId က trace တွေကို စကားပြောဆက်နဲ့ လူတစ်ယောက်အလိုက် ချိတ်ပေးတဲ့ grouping key တွေ ဖြစ်ပါတယ်။

## လေ့ကျင့်ခန်း ၅ — Feature tags တပ်ပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

# One trace per feature, each with its own tags
sum_trace = langfuse.trace(
    name="exercise-5-summarizer",
    tags=["feature:summarizer", "env:dev"],
    input={"doc_id": "doc-1"},
)
qa_trace = langfuse.trace(
    name="exercise-5-qa-bot",
    tags=["feature:qa-bot", "env:dev"],
    input={"question": "What is Langfuse?"},
)
print(sum_trace.get_trace_url())
print(qa_trace.get_trace_url())
# Expected output: two trace URLs; each shows its feature and env tags
```

**အဓိကအယူအဆ** — Tag တွေက trace တွေကို feature, environment, version အလိုက် ပြန်ရှာဖို့နဲ့ နှိုင်းယှဉ်ဖို့ အသုံးဝင်ဆုံး label တွေ ဖြစ်ပါတယ်။

## လေ့ကျင့်ခန်း ၆ — မှားတဲ့ အဖြေတစ်ခုကို debug လုပ်ပါ

```python
from langfuse import Langfuse

langfuse = Langfuse()

# Deliberately broken retrieval: picks the wrong document
def broken_retrieve(question):
    return "doc-99: unrelated billing policy text"

def answer_question(question):
    trace = langfuse.trace(
        name="exercise-6-debug",
        tags=["feature:qa-bot", "env:dev"],
        input={"question": question},
    )
    # Step 1: record retrieval so the bug is visible
    span = trace.span(name="retrieve")
    docs = broken_retrieve(question)
    span.end(output={"docs": docs})
    # Step 2: the model answers using that (wrong) context
    answer = "According to our billing policy, contact support."
    trace.generation(
        name="llm-answer",
        model="gpt-4o-mini",
        input={"question": question, "docs": docs},
        output=answer,
        usage={"input": 60, "output": 12, "total": 72},
    )
    trace.update(output=answer)
    return answer

result = answer_question("How do I reset my password?")
print(result)
print("Inspect the retrieve span output to find the wrong doc:", "doc-99")
# Expected output: the bad answer; the retrieve span output exposes the real root cause
```

**အဓိကအယူအဆ** — အဖြေမှားတာရဲ့ အခြေခံအကြောင်းပြချက်ကို trace span output တွေမှာ မြင်ရပြီး ခန့်မှန်းချက်မဟုတ်ဘဲ အထောက်အထားနဲ့ debug လုပ်နိုင်ပါတယ်။
