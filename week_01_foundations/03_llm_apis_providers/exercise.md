## လေ့ကျင့်ခန်း ၁ — OpenAI-compatible Chat Completions ခေါ်ဆိုမှု အခြေခံ

Ollama ကို local မှာ ထားပါ (default: `http://localhost:11434`)။ Python `requests` ဖြင့် `/v1/chat/completions` endpoint ကို ခေါ်ပြီး မေးခွန်းတစ်ခု အဖြေရအောင် ရေးပါ။ OpenAI Python SDK ကို `base_url` ပြောင်း၍ လည်း စမ်းကြည့်ပါ — request shape တူညီကြောင်း တွေ့ရမည်။

```python
import requests

resp = requests.post(
    "http://localhost:11434/v1/chat/completions",
    json={
        "model": "llama3.2",
        "messages": [{"role": "user", "content": "Explain what an API is in one sentence."}],
    },
)
print(resp.json()["choices"][0]["message"]["content"])
```

**Hints:** Ollama ရဲ့ `/v1/*` endpoints များက OpenAI API နဲ့ compatible ဖြစ်သည်။ Model ကို `ollama pull llama3.2` ဖြင့် အရင် download လုပ်ပါ။
**Expected behavior:** Terminal တွင် model ရဲ့ one-sentence answer ကို print ထွက်သည် — OpenAI SDK နဲ့ requests နှစ်ခုလုံးမှာ တူညီတဲ့ response structure ရသည်။

## လေ့ကျင့်ခန်း ၂ — Streaming နှင့် Non-streaming နှိုင်းယှဉ်ခြင်း

အလေ့ကျင့်ခန်း ၁ ကို အခြေခံပြီး streaming (`"stream": true`) နဲ့ non-streaming ကို သီးသန့် script နှစ်ခု ရေးပါ။ Streaming မှာ server-sent events (SSE) လိုင်းတိုင်းကို parse ပြီး delta content ကို တစ်လုံးချင်း print ပါ။ Delta တစ်ခုစီက `choices[0]["delta"]["content"]` ထဲ ရှိသည်။

**Hints:** SSE lines များသည် `data: ` နဲ့ စသည်။ Stream ပြီးတဲ့အခါ `data: [DONE]` လိုင်း ရောက်လာမည်။ `requests` မှာ `stream=True` သုံးပြီး `iter_lines()` နဲ့ loop လုပ်ပါ။
**Expected behavior:** Non-streaming မှာ အဖြေတစ်ခုတည်း အတုံ့ပြန်ပေါ်သည်။ Streaming မှာ token များ တစ်လုံးပြီးတစ်လုံး terminal ထဲ တဖြည်းဖြည်း စီးဝင်သလို မြင်ရသည်။

## လေ့ကျင့်ခန်း ၃ — Timeout ထည့်ခြင်း

`requests.post()` မှာ `timeout` parameter (ဥပမာ 5 စက္ကန့်) ထည့်ပြီး၊ connect timeout နဲ့ read timeout နှစ်မျိုးလုံး tuple (`(3, 5)`) အနေနဲ့ သုံးကြည့်ပါ။ Timeout ဖြစ်သွားပါက `requests.exceptions.Timeout` ကို catch လုပ်၍ ရှင်းလင်းတဲ့ error message ထုတ်ပါ။

**Hints:** `(connect_timeout, read_timeout)` ဟု tuple သုံးသည် — requests တရားဝင် docs တွင်ဖော်ပြထားသည်။ Local Ollama မှာ timeout မကြာမီပြီးသွားပုံမရှိလျှင် အလွန်ရှည် prompt သုံး၍ စမ်းနိုင်သည်။
**Expected behavior:** Server အမြန်ပြန်ပါက ပုံမှန်အဖြေရသည်။ Server နှေးလျှင် script က သတ်မှတ်ချိန်အတွင်း `Timeout` error ကို ဖမ်း၍ မှန်ကန်တဲ့ message ထုတ်ပြသည်။

## လေ့ကျင့်ခန်း ၄ — Retry နှင့် Exponential Backoff

Server error (HTTP 5xx) သို့မဟုတ် connection error ဖြစ်ပါက ဒီ retry logic ပါတဲ့ wrapper function `chat_with_retry()` ရေးပါ — အကြိမ် 3 ခြားပြီး sleep ချိန်များက `1s, 2s, 4s` အတိုင်း တစ်ဆီနှစ်ဆီတိုးသည်။

```python
import time

def chat_with_retry(payload, max_retries=3):
    for attempt in range(max_retries):
        try:
            resp = requests.post(URL, json=payload, timeout=10)
            if resp.status_code >= 500:
                raise RuntimeError(f"server error {resp.status_code}")
            resp.raise_for_status()
            return resp.json()
        except (RuntimeError, requests.exceptions.RequestException) as e:
            if attempt == max_retries - 1:
                raise
            wait = 2 ** attempt  # 1, 2, 4 seconds
            print(f"Attempt {attempt + 1} failed ({e}), retrying in {wait}s")
            time.sleep(wait)
```

**Hints:** `4xx` client errors (ဥပမာ 400) က retry လုပ်ဖို့ မသင့် — payload မှားနေသောကြောင့်ဖြစ်သည်။ `5xx` နဲ့ network errors မှသာ retry သင့်သည်။
**Expected behavior:** Server တစ်ခါနှစ်ခါ ပျက်ပြီး တတိယအကြိမ်မှာ အောင်မြင်လျှင် အဖြေရပြီး၊ ၃ ကြိမ်စလုံး ပျက်လျှင် နောက်ဆုံး error ကို ပြန် raise လုပ်သည်။

## လေ့ကျင့်ခန်း ၅ — Token Accounting နှင့် ကုန်ကျစားရိတ် တွက်ခြင်း

Non-streaming response မှာ `usage` field (`prompt_tokens`, `completion_tokens`, `total_tokens`) ရှိသည်။ ဤတန်ဖိုးများကို extract လုပ်ပြီး အောက်ပါအတိုင်း ကုန်ကျစားရိတ် တွက်ပေးမည့် `estimate_cost()` function ရေးပါ — price တွက် `per_1k_prompt` နဲ့ `per_1k_completion` ကို argument အနေနဲ့ လက်ခံမည်။

```python
def estimate_cost(usage, per_1k_prompt, per_1k_completion):
    # usage is the "usage" dict from a chat completion response
    prompt_cost = usage["prompt_tokens"] / 1000 * per_1k_prompt
    completion_cost = usage["completion_tokens"] / 1000 * per_1k_completion
    return prompt_cost + completion_cost
```

**Hints:** Streaming မှာ `usage` သည် default အားဖြင့် မပါဘဲ stream options ထည့်မှ သာ ရနိုင်သည် (provider အလိုက် ကွာခြားသည်)။ Price တန်ဖိုးများကို မှတ်သားထားပါ — ဒီ function က သင် ထည့်သွင်းတဲ့ price အတိုင်းပဲ တွက်ပေးသည်။
**Expected behavior:** Response တစ်ခုလုံးအတွက် token counts ၃ မျိုး ထုတ်ပြပြီး၊ ပေးထားတဲ့ price များနဲ့ တွက်ချက်ထားသော ကုန်ကျစားရိတ် ဂဏန်းတစ်ခု ရသည်။

## လေ့ကျင့်ခန်း ၆ — စုံစွာသုံးနိုင်တဲ့ CLI Chat Client တည်ဆောက်ခြင်း

အထက်ပါ အားလုံးကို ပေါင်းစပ်ပြီး CLI chat client အပြည့်အစုံ ရေးပါ — streaming output၊ timeout၊ exponential backoff retry၊ နဲ့ စကားဝိုင်းတစ်ခုဆုံးရင် token usage summary နဲ့ estimated cost ပြခြင်းတို့ ပါဝင်မည်။ Conversation history (`messages` list) ကို ဆက်လက်ထိန်းသိမ်း၍ multi-turn chat ဖြစ်အောင် လုပ်ပါ။ `quit` ရိုက်လျှင် ထွက်သည်။

**Hints:** History ထဲ မှာ `{"role": "assistant", "content": ...}` ကို တစ်ဝိုင်းပြီးတိုင်း ထည့်ပေးရမည် — မဟုတ်လျှင် model က context မဆက်နိုင်ပါ။ Streaming response ကနေ usage မရလျှင် `total_tokens` ကို prompt နဲ့ completion သီးသန့် ရေတွက်၍ ဖြည့်နိုင်သည်။
**Expected behavior:** အသုံးပြုသူ ရိုက်တဲ့ စကားတစ်ခုချင်းစီအတွက် streaming အဖြေ ထွက်လာသည်၊ server ပျက်လျှင် အလိုအလျောက် retry လုပ်သည်၊ program ထွက်ခါနီးမှာ တစ်ခါလုံးရဲ့ total token နဲ့ ကုန်ကျစားရိတ် summary ပြသည်။
