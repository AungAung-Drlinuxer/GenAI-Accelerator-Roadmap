# အဖြေများ — Context Engineering & Token Budgets

## လေ့ကျင့်ခန်း ၁ — Token counter ရေးပါ

```python
# Install with: pip install tiktoken
import tiktoken

_encoding = tiktoken.get_encoding("o200k_base")

def count_tokens(text):
    # Encode the text and return the number of tokens
    return len(_encoding.encode(text))

# Test with English and Burmese text
english = "The customer asked for a refund."
burmese = "ဖောက်သည်က ပြန်အမ်းငွေ တောင်းဆိုခဲ့ပါသည်။"  # intentional: Burmese sample data

print(count_tokens(english))
print(count_tokens(burmese))
# Expected output: 7
# Expected output: (a number, typically larger than English because
# non-ASCII scripts often use more tokens per character)
```

**အဓိကအယူအဆ** — Token အရေအတွက်က စာလုံးအရေအတွက်နဲ့ တန်းတည်တူ မဟုတ်တဲ့အတွက် တွက်ချက်စက်နဲ့ တိုင်းတာမှသာ တိကျတဲ့ budget စီမံနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Context budget checker

```python
import tiktoken

_encoding = tiktoken.get_encoding("o200k_base")

def count_tokens(text):
    return len(_encoding.encode(text))

def check_budget(parts, limit):
    # Count tokens for each labelled part of the context
    counts = {name: count_tokens(text) for name, text in parts.items()}
    total = sum(counts.values())
    if total <= limit:
        return True, total
    # Report which part is the largest before returning False
    biggest = max(counts, key=counts.get)
    print(f"Over budget: total {total} > {limit}, largest part: {biggest}")
    return False, total

parts = {
    "system": "You are a helpful support agent.",
    "history": "User asked about shipping. Assistant explained 5-10 days.",
    "question": "Can I get a refund for my last order?",
}
print(check_budget(parts, 100))
# Expected output: (True, <some number well under 100>)
```

**အဓိကအယူအဆ** — Context budget ကို အပိုင်းခွဲ တိုင်းတာထားရင် ဘယ်ပိုင်းက နေရာ အများဆုံးစားနေသလဲ ဆိုတာကို သိပြီး ရွေးချယ်စရာ ရှိလာပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Honest truncation

```python
def truncate_honestly(text, max_chars=200):
    # Short text passes through unchanged
    if len(text) <= max_chars:
        return text
    # Keep the first and last halves of the budget
    keep = (max_chars - 40) // 2
    marker = "\n[...TRUNCATED: middle section removed...]\n"
    return text[:keep] + marker + text[-keep:]

short = "Short text stays as it is."
long_text = "A" * 500 + " IMPORTANT END " + "B" * 500
result = truncate_honestly(long_text, max_chars=300)
print(truncate_honestly(short, 300) == short)
print("[...TRUNCATED" in result and result.endswith("B" * 130))
# Expected output: True
# Expected output: True
```

**အဓိကအယူအဆ** — ဖြတ်တောက်ခြင်းကို ရှင်းလင်းစွာ မှတ်ချက်ထည့်ပေးခြင်းဖြင့် model နဲ့ user နှစ်ဦးစလုံး အချက်အလက် ပျောက်နေခဲ့ကြောင်း သိရှိနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၄ — Keyword retrieval

```python
def retrieve(query, docs, top_k=2):
    # Score each document by how many query words appear in it
    query_words = set(query.lower().split())
    scored = []
    for i, doc in enumerate(docs):
        doc_lower = doc.lower()
        score = sum(1 for w in query_words if w in doc_lower)
        scored.append((score, i, doc))
    # Sort by score descending, then keep only positive scores
    scored.sort(key=lambda t: (-t[0], t[1]))
    return [doc for score, _, doc in scored[:top_k] if score > 0]

documents = [
    "Our refund policy allows returns within 30 days.",
    "Shipping to Myanmar takes 5 to 10 business days.",
    "Premium members receive free shipping on all orders.",
]
print(retrieve("How long does shipping take?", documents, top_k=2))
# Expected output: ['Shipping to Myanmar takes 5 to 10 business days.', 'Premium members receive free shipping on all orders.']
```

**အဓိကအယူအဆ** — ရိုးရိုး keyword matching ပင်လျှင် document တွေအားလုံးကို ထည့်စရာ မလိုဘဲ ဆ

## လေ့ကျင့်ခန်း ၅ — History summarisation

Chat history တစ်ခုရဲ့ ရှည်လာမှုကို ထိန်းသိမ်းဖို့အတွက် အသုံးအများဆုံးနည်းလမ်းကတော့ ဟောင်းနေပြီဖြစ်တဲ့ မက်ဆေ့ဂ်တွေကို အနှစ်ချုပ်တစ်ခုအဖြစ် ချုံ့ဖြတ်ပြီး၊ မူရင်း မက်ဆေ့ဂ်အစား အစားထိုးတာ ပဲ ဖြစ်ပါတယ်။ ဒီနည်းနဲ့ဆိုရင် token အရေအတွက် သိသိသိသိကျသွားပေမယ့် မေးမြန်းချက်ရဲ့ အခြေအနေ (context) ကတော့ ဆက်လက်တည်ရှိနေမှာ ဖြစ်ပါတယ်။

```python
def build_context(messages, keep_recent):
    # If the history is short enough, return it unchanged
    if len(messages) <= keep_recent:
        return list(messages)

    # Split history into the old part and the recent part
    old_part = messages[:-keep_recent]
    recent_part = messages[-keep_recent:]

    # Build a one-line summary from the old messages
    # Each message is a dict with "role" and "content" keys
    speakers = []
    for msg in old_part:
        role = msg.get("role", "unknown")
        if role not in speakers:
            speakers.append(role)

    # Count how many messages each role contributed
    counts = {}
    for msg in old_part:
        role = msg.get("role", "unknown")
        counts[role] = counts.get(role, 0) + 1

    # Compose the summary line
    parts = []
    for role, count in counts.items():
        parts.append(f"{count} {role} messages")
    summary_text = (
        "Summary of earlier conversation: "
        + ", ".join(parts)
        + " about the topic discussed before."
    )

    # Place the summary first, followed by the recent messages
    summary_message = {"role": "system", "content": summary_text}
    return [summary_message] + list(recent_part)


# Example usage
history = [
    {"role": "user", "content": "Hi, I need help with Python."},
    {"role": "assistant", "content": "Sure! What do you want to know?"},
    {"role": "user", "content": "How do lists work?"},
    {"role": "assistant", "content": "Lists are ordered collections in Python."},
    {"role": "user", "content": "Can you show me slicing?"},
]

result = build_context(history, 3)
for msg in result:
    print(f"[{msg['role']}] {msg['content']}")
```

`build_context` function ကို ခေါ်တဲ့အခါမှာ `messages[:-keep_recent]` ဆိုတဲ့ slicing နဲ့ ဟောင်းတဲ့ပိုင်းကို ဖယ်ရှားပြီး၊ `messages[-keep_recent:]` နဲ့ နောက်ဆုံးမက်ဆေ့ဂ်တွေကို အပြည့်ချန်ထားပါတယ်။ History အရေအတွက်က `keep_recent` ထက် မကျေဘဲ တိုရင်တော့ ဘာမှ မပြောင်းပြနမ်ဘဲ မူရင်းအတိုင်း ပြန်ပေးပြီး၊ ရှည်ရင်တော့ `system` role နဲ့ ရှင်းလင်းတဲ့ အနှစ်ချုပ်တစ်ကြောင်းကို ရှေ့ဆုံးမှာ ထည့်ပေးပါတယ်။

**အဓိကအယူအဆ** — History ရှည်လာတဲ့အခါ ဟောင်းတဲ့မက်ဆေ့ဂ်တွေကို summary တစ်ကြောင်းနဲ့ အစားထိုးပြီး နောက်ဆုံး `keep_recent` မက်ဆေ့ဂ်တွေကိုသာ အပြည့်ချန်ခြင်းဖြင့် context ကို ဆက်တည်ရှိစေလျက် token အရေအတွက်ကို သိသိသိသိ ကျစေနိုင်ပါတယ်။

## လေ့ကျင့်ခန်း ၆ — Cache-friendly prompt structure

```python
def make_prompt(prefix, question):
    # Keep the stable (cacheable) part first, append the variable part at the end
    return prefix + question


# A long, stable prefix that stays identical across requests
prefix = (
    "You are a senior Python engineer. "
    "Answer concisely with runnable code examples. "
    "Always explain trade-offs briefly. "
)

# Two different questions share the same prefix
q1 = "How do I read a file line by line?"
q2 = "How do I sort a list of dicts by a key?"

p1 = make_prompt(prefix, q1)
p2 = make_prompt(prefix, q2)

# Both prompts must start with the exact same prefix
print("p1 starts with prefix:", p1.startswith(prefix))
print("p2 starts with prefix:", p2.startswith(prefix))

# The prefixes (everything before the question) must be identical
print("Common prefix identical:", p1[:len(prefix)] == p2[:len(prefix)])

# The tails differ because the questions differ
print("Different tails:", p1 != p2)
```

**အဓိကအယူအဆ** — ပြောင်းလဲတဲ့ အပိုင်းကို prompt ရဲ့ နောက်ဆုံးမှာ ထည့်ပြီး ရှေ့ prefix ကို မပြောင်းဘဲ တူညီစွာ ထားခြင်းအားဖြင့် မတူညီတဲ့ မေးခွန်းများအတွက်လည်း cache အရ အလားအလာ ရရှိစေသည်။
