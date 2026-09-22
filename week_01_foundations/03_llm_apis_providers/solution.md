## လေ့ကျင့်ခန်း ၁ — OpenAI-compatible Chat Completions ဖန်ရှင်ခေါ်ဆိုမှု

```python
import os
import requests

# OpenAI-compatible endpoint: works for OpenAI, LM Studio, vLLM, etc.
BASE_URL = "https://api.openai.com/v1/chat/completions"
API_KEY = os.environ.get("OPENAI_API_KEY", "")

payload = {
    "model": "gpt-4o-mini",
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain what a token is in one sentence."}
    ],
    "temperature": 0.7,
    "max_tokens": 200,
}

headers = {"Authorization": f"Bearer {API_KEY}"}
response = requests.post(BASE_URL, json=payload, headers=headers, timeout=30)
response.raise_for_status()

data = response.json()
print(data["choices"][0]["message"]["content"])
```

**အဓိကအယူအဆ** — Chat Completions API သည် `messages` စာရင်းတွင် `system`, `user`, `assistant` role များဖြင့် စကားဝိုင်းကို သတ်မှတ်ပုံစံဖြင့် ပေးပို့ပြီး provider အများအပြားက ၎င်းပုံစံကို တူညီစွာ လက်ခံကြသည်။

## လေ့ကျင့်ခန်း ၂ — Streaming နှင့် Non-streaming နှိုင်းယှဉ်ခြင်း

```python
import os
import json
import requests

BASE_URL = "https://api.openai.com/v1/chat/completions"
API_KEY = os.environ.get("OPENAI_API_KEY", "")
headers = {"Authorization": f"Bearer {API_KEY}"}

payload = {
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Write a short paragraph about rain."}],
    "stream": True,
}

# Streaming: server sends chunks of "text/event-stream" data
with requests.post(
    BASE_URL, json=payload, headers=headers, stream=True, timeout=60
) as response:
    response.raise_for_status()
    for line in response.iter_lines(decode_unicode=True):
        if not line or not line.startswith("data: "):
            continue
        body = line[len("data: "):]
        if body.strip() == "[DONE]":
            break
        chunk = json.loads(body)
        delta = chunk["choices"][0]["delta"].get("content", "")
        print(delta, end="", flush=True)
print()
```

**အဓိကအယူအဆ** — Streaming ဖြင့် server-sent events အနေဖြင့် စကားလုံးတစ်ခုချင်း ရောက်လာမှုကို အသုံးပြုသူ ချက်ချင်းမြင်ရစေပြီး၊ `[DONE]` sentinel ရောက်လာမှသာ စာကြောင်းပြီးဆုံးကြောင်း သိနိုင်သည်။

## လေ့ကျင့်ခန်း ၃ — Timeout နှင့် Retry/Backoff ဆော့ဖ်ဝဲလ်ရေးခြင်း

```python
import os
import time
import requests

BASE_URL = "https://api.openai.com/v1/chat/completions"
API_KEY = os.environ.get("OPENAI_API_KEY", "")
headers = {"Authorization": f"Bearer {API_KEY}"}

def chat_completion(prompt, max_retries=4):
    payload = {
        "model": "gpt-4o-mini",
        "messages": [{"role": "user", "content": prompt}],
    }
    for attempt in range(max_retries):
        try:
            response = requests.post(
                BASE_URL, json=payload, headers=headers, timeout=20
            )
            if response.status_code in (429, 500, 502, 503):
                # exponential backoff: 1s, 2s, 4s, 8s
                wait = 2 ** attempt
                print(f"Retrying in {wait}s after status {response.status_code}")
                time.sleep(wait)
                continue
            response.raise_for_status()
            return response.json()["choices"][0]["message"]["content"]
        except requests.Timeout:
            print("Request timed out, retrying...")
            time.sleep(2 ** attempt)
    raise RuntimeError("All retries failed")

if __name__ == "__main__":
    print(chat_completion("Say hello in Burmese."))
```

**အဓိကအယူအဆ** — Timeout သတ်မှတ်ခြင်းနှင့် retryable status code (429, 5xx) များအတွက် exponential backoff ဖြင့် ပြန်ကြိုးစားခြင်းသည် API service များ၏ ယာယီချို့ယွင်းမှုများကို ကြံ့ခိုင်စွာ ရင်ဆိုင်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၄ — Token စာရင်းစနစ်နှင့် ကုန်ကျစရိတ်တွက်ချက်ခြင်း

```python
import os
import requests

BASE_URL = "https://api.openai.com/v1/chat/completions"
API_KEY = os.environ.get("OPENAI_API_KEY", "")
headers = {"Authorization": f"Bearer {API_KEY}"}

payload = {
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Summarize the water cycle briefly."}],
}
response = requests.post(BASE_URL, json=payload, headers=headers, timeout=30)
response.raise_for_status()

usage = response.json()["usage"]
print("prompt_tokens:", usage["prompt_tokens"])
print("completion_tokens:", usage["completion_tokens"])
print("total_tokens:", usage["total_tokens"])

# Cost calculation: read real prices from your provider's pricing page.
# Example only with placeholders; replace with current published rates.
INPUT_PRICE_PER_1K = 0.00015   # USD per 1,000 prompt tokens (example)
OUTPUT_PRICE_PER_1K = 0.0006  # USD per 1,000 completion tokens (example)

cost = (
    usage["prompt_tokens"] / 1000 * INPUT_PRICE_PER_1K
    + usage["completion_tokens"] / 1000 * OUTPUT_PRICE_PER_1K
)
print(f"Estimated cost: ${cost:.6f}")
```

**အဓိကအယူအဆ** — API ၏ `usage` field သည် prompt နှင့် completion token အရေအတွက်ကို တိတိကျကျပြောပြသဖြင့် ၎င်းကို provider ၏ တရားဝင် pricing ဇယားဖြင့် မြှောက်ပါက တစ်ခေါက်ချင်းစီ၏ ကုန်ကျစရိတ်ကို တွက်ချက်နိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Ollama ဖြင့် On-prem Inference

```python
import requests

# Ollama serves an OpenAI-compatible endpoint on localhost by default
BASE_URL = "http://localhost:11434/v1/chat/completions"

payload = {
    "model": "llama3.2",  # run: ollama pull llama3.2
    "messages": [
        {"role": "user", "content": "List three benefits of on-prem inference."}
    ],
    "stream": False,
}

try:
    response = requests.post(BASE_URL, json=payload, timeout=120)
    response.raise_for_status()
    print(response.json()["choices"][0]["message"]["content"])
except requests.ConnectionError:
    print("Ollama is not running. Start it with: ollama serve")
```

**အဓိကအယူအဆ** — Ollama သည် local machine ပေါ်တွင် model များကို run ပေးပြီး OpenAI-compatible endpoint ဖြင့် ဆက်သွယ်နိုင်စေသဖြင့် cloud code ကို base URL ပြောင်းရုံဖြင့် on-prem သို့ ရွှေ့နိုင်သည်။

## လေ့ကျင့်ခန်း ၆ — စုံစွာသုံးနိုင်တဲ့ CLI Chat Client တည်ဆောက်ခြင်း

```python
import time
import json
import urllib.request
import urllib.error

API_KEY = "YOUR_API_KEY"
API_URL = "https://api.openai.com/v1/chat/completions"
MODEL = "gpt-4o-mini"

# Pricing: USD per 1K tokens (prompt / completion)
PRICE_PROMPT = 0.00015
PRICE_COMPLETION = 0.0006

MAX_RETRIES = 5
TIMEOUT = 30  # seconds


def call_api_with_backoff(messages):
    """Send a streaming chat request with exponential backoff retry on failure."""
    payload = {
        "model": MODEL,
        "messages": messages,
        "stream": True,
    }
    body = json.dumps(payload).encode("utf-8")

    for attempt in range(1, MAX_RETRIES + 1):
        try:
            req = urllib.request.Request(
                API_URL,
                data=body,
                headers={
                    "Authorization": f"Bearer {API_KEY}",
                    "Content-Type": "application/json",
                },
                method="POST",
            )
            return urllib.request.urlopen(req, timeout=TIMEOUT)
        except (urllib.error.URLError, urllib.error.HTTPError, TimeoutError) as e:
            wait = 2 ** attempt  # exponential backoff: 2, 4, 8, 16, 32
            print(f"\n[Connection error: {e}. Retrying in {wait}s "
                  f"(attempt {attempt}/{MAX_RETRIES})]")
            time.sleep(wait)
    raise RuntimeError("Max retries exceeded; server unavailable.")


def stream_response(messages):
    """Stream the assistant reply and return (full_text, usage_dict)."""
    resp = call_api_with_backoff(messages)
    chunks = []
    usage = None

    # Read the SSE stream line by line
    for raw_line in resp:
        line = raw_line.decode("utf-8").strip()
        if not line.startswith("data: "):
            continue
        data = line[len("data: "):]
        if data == "[DONE]":
            break
        obj = json.loads(data)
        if obj.get("usage"):
            usage = obj["usage"]
        for choice in obj.get("choices", []):
            delta = choice.get("delta", {}).get("content")
            if delta:
                chunks.append(delta)
                print(delta, end="", flush=True)

    print()  # newline after streamed text
    text = "".join(chunks)
    return text, usage


def estimate_tokens(text):
    """Rough token estimate: ~1 token per 4 characters."""
    return max(1, len(text) // 4)


def main():
    messages = [
        {"role": "system", "content": "You are a helpful assistant."}
    ]
    total_prompt_tokens = 0
    total_completion_tokens = 0

    print("CLI Chat Client — type 'quit' to exit.\n")

    while True:
        try:
            user_input = input("You: ").strip()
        except (EOFError, KeyboardInterrupt):
            break
        if not user_input:
            continue
        if user_input.lower() == "quit":
            break

        messages.append({"role": "user", "content": user_input})

        print("Assistant: ", end="")
        try:
            reply, usage = stream_response(messages)
        except RuntimeError as e:
            print(f"[Failed: {e}]")
            messages.pop()  # drop the failed turn from history
            continue

        # Keep multi-turn context: store the assistant reply in history
        messages.append({"role": "assistant", "content": reply})

        # Collect usage; fall back to estimation if streaming gave none
        if usage:
            total_prompt_tokens += usage.get("prompt_tokens", 0)
            total_completion_tokens += usage.get("completion_tokens", 0)
        else:
            est_prompt = estimate_tokens(user_input)
            est_completion = estimate_tokens(reply)
            total_prompt_tokens += est_prompt
            total_completion_tokens += est_completion
        print()

    total_tokens = total_prompt_tokens + total_completion_tokens
    cost = (total_prompt_tokens / 1000 * PRICE_PROMPT
            + total_completion_tokens / 1000 * PRICE_COMPLETION)

    print("\n===== Session Summary =====")
    print(f"Prompt tokens:     {total_prompt_tokens}")
    print(f"Completion tokens: {total_completion_tokens}")
    print(f"Total tokens:      {total_tokens}")
    print(f"Estimated cost:    ${cost:.4f}")


if __name__ == "__main__":
    main()
```

**အဓိကအယူအဆ** — Streaming output၊ timeout နဲ့ exponential backoff retry တို့ကို ပေါင်းစပ်ထားပြီး conversation history ထဲမှာ assistant အဖြေကို တစ်ဝိုင်းချင်း ထည့်ထားမှသာ multi-turn context ဆက်နိုင်ပြီး စကားဝိုင်းအဆုံးမှာ token usage နဲ့ estimated cost summary ပြသနိုင်သည်။
