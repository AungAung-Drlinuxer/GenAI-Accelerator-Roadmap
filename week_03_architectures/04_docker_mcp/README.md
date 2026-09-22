# Containerization & MCP Integration

Docker နဲ့ Python backend service တွေကို containerize လုပ်ခြင်း၊ multi-stage build ရေးခြင်း၊ Compose နဲ့ api/worker/db/redis တွေကို ချိတ်ဆက်ခြင်းနဲ့ MCP server ကို backend ထဲမှာ ဖော်ပြခြင်းတို့ကို သင်ရမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- Python service တစ်ခုအတွက် ထိထိရှိရှိ Dockerfile ရေးတာ
- Multi-stage build နဲ့ image size လျှော့ပြီး production-ready ဖန်တီးတာ
- Docker Compose နဲ့ api, worker, db, redis service တွေကို တစ်နေရာတည်းက စီမံတာ
- Secret တွေကို environment variable အနေနဲ့ လုံခြုံစွာ ထည့်သွင်းအသုံးချတာ
- Model Context Protocol (MCP) server တစ်ခုကို backend service ထဲမှာ expose လုပ်တာ

## သင်ခန်းစာများ

1. **Python Dockerfile အခြေခံ** — base image ရွေးချယ်ခြင်း၊ dependency install လုပ်ခြင်းနဲ့ image layer တွေရဲ့ သဘောတရား
2. **Multi-stage builds** — build stage နဲ့ runtime stage ခွဲခြင်း၊ လိုအပ်တဲ့ file တွေပဲ နောက်ဆုံး image ထဲထည့်ခြင်း
3. **Docker Compose နဲ့ service wiring** — api, worker, db (Postgres), redis တွေကို network တစ်ခုတည်းမှာ ချိတ်ဆက်ခြင်း
4. **Secrets as environment variables** — `.env` file, Compose environment config နဲ့ container runtime မှာ secret တွေကို စီမံခြင်း
5. **MCP server exposure** — stdio နဲ့ HTTP/SSE transport ကွာခြားချက်၊ MCP server ကို backend service တစ်ခုအနေနဲ့ container ထဲ run ခြင်း
6. **Compose ထဲမှာ MCP service ထည့်ခြင်း** — MCP server container ကို api service နဲ့ ချိတ်ဆက်ပြီး end-to-end စမ်းသပ်ခြင်း

## လိုအပ်ချက်များ (Prerequisites)

- Python basics (functions, modules, virtual environment) နဲ့ FastAPI အခြေခံ
- Week 2 က production API patterns တွေကို နားလည်ထားခြင်း
- ကိုယ်ပိုင် computer မှာ Docker Engine (ဒါမှမဟုတ် Docker Desktop) တပ်ဆင်ထားခြင်း
- Command line / terminal အသုံးပြုတာ အနည်းငယ် သိထားခြင်း
- Git နဲ့ project folder structure အခြေခံ

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- LLM feature ပါတဲ့ API service တစ်ခုကို staging ဒါမှမဟုတ် production မှာ deploy ရမယ့်အခါ
- အဖွဲ့ဝင်တွေအားလုံး တူညီတဲ့ environment နဲ့ develop လုပ်နိုင်ဖို့ local setup တွေကို ရိုးရိုးရှင်းရှင်း ဖန်တီးချင်တဲ့အခါ
- API, background worker, database, cache တွေပါတဲ့ full stack တစ်ခုကို တစ် command နဲ့ ထ ချင်တဲ့အခါ
- AI agent တွေကို tools တွေနဲ့ ချိတ်ပေးမယ့် MCP server တစ်ခုကို service တစ်ခုအနေနဲ့ ထုတ်ပေးချင်တဲ့အခါ

## ကိုးကား

- Dockerfile reference — https://docs.docker.com/reference/dockerfile/
- Multi-stage builds guide — https://docs.docker.com/build/building/multi-stage/
- Docker Compose overview — https://docs.docker.com/compose/
- Compose environment variables — https://docs.docker.com/compose/environment-variables/env-file/
- Model Context Protocol documentation — https://modelcontextprotocol.io
- MCP Python SDK — https://github.com/modelcontextprotocol/python-sdk
