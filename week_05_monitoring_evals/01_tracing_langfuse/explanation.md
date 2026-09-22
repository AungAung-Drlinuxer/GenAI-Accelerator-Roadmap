# Tracing with Langfuse

## ၁။ Trace, Span, Generation

### ဘာကို ဆိုလိုတာလဲ

Trace ဆိုတာက LLM application တစ်ခုက တစ် request အတွင်း လုပ်ဆောင်ချက် အားလုံးရဲ့ မှတ်တမ်း ဖြစ်ပါတယ်။ Span က trace ထဲက တစ်ဆင့်စီရဲ့ အချိန်ကုန် မှတ်တမ်း ဖြစ်ပြီး၊ generation က LLM ခေါ်ဆိုမှု တစ်ခု အတွက် သီးခြား span အမျိုးအစား ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ

LLM app တစ်ခုက prompt ဆင့်ပြီး၊ tool ခေါ်ပြီး၊ အဖြေစုပြီးဆိုတာ အဆင့်များစွာ လုပ်တတ်ပါတယ်။ Console log လေးတွေနဲ့ပဲ ကြည့်ရင် ဘယ်ဆင့်မှာ နှောင့်နှေးနေ၊ ဘယ်ဆင့်မှာ မှားနေလဲဆိုတာ ရှာရခက်ပါတယ်။ Trace structure တစ်ချက်က ဒီပြဿနာကို ဖြေရှင်းပေးပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Langfuse SDK ကို သုံးပြီး trace တစ်ခု ဖန်တီးပြီး၊ အဲဒီထဲမှာ span တွေ၊ generation တွေကို nested ပုံစံနဲ့ ထည့်သွင်းပါတယ်။ `end()` ဆိုတာ ခေါ်တဲ့အခါ duration တွေ အလိုအလျောက် တွက်ပေးပြီး Langfuse UI မှာ tree ပုံစံနဲ့ ပြပါတယ်။

### ဥပမာ

```python
from langfuse import Langfuse

langfuse = Langfuse()

# Create a trace for one user request
trace = langfuse.trace(
    name="support-bot-answer",
    input={"question": "How do I reset my password?"},
)

# Create a nested span for a retrieval step
span = trace.span(name="retrieve-docs")

# Simulate retrieval work
docs = ["doc1: Go to Settings > Security.", "doc2: Click Reset Password."]
span.end(output={"docs": docs})

# Create a generation span for the LLM call
generation = trace.generation(
    name="llm-answer",
    model="gpt-4o-mini",
    input=[{"role": "user", "content": "How do I reset my password?"}],
    output="Go to Settings > Security and click Reset Password.",
    usage={"input": 12, "output": 14, "total": 26},
)
print(trace.get_trace_url())
# Expected output: a URL to view this trace in the Langfuse UI
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production မှာ အဖြေတစ်ခုမှားရင် "ဘာဖြစ်လို့လဲ" ကို trace tree ကိုကြည့်ပြီး ချက်ချင်း ဖြေရှင်းနိုင်ပါတယ်။ ဒါပြီး နောက်ပိုင်း evaluation dataset တွေကိုလည်း trace တွေကနေ ဆွဲထုတ်နိုင်လို့ အရေးကြီးပါတယ်။

## ၂။ Session နဲ့ User

### ဘာကို ဆိုလိုတာလဲ

Session က user တစ်ယောက်ရဲ့ စကားပြောဆက်တစ်ခုလုံးကို ကိုယ်စားပြုတဲ့ grouping ဖြစ်ပြီး၊ user ကတော့ လူတစ်ယောက် (သို့မဟုတ် account တစ်ခု) ကို ကိုယ်စားပြုပါတယ်။

### ဘာကြောင့် လဲ

Trace တစ်ခုချင်းစီက request တစ်ခုပဲ ဖြစ်ပါတယ်။ Chat app တစ်ခုမှာ user တစ်ယောက်က turn ဆယ်ပတ်တွေ ပြောတတ်ပါတယ်။ ဒီတွေကို စုကြည့်ဖို့ session id နဲ့ user id တွေ လိုအပ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

Trace ဖန်တီးတဲ့အခါ `session_id` နဲ့ `user_id` ထည့်ပေးရုံပါပါတယ်။ Langfuse မှာ အဲဒီ id တွေအလိုက် trace တွေကို အလိုအလျောက် စုပေးပါတယ်။

### ဥပမာ

```python
# Trace with session and user grouping
trace = langfuse.trace(
    name="chat-turn",
    sessionId="session-abc-123",
    userId="user-42",
    input={"message": "What is my order status?"},
)
print(trace.get_trace_url())
# Expected output: a trace URL; grouped under session-abc-123 and user-42
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

User complaint တစ်ခုဝင်လာရင် user id နဲ့ သူ့ session တွေကို ရှာပြီး ဘယ် turn မှာ ပြဿနာရှိလဲဆိုတာ မြန်မြန် ရှာနိုင်ပါတယ်။ Cost report တွေလည်း user အလိုက် ခွဲကြည့်နိုင်ပါတယ်။

## ၃။ Prompt, Completion, Usage Capture

### ဘာကို ဆိုလိုတာလဲ

Generation တစ်ခုမှာ LLM ကို ပို့ခဲ့တဲ့ prompt (input)၊ ပြန်ရတဲ့ completion (output)၊ နဲ့ token usage (input/output/total) တွေကို မှတ်တမ်းတင်တာ ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ

အဖြေတစ်ခုကိုတော့ မြင်ရပေမယ့် ဘယ် prompt နဲ့ မေးခဲ့လဲဆိုတာ မမှတ်ထားရင် နောက်ပြန် စစ်လို့မရပါဘူး။ Usage ကတော့ cost နဲ့ performance တွေ တွက်ဖို့ လိုအပ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

LLM ခေါ်တဲ့ code တွေကို generation span တစ်ခုနဲ့ ဖြတ်ရေးပြီး input, output, usage, model တွေကို argument အနေနဲ့ ထည့်ပေးရပါတယ်။ SDK version မတူရင် `usage_details` ဆိုတဲ့ field နာမည်လည်း တွေနိုင်လို့ docs နဲ့ တိုက်ဆိုင်စစ်ပါ။

### ဥပမာ

```python
generation = trace.generation(
    name="llm-answer",
    model="gpt-4o-mini",
    input=[
        {"role": "system", "content": "You are a helpful support agent."},
        {"role": "user", "content": "How do I reset my password?"},
    ],
    output="Go to Settings > Security and click Reset Password.",
    usage={"input": 25, "output": 14, "total": 39},
)
print(generation.name)
# Expected output: llm-answer
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

မကြာခဏ မှားတတ်တဲ့ prompt pattern တွေ၊ context ရှည်လို့ကုန်တဲ့ token တွေ၊ စသည်တွေကို မှတ်တမ်းရှိမှ အချက်အလက်နဲ့ ပြင်နိုင်ပါတယ်။

## ၄။ Feature အလိုက် Tagging

### ဘာကို ဆိုလိုတာလဲ

Trace တွေကို feature, version, environment စသည်အလိုက် `tags` တပ်ပြီး ခွဲခြားနည်း ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ

App တစ်ခုမှာ summarization, chat, extraction စသဖြင့် feature များစွာ ပါတတ်ပါတယ်။ Tag မရှိရင် ဘယ် feature က ငွေကုန်အများဆုံး၊ ဘယ် feature က မှားအများဆုံးဆိုတာ ခွဲမမြင်ရပါဘူး။

### ဘယ်လို အလုပ်လုပ်လဲ

Trace ဖန်တီးတဲ့အခါ `tags=["feature:summarizer", "env:prod"]` လိုမျိုး ထည့်ပါ။ UI မှာ tag အလိုက် filter လုပ်လို့ရပါတယ်။

### ဥပမာ

```python
trace = langfuse.trace(
    name="doc-summarizer",
    tags=["feature:summarizer", "env:staging"],
    input={"doc_id": "doc-7"},
)
print(trace.get_trace_url())
# Expected output: a trace URL showing the tags in the UI
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

New prompt version deploy လုပ်ပြီးတဲ့နောက် ဒီ version ရဲ့ trace တွေကိုခွဲကြည့်ပြီး quality ရှိမရှိ အမြန် သိနိုင်ပါတယ်။

## ၅။ Trace-based Debugging

### ဘာကို ဆိုလိုတာလဲ

ဆိုးတဲ့ အဖြေတစ်ခုကို အဲဒီအဖြေ ထွက်ခဲ့တဲ့ trace ပြန်ကြည့်ပြီး — ဘယ် context သွင်းခဲ့လဲ၊ ဘယ် prompt သုံးခဲ့လဲ၊ ဘယ်ဆင့်မှာ ပြဿနာစလဲ — ရှာဖွီးတာ ဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ

အဖြေမှားတာကို "LLM ညံ့လို့" လို့ အလွယ် မကြော်ပါနဲ့။ တကယ်တော့ retrieval က မှားတဲ့ doc ကိုပဲ ရွေးခဲ့တာ ဖြစ်နိုင်ပြီး၊ trace က အဲဒီအချက်ကို ပြပေးနိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ

(၁) မှားတဲ့ trace ကိ်ု user/session id နဲ့ ရှာပါ။ (၂) span tree ကို အစဉ်လိုက်ကြည့်ပါ။ (၃) ဘယ်ဆင့်မှာ အချက်အလက် မှားသွားလဲ အတည်ပြုပါ။ (၄) အဲဒီဆင့်က code ကို ပြင်ပါ။

### ဥပမာ

```python
# Simplified pattern: record each step so debugging is possible
def answer_question(question, session_id):
    trace = langfuse.trace(
        name="qa-bot", sessionId=session_id,
        input={"question": question},
    )
    span = trace.span(name="retrieve")
    docs = retrieve(question)          # step 1: gather context
    span.end(output={"docs": docs})
    answer = call_llm(question, docs)  # step 2: ask the model
    trace.generation(
        name="llm-answer", model="gpt-4o-mini",
        input={"question": question, "docs": docs},
        output=answer,
        usage={"input": 80, "output": 20, "total": 100},
    )
    trace.update(output=answer)
    return answer

answer = answer_question("How do I reset my password?", "session-abc-123")
print(answer)
# Expected output: the bot answer; every step is now inspectable in the trace
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production issue တွေကို ခန့်မှန်းချက်နဲ့ မဟုတ်ဘဲ trace အထောက်အထားနဲ့ ပြင်ဆင်နိုင်ရင် debugging time တွေ သိသိသာသာ လျော့ပါတယ်။

## အနှစ်ချုပ်

Langfuse tracing က LLM app ရဲ့ တစ်ချက်ချင်း ပုံရိပ်ကို ပေးပါတယ် — ဘယ်ဆင့်၊ ဘယ် prompt၊ ဘယ် cost၊ ဘယ် user၊ ဘယ် feature။ ဒီ data ရှိမှ နောက်ပိုင်း evaluation နဲ့ monitoring တွေ အဓိပ္ပာယ်ရှိပါတယ်။
