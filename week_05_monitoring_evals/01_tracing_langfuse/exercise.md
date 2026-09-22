# လေ့ကျင့်ခန်းများ — Tracing with Langfuse

## လေ့ကျင့်ခန်း ၁ — ပထမ trace ဖန်တီးပါ

`langfuse.trace()` နဲ့ trace တစ်ခု ဖန်တီးပြီး name, input ထည့်ပါ။ `get_trace_url()` နဲ့ URL ရနိုင်ကြောင်း စစ်ပါ။

**Hints:** `from langfuse import Langfuse` ကနေ client ဖန်တီးပါ။ Environment variable တွေ (`LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`) သတ်မှတ်ထားရန် လိုအပ်ပါတယ်။

**Expected behavior:** Python ကနေ run လို့ ဒီ trace URL တစ်ခု print ထွက်ပြီး Langfuse UI မှာ မြင်ရပါတယ်။

## လေ့ကျင့်ခန်း ၂ — Span တွေ nested ပုံစံ ထည့်ပါ

Trace တစ်ခုထဲမှာ `retrieve` span တစ်ခု နဲ့ `format` span တစ်ခု ဆက်တိုက် ထည့်ပြီး အသီးသီး `end()` ခေါ်ပါ။

**Hints:** `trace.span(name=...)` ကို ခေါ်ပြီး `span.end(output=...)` နဲ့ ပိတ်ပါ။ Output dict လေးတွေ ထည့်ကြည့်ပါ။

**Expected behavior:** UI မှာ trace tree ထဲမှာ span နှစ်ခု အစဉ်လိုက် မြင်ရပြီး duration တွေ သိမ်းဆည်းရပါတယ်။

## လေ့ကျင့်ခန်း ၃ — Generation capture လုပ်ပါ

Trace ထဲမှာ `trace.generation()` တစ်ခု ထည့်ပြီး model, input messages, output, နဲ့ `usage` ထည့်ပါ။

**Hints:** Input ကို chat message list format (`[{"role": "user", "content": "..."}]`) နဲ့ ရေးပါ။ Usage key တွေက `input`, `output`, `total` ပါ။

**Expected behavior:** UI ရဲ့ generation detail မှာ prompt, completion, token usage တွေ ပြည့်စုံ မြင်ရပါတယ်။

## လေ့ကျင့်ခန်း ၄ — Session နဲ့ user တွဲပါ

Trace နှစ်ခု ဖန်တီးပြီး `sessionId` တူတူ၊ `userId` တူတူ ပေးပါ။

**Hints:** `langfuse.trace(name=..., sessionId=..., userId=...)` ဆိုတဲ့ argument တွေကို သုံးပါ။

**Expected behavior:** Langfuse UI မှာ session id တစ်ခုအောက်မှာ trace နှစ်ခုစလုံး စုမြင်ရပါတယ်။

## လေ့ကျင့်ခန်း ၅ — Feature tags တပ်ပါ

`summarizer` နဲ့ `qa-bot` feature နှစ်မျိုးအတွက် trace တစ်ခုစီ ဖန်တီးပြီး ကွဲပြားတဲ့ `tags` တပ်ပါ။

**Hints:** `tags=["feature:summarizer"]` ပုံစံ သုံးပါ။ Environment tag (`env:dev`) ပါ တွဲထည့်ကြည့်ပါ။

**Expected behavior:** UI filter မှာ tag နဲ့ ရှာလိုက်ရင် feature တစ်ခုချင်းစီရဲ့ trace တွေပဲ ပေါ်လာပါတယ်။

## လေ့ကျင့်ခန်း ၆ — မှားတဲ့ အဖြေတစ်ခုကို debug လုပ်ပါ

Retrieval function တစ်ခု ရေးပြီး ဒီဇိုင်းအရ မှားတဲ့ document တစ်ခုကို ပြန်ပေးအောင် ပြုလုပ်ပါ။ ပြီးရင် trace span output တွေကို ကြည့်ပြီး ဘယ်ဆင့်မှာ ပြဿနာရှိလဲ ရှာပါ။

**Hints:** `retrieve` span ရဲ့ output ကို trace ထဲမှာ မှတ်ထားပါ။ Span output ကို ကြည့်ပြီး မှားတဲ့ doc ကို ဖော်ပြပါ။

**Expected behavior:** အဖြေမှားတာက LLM ရဲ့ အမှားမဟုတ်ဘဲ retrieval span ထဲက မှားတဲ့ doc ကြောင့်ဆိုတာ trace မှတ်တမ်းနဲ့ ပြသနိုင်ပါတယ်။
