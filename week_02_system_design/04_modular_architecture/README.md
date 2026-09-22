# Modular Architecture: Workflows, Nodes, DI

AI system တစ်ခုကို တစ်ခြား component များနဲ့ ချိတ်ဆက်ထားတဲ့ workflow နှင့် node များအဖြစ် ခွဲခြားဖွဲ့စည်းပုံ၊ အမျိုးအစားခွဲနိုင်တဲ့ input/output၊ dependency injection နှင့် failure isolation အခြေခံများကို ဒီ module မှာ လေ့လာမည်။

## ဒီ module မှာ ဘာသင်မလဲ

- Orchestration (အလုပ်ချုပ်ဆောင်မှု) နှင့် အဆင့်တစ်ဆင့်ချင်းရှိ step များကို ဘယ်လိုခွဲကြောင်း
- Typed node input/output သတ်မှတ်ပြီး data စီးဆင်းမှုကို အမှန်ချကြည့်နိုင်စွမ်း
- Dependency injection ကိုသုံးပြီး test ရလွယ်တဲ့ node များရေးနည်း
- တစ်ခုမှာ fail ရင် တစ်ခြားအပိုင်းကို မထိခိုက်အောင် failure isolation လုပ်နည်း
- Observability hooks သုံးပြီး workflow ထဲက အချက်အလက်များကို စောင့်ကြည့်နည်း

## သင်ခန်းစာများ

1. Workflow ဆိုတာဘာလဲ — node များကို ချိတ်ဆက်ပုံ အယူအဆ
2. Orchestration နှင့် step logic ကို သီးသန့်ခွဲထားခြင်း၏ အကျိုးကျေးဇူး
3. Python type hints နှင့် Pydantic သုံးပြီး typed node input/output သတ်မှတ်ခြင်း
4. Dependency injection အခြေခံ — function parameter များမှတဆင့် dependency ထိုးခြင်း
5. Failure isolation — node တစ်ခု fails သောအခါ error ကို ကန့်သတ်ခြင်း၊ retry နှင့် fallback
6. Observability hooks — logging၊ callback နှင့် tracing အခြေခံများ
7. Mini project — typed nodes နှင့် DI ပါတဲ့ simple document-processing workflow

## လိုအပ်ချက်များ (Prerequisites)

- Week 1 မှ Python basics၊ functions၊ classes နှင့် type hints
- Virtual environment ဖန်တီးပြီး `pip` နှင့် package ထည့်သွင်းနိုင်စွမ်း
- Git basics — code များကို repo ထဲမှာ စီမန်းနိုင်ခြင်း
- REST API ခေါ်ဆိုမှုအခြေခံ (HTTP request/response) အသိပညာ

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- AI pipeline တစ်ခုကို prompt တစ်ခုတည်းနဲ့ မဟုတ်ဘဲ အဆင့်များစွာဖြင့် တည်ဆောက်လိုသောအခါ
- Feature တစ်ခု ပြောင်းရင် တစ်ခြားအပိုင်းပါပျက်စေချင်တဲ့ ကြီးမားတဲ့ codebase များတွင်
- Unit test ရေးလိုသော်လည်း external service (LLM API၊ database) အစစ်နဲ့ ချိတ်ချင်မသောအခါ
- Production မှာ တစ်စိတ်တစ်ပိုင်း fail ရင် တစ်ကောင်လုံး မဆိုင်းတန့်စေချင်တဲ့ system များတွင်
- Team နှင့် အတူ component ခွဲဝေ၍ တည်ဆောက်လိုသောအခါ

## ကိုးကား

- Datalumina Launchpad — AI Engineering Roadmap: https://launchpad.datalumina.com
- Python typing documentation (type hints): https://docs.python.org/3/library/typing.html
- Pydantic documentation (data validation): https://docs.pydantic.dev/latest/
- FastAPI dependency injection concepts: https://fastapi.tiangolo.com/tutorial/dependencies/
- Python logging documentation: https://docs.python.org/3/library/logging.html
