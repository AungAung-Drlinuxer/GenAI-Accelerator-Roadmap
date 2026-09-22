# Prompt Engineering Fundamentals

LLM application တစ်ခုကို တည်ဆောက်ရာမှာ prompt design က ရလာဒ်အရည်အသွေးကို အများဆုံးသက်ရောက်တဲ့ အချက်ဖြစ်တဲ့အတွက် ဒီ module မှာ prompt engineering အခြေခံများကို လေ့လာမည်။

## ဒီ module မှာ ဘာသင်မလဲ

- System prompt နဲ့ user prompt ကွာခြားချက်နဲ့ သင့်သည့်အသုံးပြုပုံ
- Instruction hierarchy ဆိုတာ ဘာလဲ၊ ဘာကြောင့် model ရဲ့ behavior ကို ထိန်းချုပ်နိုင်လဲ
- Few-shot examples ထည့်သွင်းပြီး output quality တိုးတက်စေပုံ
- Output format constraints (JSON schema၊ delimiter) သတ်မှတ်ပုံ
- Temperature နဲ့ top_p parameter များရဲ့ tradeoff
- Prompt versioning လုပ်ပုံနဲ့ prompt များကို source codeကဲ့သလို စီမံခန့်ခွဲပုံ

## သင်ခန်းစာများ

1. System vs User Prompts
2. Instruction Hierarchy
3. Few-shot Examples
4. Output Format Constraints
5. Temperature နှင့် Top-p Tradeoffs
6. Prompt Versioning

## လိုအပ်ချက်များ (Prerequisites)

- Python အခြေခံ (function၊ dictionary၊ JSON serialization)
- OpenAI API key ရှိခြင်း (သို့) တခြား LLM API တစ်ခု အသုံးပြုနိုင်ခြင်း
- `pip install openai` နဲ့ client library install လုပ်ထားခြင်း

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- Chatbot၊ assistant၊ RAG pipeline စတဲ့ LLM application များ တည်ဆောက်ချင်စဉ်
- Model output က structured data (JSON) ဖြစ်လာစေချင်စဉ်
- Prompt များစွာကို အုပ်စုလိုက် test လုပ်ပြီး production မှာ ထိန်းသိမ်းချင်စဉ်
- Model behavior ကို reliability ရှိရှိ ထပ်ခါထပ်ခါ ရလာစေချင်စဉ်

## ကိုးကား

- OpenAI Prompt Engineering Guide — https://platform.openai.com/docs/guides/prompt-engineering
- OpenAI Text Generation Guide — https://platform.openai.com/docs/guides/text-generation
- OpenAI API Reference (Chat Completions) — https://platform.openai.com/docs/api-reference/chat
