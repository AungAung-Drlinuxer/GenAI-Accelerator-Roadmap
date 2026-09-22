# Context Engineering အသေးစိတ်ရှင်းလင်းချက်

## ၁။ Context Window ထဲမှာ ဘာတွေ ပါဝင်သလဲ

### ဘာကို ဆိုလိုတာလဲ
Context window ဆိုတာ model တစ်ခုက တစ်ကြိမ်တည်းနဲ့ ဖတ်နိုင်တဲ့ token အရေအတွက် နယ်နိမိတ်ပါ။ ဒီ window ထဲမှာ system prompt၊ user message၊ assistant ၏ ယခင်အဖြေတွေ၊ tool output နဲ့ ကျွန်ုပ်တို့ ထည့်သွင်းတဲ့ document တွေ အားလုံး ပါဝင်ပါတယ်။

### ဘာကြောင့် လဲ
LLM ဟာ window အပြင်က အချက်အလက်ကို မမြင်နိုင်ပါ။ ဒါကြောင့် model ရဲ့ အဖြေ အရည်အသွေးဟာ ဘာကို window ထဲ ထည့်လိုက်လဲ ဆိုတဲ့ ရွေးချယ်မှုပေါ်မှာ အဓိက မူတည်ပါတယ်။ ဒါကို "context engineering" လို့ ခေါ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Token တစ်ခုဆိုတာ စကားလုံးတစ်ခုရဲ့ အစိတ်အပိုင်းဖြစ်ပြီး၊ မြန်မာစာ အပါအဝင် ဘာသာစကား အများစုအတွက် စာလုံးတစ်လုံးကို token တစ်ခုထက် ပိုနိုင်ပါတယ်။ အင်္ဂလိပ်စာမှာ စကားလုံး ၄ ခုခန့်က token ၃ ခုလောက်နဲ့ ညီပါတယ်။ tiktoken ဆိုတဲ့ open-source library နဲ့ token အရေအတွက်ကို တိကျစွာ တွက်နိုင်ပါတယ်။

### ဥပမာ
```python
# Install with: pip install tiktoken
import tiktoken

# Load the encoding used by modern OpenAI models
encoding = tiktoken.get_encoding("o200k_base")

# Count tokens for a short string
text = "The customer asked for a refund."
tokens = encoding.encode(text)
print(len(tokens))

# Expected output: 7
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
API error များကို ရှောင်ရန်နဲ့ ကုန်ကျစရိတ် ကြိုတွက်ရန်အတွက် token count ကို မှန်းနိုင်ရင် စနစ်တကျ design လုပ်နိုင်ပါတယ်။ Input token တွေကို တစ်ခါ request တိုင်း ပေးရတဲ့အတွက် ကုန်ကျစရိတ် တိုက်ရိုက် တွန့်ဆန့်ဖြစ်စေပါတယ်။

## ၂။ Stuffing vs Retrieval

### ဘာကို ဆိုလိုတာလဲ
"Stuffing" ဆိုတာ document တွေအားလုံးကို context window ထဲ တန်းစောင်း ထည့်တဲ့နည်းပါ။ "Retrieval" (RAG ဟုလည်း ခေါ်သည်) ဆိုတာ မေးခွန်းနဲ့ ဆက်စပ်တဲ့ အပိုင်းလေးတွေကိုသာ ရွေးပြီး ထည့်တဲ့နည်းပါ။

### ဘာကြောင့် လဲ
Document တွေ ကြီးလာတဲ့အခါ stuffing က window ကို ကျော်သွားစေပြီး၊ ရှိသမျျ ထည့်လိုက်တာက model ကို ဆီလျော်တဲ့ အချက်အလက်ကို ရှာရခက်အောင် လုပ်တတ်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Retrieval မှာ document တွေကို အပိုင်းလေးများ (chunk) ခွဲပြီး၊ မေးခွန်းနဲ့ ဆင်တူတဲ့ chunk များကိုသာ ရွေးပြီး prompt ထဲ ထည့်ပါတယ်။ ရိုးရိုးစပ်စပ် (keyword) matching နဲ့ စတင်လို့ရပြီး၊ နောက်ပိုင်းမှာ embedding နည်းသို့ တိုးတက်နိုင်ပါတယ်။

### ဥပမာ
```python
# Simple keyword-based retrieval over small chunks
documents = [
    "Our refund policy allows returns within 30 days.",
    "Shipping to Myanmar takes 5 to 10 business days.",
    "Premium members receive free shipping on all orders.",
]
query = "How long does shipping take?"

def retrieve(query, docs, top_k=2):
    # Rank documents by count of overlapping query words
    query_words = set(query.lower().split())
    scored = [
        (sum(w in d.lower() for w in query_words), d) for d in docs
    ]
    # Sort by score descending, keep the top_k results
    scored.sort(key=lambda pair: pair[0], reverse=True)
    return [d for score, d in scored[:top_k] if score > 0]

print(retrieve(query, documents))
# Expected output: ['Shipping to Myanmar takes 5 to 10 business days.', 'Premium members receive free shipping on all orders.']
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Knowledge base က ကြီးလာတဲ့အခါ retrieval က cost ကို သိသိသာသာ လျှော့ပေးပြီး အဖြေ တိကျမှုလည်း တိုးစေပါတယ်။ Model က မလိုအပ်တဲ့ အချက်အလက်တွေကို ဖြတ်သွားရမှာကြောင့် ရှုပ်ထွေးမှုလည်း လျှော့ပါတယ်။

## ၃။ Truncation Honesty

### ဘာကို ဆိုလိုတာလဲ
Context ရှည်ကြီးကို ဖြတ်တောက်ရင် ဖြတ်လိုက်တယ်ဆိုတာကို user (သို့) model ကိုယ်တိုင်း ပွင့်ပွင့်လင်းလင်း ပြောပြရမယ်လို့ ဆိုလိုပါတယ်။ ဥပမာ — "[ဤစကားဝှက်ကို အလယ်မှာ ဖြတ်တောက်ထားပါသည်]" ဆိုတဲ့ မှတ်ချက် ထည့်ခြင်းဖြစ်ပါတယ်။

### ဘာကြောင့် လဲ
ဘာမှမပြောဘဲ ဖြတ်လိုက်ရင် model ဟာ ပြတ်တဲ့ အလယ်မှာ အရေးကြီးတဲ့ အချက်အလက် ရှိမရှိ မသိနိုင်ပါ။ User ဘက်မှာလည်း model က အားလုံးကို ဖတ်မိတယ်လို့ မှားထင်စေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
ဖြတ်တောက်ခြင်း လုပ်တဲ့အခါ (၁) ဘယ်အပိုင်းကို ဖြတ်လိုက်လဲ၊ (၂) ဖြတ်တောက်မှု ရှိခဲ့ကြောင်း သတင်းအချက်အလက် ထည့်ပေးရပါတယ်။

### ဥပမာ
```python
def truncate_honestly(text, max_chars=200):
    # Return unchanged if the text already fits
    if len(text) <= max_chars:
        return text
    # Keep the head and tail, mark the cut in the middle
    keep = max_chars // 2
    middle = "\n[...TRUNCATED: middle section removed for length...]\n"
    return text[:keep] + middle + text[-keep:]

long_text = "A" * 300
result = truncate_honestly(long_text)
print(len(result) < 320)
# Expected output: True
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
Chatbot တွေမှာ chat history ကို ဖြတ်ရတဲ့အခါ များတယ်။ ပွင့်ပွင့်လင်းလင်း မှတ်ချက်ထည့်ရင် model က "အရင်စာပိုဒ်က ဘာကြောင့် ဆက်မနေတာလဲ" ဆိုတာကို နားလည်ပြီး မှားတဲ့ ယူဆချက် လုပ်လေ့ မရှိပါ။

## ၄။ Summarisation နဲ့ Context ကျဉ်အောင် လုပ်နည်း

### ဘာကို ဆိုလိုတာလဲ
မက်ဆေ့ဂ်ရှည်တွေ (သို့) စကားဝှက်ဟောင်းတွေကို အနှစ်ချုပ် အဖြေသစ်နဲ့ အစားထိုးတဲ့နည်းပါ။ Context window ကို ကျဉ်အောင် လုပ်ပေးပါတယ်။

### ဘာကြောင့် လဲ
Chat ရှည်တဲ့ စကားဝှက်တွေမှာ token အရေအတွက်က တဖြည်းဖြည်း တိုးလာပြီး ကုန်ကျစရိတ် မြင့်လာပါတယ်။ ဟောင်းတဲ့ အပိုင်းတွေကို အနှစ်ချုပ်ထားရင် အဓိက အချက်အလက်တွေ မြင်နေဆဲဖြစ်စေပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
ဆုံးရှုံးမှု နည်းအောင် ရှေ့ဆုံးက မက်ဆေ့ဂ် အနည်းငယ်ကို အတုံးလိုက် ထားပြီး၊ အနီးကစပ် မက်ဆေ့ဂ်တွေကို အပြည့် ထားတဲ့ "keep recent + summarise old" နည်းကို သုံးပါတယ်။

### ဥပမာ
```python
def build_context_with_summary(messages, keep_recent=2):
    # If the history is short, no summary is needed
    if len(messages) <= keep_recent:
        return list(messages)
    old = messages[:-keep_recent]
    recent = messages[-keep_recent:]
    # In production, call an LLM to write this summary
    summary_text = " ".join(old)
    summary = ("Summary of earlier conversation: "
               + summary_text[:80] + "...")
    return [summary] + list(recent)

msgs = ["m1", "m2", "m3", "m4", "m5"]
print(build_context_with_summary(msgs, keep_recent=2))
# Expected output: ['Summary of earlier conversation: m1 m2 m3...', 'm4', 'm5']
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
ရှည်လျားတဲ့ chatbot session တွေမှာ ဒီနည်းက token usage ကို ကြိမ်ဆန့်လိုက် တိုးစေပြီး၊ မက်ဆေ့ဂ်ကြားထဲ အရေးကြီးတဲ့ constraint တွေ မပျောက်အောင် ဆက်ထားပေးပါတယ်။ အနှစ်ချုပ်ထဲ ဆုံးရှုံးသွားတဲ့ အချက်အလက် ရှိနိုင်တဲ့အတွက် အရေးကြီး rules တွေကို အနှစ်ချုပ်ထဲ သေချာ ထည့်ပေးရပါတယ်။

## ၅။ Prompt Caching နဲ့ Cost/Latency

### ဘာကို ဆိုလိုတာလဲ
Prompt caching ဆိုတာ prompt ရဲ့ အစပိုင်း (prefix) တူတဲ့ request နောက်ဆက်တိုင်းအတွက် provider ဘက်က cache လုပ်ထားတဲ့ အချက်အလက်ကို ပြန်သုံးခြင်းပါ။ ရလဒ်အနေနဲ့ cached input token တွေရဲစရိတ် လျှော့ပေးပြီး latency ပိုမြန်စေပါတယ်။

### ဘာကြောင့် လဲ
System prompt ကြီး၊ document တွေ၊ few-shot ဥပမာတွေ အားလုံးကို တစ်ခါ request တိုင်း အသစ်နဲ့ တွက်ရပါတယ်။ Cache လုပ်ထားရင် ဒီပိုင်းတွေကို ကုန်သက်သာစွာ ပြန်သုံးနိုင်ပါတယ်။

### ဘယ်လို အလုပ်လုပ်လဲ
Cache က prefix အတိအကျ တူဖို့ လိုပါတယ်။ ဒါကြောင့် prompt ကို ရှေ့မှာ မတည်တံ့တဲ့ အပိုင်း (မေးခွန်း၊ timestamp စသည်) နဲ့ နောက်မှာ မတည်တံ့တဲ့ အပိုင်းထည့်ပြီး ရေးရပါတယ်။ Cache တွင်တူညီမှု ရှိစေရန် prefix တစ်ခုလုံး စာလုံးတစ်လုံးချင်း တူရပါတယ်။

### ဥပမာ
```python
# Structure prompts so the stable prefix comes first
STABLE_PREFIX = (
    "You are a support agent for our e-commerce store.\n"
    "Policies:\n"
    "- Refunds within 30 days of purchase.\n"
    "- Shipping to Myanmar takes 5 to 10 business days.\n"
)

def make_prompt(user_question):
    # The changing part goes at the END so the prefix stays identical
    return STABLE_PREFIX + "Question: " + user_question + "\nAnswer:"

print(make_prompt("Can I get a refund?")[:20])
# Expected output: You are a support ag
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ
အခြေခံအားဖြင့် — input token များတာက ကုန်ကျစရိတ် တိုးစေပြီး၊ ပထမ token ရဖို့ အချိန် (latency) ပိုကြာစေပါတယ်။ Cache မှတ်ရင် cached token တွေက ကန့်ကွက်ရှင်းပြွေရေးကုန်ကျစရိတ်ထက် သိသာစွာ သက်သာပါတယ်။ ဒါကြောင့် ကြီးတဲ့ system prompt + retrieval document တွေ ပါတဲ့ system တွေမှာ caching က design ၏ တစ်စိတ်တစ်ပိုင်း ဖြစ်သင့်ပါတယ်။

## အနှစ်ချုပ်
Context engineering ဆိုတာ — ဘာကို ထည့်မလဲ၊ ဘာကို ဖြတ်မလဲ၊ ဘာကို အနှစ်ချုပ်မလဲ၊ ဘာကို cache လုပ်မလဲ ဆိုတဲ့ ရွေးချယ်မှုတွေပါ။ Token အရေအတွက်ကို တိုင်းတာတတ်ရင်၊ retrieval နဲ့ ရှင်းရှင်လင်းလင်း truncation လုပ်တတ်ရင်၊ caching-friendly prompt ဖွဲ့တတ်ရင် — ကုန်ကျစရိတ်၊ latency နဲ့ အဖြေ အရည်အသွေး သုံးခုလုံး တစ်ပြိုင်တည်း တိုးတက်စေနိုင်ပါတယ်။
