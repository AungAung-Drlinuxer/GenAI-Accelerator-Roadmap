# လေ့ကျင့်ခန်းများ — Context Engineering & Token Budgets

## လေ့ကျင့်ခန်း ၁ — Token counter ရေးပါ
`tiktoken` သုံးပြီး စာသားတစ်ခုရဲ့ token အရေအတွက်ကို တွက်ပေးမယ့် `count_tokens(text)` function ရေးပါ။ မြန်မာစာ တစ်ကြောင်းနဲ့ အင်္ဂလိပ်စာ တစ်ကြောင်း နှစ်ခုစလုံးကို တိုင်းပြပါ။
**Hints:** `tiktoken.get_encoding("o200k_base")` နဲ့ encoder ယူပြီး `.encode(text)` ၏ length ကို တွက်ပါ။
**Expected behavior:** Function က စာသားတစ်ခုချင်းဆီအတွက် ကိန်းဂဏန်းတစ်ခု (token count) ပြန်ထုတ်ပါ။

## လေ့ကျင့်ခန်း ၂ — Context budget checker
System prompt၊ chat history၊ user message သုံးမျိုးပေါင်းချက်ရဲ့ စုစုပေါင်း token ကို တွက်ပြီး `check_budget(parts, limit)` function ရေးပါ။ Limit ကျော်ရင် `False` နဲ့ ဘယ်ပိုင်းက အများဆုံးနေသလဲ ဆိုတာပါ ပြပါ။
**Hints:** လေ့ကျင့်ခန်း ၁ ရဲ့ `count_tokens` ကို ပြန်သုံးပြီး dict တစ်ခုထဲ တစ်ပိုင်းချင်း count သိမ်းပါ။
**Expected behavior:** Limit အတွင်းရှိရင် `(True, total)`၊ ကျော်ရင် `(False, total)` ပြန်ပါ။

## လေ့ကျင့်ခန်း ၃ — Honest truncation
စာသားရှည်ကြီးကို အလယ်ကနေ ဖြတ်တောက်ပြီး ဖြတ်တောက်မှုကို ရှင်းရှင်းလင်းလင်း မှတ်ချက်ထည့်တဲ့ `truncate_honestly(text, max_chars)` function ရေးပါ။ အစပိုင်းနဲ့ အဆုံးပိုင်း နှစ်ခုလုံး ဆက်နေရမယ်။
**Hints:** `max_chars // 2` ခန့် ရှေ့/နောက် ချန်ပြီး အလယ်မှာ "[...TRUNCATED...]" marker ထည့်ပါ။
**Expected behavior:** တိုတဲ့ input က အတုံးလိုက် ပြန်ပြီး၊ ရှည်တဲ့ input မှာ marker ပါတဲ့ ရလဒ် ထွက်ပါ။

## လေ့ကျင့်ခန်း ၄ — Keyword retrieval
Document list တစ်ခုကနေ မေးခွန်းနဲ့ ဆင်ဆင်တူတဲ့ document `top_k` ခုကို ရွေးပေးမယ့် `retrieve(query, docs, top_k)` function ရေးပါ။
**Hints:** မေးခွန်းရဲ့ စကားလုံးတွေနဲ့ document ထဲ ဘယ်လောက် အကြိမ်ကြိမ် တွေ့လဲ ဆိုတာကို score အဖြစ် သုံးပြီး sort လုပ်ပါ။
**Expected behavior:** Score မရှိတဲ့ document တွေ ပြန်မထွက်ဘဲ၊ score အမြင့်ဆုံး `top_k` ခုကိုသာ ပြန်ပါ။

## လေ့ကျင့်ခန်း ၅ — History summarisation
Chat history တစ်ခုကို ယူပြီး ဟောင်းတဲ့ မက်ဆေ့ဂ်တွေကို အနှစ်ချုပ် တစ်ကြောင်းနဲ့ အစားထိုးပြီး၊ နောက်ဆုံး `keep_recent` မက်ဆေ့ဂ်တွေကို အပြည့် ချန်တဲ့ `build_context(messages, keep_recent)` function ရေးပါ။
**Hints:** `messages[:-keep_recent]` က ဟောင်းတဲ့ပိုင်း၊ `messages[-keep_recent:]` က အသစ်ပိုင်း ဖြစ်ပါတယ်။
**Expected behavior:** History တိုရင် မပြောင်းဘဲ ပြန်ပြီး၊ ရှင်းလင်းစွာ အနှစ်ချုပ် summary ကို ရှေ့ဆုံးမှာ ထည့်ပါ။

## လေ့ကျင့်ခန်း ၆ — Cache-friendly prompt structure
ပြောင်းလဲတဲ့ အပိုင်းကို prompt ရဲ့ နောက်ဆုံးမှာ ထည့်ပေးမယ့် `make_prompt(prefix, question)` function ရေးပြီး၊ မတူညီတဲ့ မေးခွန်းနှစ်ခုအတွက် ရလဒ်တွေရဲ့ ရှေ့ပိုင်း `prefix` တူကြောင်း စစ်ပါ။
**Hints:** မတူတဲ့ မေးခွန်းနှစ်ခုနဲ့ prompt နှစ်ခု ဖန်တီးပြီး နှစ်ခုစလုံးဟာ `prefix` နဲ့ စတာကို `startswith` နဲ့ စစ်ပါ။
**Expected behavior:** မေးခွန်း မတူကြောင့်နောက်ပိုင်း ကွဲသွားပေမယ့်၊ ရှေ့ prefix အပိုင်း အတိအကျ တူနေပါ။
