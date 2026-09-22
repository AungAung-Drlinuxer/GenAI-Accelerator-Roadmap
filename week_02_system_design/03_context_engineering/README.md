# Context Engineering & Token Budgets

LLM ဆီ ဆက်သွယ်ပေးတဲ့ context window ကို စနစ်တကျ စီမံခြင်းနဲ့ token budget စီမံခြင်းအကြောင်း သင်ရမယ့် module ဖြစ်ပါတယ်။

## ဒီ module မှာ ဘာသင်မလဲ
- Context window ထဲမှာ ဘာတွေ ပါဝင်သလဲ၊ ဘယ်လောက်အထိ နေရာယူသလဲ
- Document တွေကို တန်းစောင်း ထည့်တဲ့ (stuffing) နည်းနဲ့ retrieval နည်း ကွာခြားချက်
- Context ကို ဖြတ်တောက်ရင် "honest truncation" ဆိုတာ ဘာလဲ
- ရှည်လျားတဲ့ စကားဝှက်များကို အနှစ်ချုပ် (summarisation) ဖြင့် စီမံနည်း
- Prompt caching နဲ့ ကုန်ကျစရိတ်/latency သက်သာအောင် လုပ်နည်း
- Context size နဲ့ ကုန်ကျစရိတ်၊ တုံ့ပြန်နှုန်း ဆက်နွယ်မှု

## သင်ခန်းစာများ
1. Context window ထဲ ဘာတွေ ပါဝင်သလဲ
2. Stuffing vs Retrieval
3. Truncation honesty (ဖြတ်တောက်မှု တင်ပြခြင်း)
4. Summarisation နဲ့ context ကျဉ်းအောင် လုပ်နည်း
5. Prompt caching
6. Context size ၏ cost/latency သက်ရောက်မှု

## လိုအပ်ချက်များ (Prerequisites)
- Week 1 ၏ LLM API အခြေခံ သင်ခန်းစာများကို ပြီးထားရန်
- Python function/class ရေးတတ်ရန်
- API key တစ်ခု သတ်မှတ်ပြီးသားဖြစ်ရန်

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ
- Chatbot တစ်ခုရဲ့ chat history ကြီးလာတဲ့အခါ token limit မကျော်အောင် စီမံချင်တဲ့အခါ
- Document ရှည်ကြီးတွေကို LLM နဲ့ စစ်ဆေးခိုင်းတဲ့ system ဆောက်တဲ့အခါ
- API ကုန်ကျစရိတ်ကို လျှော့ချင်တဲ့အခါ

## ကိုးကား
- OpenAI Prompt Caching Guide: https://platform.openai.com/docs/guides/prompt-caching
- OpenAI Models & Pricing Documentation: https://platform.openai.com/docs/models
- tiktoken library (open-source tokenizer): https://github.com/openai/tiktoken
