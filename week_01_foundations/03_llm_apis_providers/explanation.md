# Week 1 — LLM APIs & Providers (hosted + on-prem)

## OpenAI-compatible Chat Completions shape

### ဘာကို ဆိုလိုတာလဲ
Chat Completions API ဆိုသည်မှာ model တစ်ခုဆီသို့ message စာရင်း (system, user, assistant) ပို့လိုက်ပြီး အဖြေ (completion) တစ်ခု ပြန်ရယူသော standard API format ဖြစ်သည်။ OpenAI က စတင်သတ်မှတ်ခဲ့ပြီး ယနေ့တွင် provider များစွာက ထို format ကို အတုခိုး (compatible) အသုံးပြုကြသည်။

### ဘာကြောင့် လဲ
Format တစ်ခုတည်းဖြင့် provider များစွာကို ပြောင်းလဲ အသုံးပြုနိုင်သောကြောင့် ဖြစ်သည်။ Code ထဲရှိ base URL နှင့် API key ကိုသာ ပြောင်းလိုက်လျှင် provider အများအုပ်စုနှင့် အလုပ်လုပ်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
HTTP POST request တစ်ခုကို `/v1/chat/completions` endpoint သို့ ပို့သည်။ Body ထဲတွင် `model`, `messages`, `temperature` စသည့် field များ ပါဝင်သည်။ Response တွင် `choices[0].message.content` အနေဖြင့် အဖြေကို ရရှိသည်။

### ဥပမာ

```python
from openai import OpenAI

client = OpenAI(api_key="YOUR_API_KEY")

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain what an API is in one sentence."},
    ],
    temperature=0.7,
)

print(response.choices[0].message.content)
# Expected output:
# An API is a set of rules that lets one software application talk to another.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production system တစ်ခုတွင် provider တစ်ခုတည်းကိုသာ မှီခိုနေခြင်းသည် အန္တရာယ်ရှိသည်။ OpenAI-compatible format ကို နားလည်ထားလျှင် vLLM, Together AI, Groq ကဲ့သို့သော provider များဆီသို့ လွယ်ကူစွာ ကူးပြောင်းနိုင်သည်။

## Streaming vs non-streaming

### ဘာကို ဆိုလိုတာလဲ
Streaming ဆိုသည်မှာ model ၏ အဖြေကို token အပိုင်းလိုက် တစ်ဆက်တည်း စီးဝင်လာအောင် လက်ခံယူခြင်းဖြစ်ပြီး၊ non-streaming က အဖြေတစ်ခုလုံး ပြီးသွားမှ တစ်ကြိမ်တည်း ရယူခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
အဖြေရှည်များအတွက် non-streaming ဖြင့် စောင့်ဆိုင်းရသည်မှာ စိတ်ညစ်ဖွယ်ဖြစ်သည်။ Streaming ဖြင့် user သည် စာကို တဖြည်းဖြည်း ရေးနေသလို မြင်ရသောကြောင့် user experience ပိုကောင်းသည်။

### ဘယ်လို အလုပ်လုပ်လဲ
API request တွင် `stream=True` ထားလိုက်လျှင် server က Server-Sent Events (SSE) အနေဖြင့် chunk များ တစ်ဆက်တည်း ပို့သည်။ Client က chunk တစ်ခုချင်းစီကို လက်ခံကာ စုစည်းပြသည်။

### ဥပမာ

```python
from openai import OpenAI

client = OpenAI(api_key="YOUR_API_KEY")

stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Count from 1 to 5, one number per line."}],
    stream=True,
)

full_text = ""
for chunk in stream:
    delta = chunk.choices[0].delta
    if delta.content:
        print(delta.content, end="", flush=True)
        full_text += delta.content
print()
# Expected output:
# 1
# 2
# 3
# 4
# 5
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Chatbot application တိုင်းလိုလိုတွင် streaming ကို အသုံးပြုကြသည်။ သို့သော် streaming သည် error handling နှင့် token accounting ကို ရှုပ်ထွေးစေသောကြောင့် non-streaming ထက် သတိထားဆောင်ရွက်ရသည်။

## Retries, backoff နှင့် timeouts

### ဘာကို ဆိုလိုတာလဲ
Retry ဆိုသည်မှာ request တစ်ခု ကျွမ်းသွားလျှင် ထပ်မံ ကြိုးပမ်းခြင်းဖြစ်သည်။ Backoff ဆိုသည်မှာ retry အကြိမ်တိုင်းတွင် စောင့်ချိန်ကို တဖြည်းဖြည်း တိုးလာအောင် လုပ်ခြင်းဖြစ်သည်။ Timeout က request တစ်ခုအတွက် စောင့်ဆိုင်းမည့် အချိန်အတတ် ကန့်သတ်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ
Network ချို့ယွင်းမှု၊ rate limit (HTTP 429)၊ server error (HTTP 5xx) တို့သည် production တွင် မဖြစ်မနေ ကြုံတွေ့ရမည့် အခြေအနေများဖြစ်သည်။ ယင်းတို့အတွက် ပြင်ဆင်မထားပါက system တစ်ခုလုံး ရပ်တန့်သွားနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
Retry ကို exponential backoff ဖြင့် လုပ်သည် — ဥပမာ 1 စက္ကန့်၊ 2 စက္ကန့်၊ 4 စက္ကန့် စသည်ဖြင့် နှစ်ဆတိုးသည်။ ရှောင်ရမည့် error (ဥပမာ 400 authentication error) ကို retry လုပ်လျှင် အကျိုးမရှိပါ။ OpenAI Python SDK တွင် built-in retry ပါဝင်သည်။

### ဥပမာ

```python
import time
from openai import OpenAI

client = OpenAI(api_key="YOUR_API_KEY", timeout=30.0, max_retries=3)

def ask_with_backoff(prompt, max_attempts=4):
    for attempt in range(1, max_attempts + 1):
        try:
            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[{"role": "user", "content": prompt}],
            )
            return response.choices[0].message.content
        except Exception as e:
            if attempt == max_attempts:
                raise
            wait = 2 ** attempt
            print(f"Attempt {attempt} failed: {e}. Retrying in {wait}s...")
            time.sleep(wait)

print(ask_with_backoff("Say hello."))
# Expected output:
# Hello! How can I help you today?
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Production system တွင် LLM API call က အားနည်းရှုံးနိုင်သော အချက်တစ်ခုဖြစ်သည်။ Timeout မထားပါက request များ ကြာမြင့်စွာ ဆိုင်းနေကာ ဆက်သွယ်မှု thread များ ပိတ်သွားနိုင်သည်။ ထို့ကြောင့် retries + backoff + timeout သုံးခုလုံး မဖြစ်မနေ ထည့်သွင်းရမည်။

## Token accounting နှင့် cost

### ဘာကို ဆိုလိုတာလဲ
Token ဆိုသည်မှာ model ဖတ်ရှုသော စာသား၏ အပိုင်းအစ သေးသေးလေးများဖြစ်သည်။ API အမှန်တရားအားဖြင့် character မဟုတ်ဘဲ token အလိုက် အသုံးပြုသူကို ကုန်ကျစရိတ် တွက်ချက်သည်။

### ဘာကြောင့် လဲ
Model တစ်ခုချင်းစီတွင် context window အကန့်အသတ်နှင့် input/output token ဈေးနှုန်း မတူညီသောကြောင့် ဖြစ်သည်။ Token အရေအတွက်ကို မသိပါက ကုန်ကျစရိတ်ကို ခန့်မှန်းလို့ မရပါ။

### ဘယ်လို အလုပ်လုပ်လဲ
Response object ထဲရှိ `response.usage` field တွင် `prompt_tokens` (input), `completion_tokens` (output), `total_tokens` တို့ ပါဝင်သည်။ ကုန်ကျစရိတ် = input token × input ဈေး + output token × output ဈေး ဖြစ်သည်။ ဈေးနှုန်းများကို provider ၏ pricing page တွင် ကြည့်ရမည်။

### ဥပမာ

```python
from openai import OpenAI

client = OpenAI(api_key="YOUR_API_KEY")

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello, how are you?"}],
)

usage = response.usage
print(f"Input tokens:  {usage.prompt_tokens}")
print(f"Output tokens: {usage.completion_tokens}")
print(f"Total tokens:  {usage.total_tokens}")
# Expected output:
# Input tokens:  13
# Output tokens:  10
# Total tokens:  23
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Application တစ်ခုတွင် API call သန်းပေါင်းများ ဖြစ်လာသောအခါ token အသေးစိတ် ကွာခြားချက်များသည် ကုန်ကျစရိတ် ကြီးမားစွာ ကွာခြားစေသည်။ Token usage ကို log တင်ခြင်းဖြင့် ဘတ်ဂျက် ထိန်းချုပ်နိုင်ပြီး prompt ရှည်လွန်းမှုကိုပါ တွေ့ရှိနိုင်သည်။ (မှတ်ချက် — အထက်ပါ token အရေအတွက်သည် စာသားအပေါ် မူတည်ပါသည်။)

## Ollama for on-prem inference

### ဘာကို ဆိုလိုတာလဲ
Ollama ဆိုသည်မှာ ကိုယ်ပိုင် computer သို့မဟုတ် server (on-prem) ပေါ်တွင် open-weight LLM များ (ဥပမာ Llama, Gemma, Mistral) ကို အလွယ်တကူ လည်ပတ်စေသော tool ဖြစ်သည်။

### ဘာကြောင့် လဲ
Data ကို အခြားသူ့ server သို့ မပို့လိုသော အဖွဲ့အစည်းများအတွက် on-prem inference သည် လုံခြုံမှုနှင့် privacy အတွက် အရေးကြီးသည်။ ထို့အပြင် per-token ကုန်ကျစရိတ် မရှိသောကြောင့် အသုံးပြုမှု များလာသောအခါ စျေးသက်သာနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ
`ollama pull llama3.2` ကဲ့သို့ command ဖြင့် model ကို ဒေါင်းလုဒ်လုပ်ပြီး Ollama သည် local HTTP server တစ်ခု (`http://localhost:11434`) ကို ဖွင့်ပေးသည်။ ၎င်းသည် OpenAI-compatible endpoint ပါ ပံ့ပိုးသောကြောင့် OpenAI SDK ဖြင့်ပင် ချိတ်ဆက်နိုင်သည်။

### ဥပမာ

```python
from openai import OpenAI

# Ollama exposes an OpenAI-compatible endpoint on localhost
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

response = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "Hello, who are you?"}],
)
print(response.choices[0].message.content)
# Expected output:
# Hi! I'm LLaMA, an AI assistant developed by Meta AI.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Hosted API နှင့် on-prem Ollama ကို code တစ်ခုတည်းဖြင့် ပြောင်းသုံးနိုင်ခြင်းသည် architecture ကို စိတ်ချလက်ချ ပြောင်းလဲနိုင်စေသည် — development တွင် local model သုံးကာ production တွင် hosted model သို့မဟုတ် ၎င်း၏ပြောင်းပြန် ရွေးချယ်နိုင်သည်။ သို့သော် on-prem တွင် hardware (GPU/RAM) လိုအပ်ချက်နှင့် model quality ကွာဟချက်များကို တွက်ချေမှတ်ရမည်။

## အနှစ်ချုပ်

- **Chat Completions format** သည် provider များစွာက လက်ခံသော de facto standard ဖြစ်ပြီး base URL ပြောင်းရုံဖြင့် provider ပြောင်းနိုင်သည်။
- **Streaming** (`stream=True`) ဖြင့် user experience တိုးတက်စေပြီး၊ **non-streaming** က error handling နှင့် token တွက်ချက်မှု ပိုလွယ်သည်။
- **Retries + exponential backoff + timeout** သုံးခုစလုံး production တွင် မဖြစ်မနေ ထည့်သွင်းရမည်။
- **Token usage** (`response.usage`) ကို log တင်ခြင်းဖြင့် ကုန်ကျစရိတ် ထိန်းချုပ်နိုင်သည်။
- **Ollama** ဖြင့် OpenAI-compatible on-prem inference လုပ်နိုင်ပြီး privacy လိုအပ်ချက်ရှိပါက ရွေးချယ်စရာ ကောင်းတစ်ခု ဖြစ်သည်။
