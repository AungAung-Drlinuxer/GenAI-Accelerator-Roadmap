# Container Deploy & Reverse Proxy with TLS

AI application တစ်ခုကို Docker container နှင့် တွဲဖက်ပြီး Caddy reverse proxy ကို အသုံးပြု၍ automatic HTTPS ဖြင့် production ပတ်ဝန်းကျင်သို့ တင်ပေးနည်းကို လေ့လက်မည်။

## ဒီ module မှာ ဘာသင်မလဲ

- Docker image များကို build ပြီး registry သို့ push တင်နည်း
- Environment variable များဖြင့် configuration ကို code မှ ခွဲထုတ်နည်း
- Caddy reverse proxy ကို အသုံးပြုပြီး automatic HTTPS certificate ရယူနည်း
- Health check endpoint များ ရေးသားပြီး container တွင် ချိတ်ဆက်နည်း
- Zero-downtime restart ပြုလုပ်နိုင်သော deployment ပုံစံကို ဖန်တီးနည်း

## သင်ခန်းစာများ

1. Container image build နှင့် ship လုပ်ခြင်း
2. Environment/config separation
3. Caddy reverse proxy နှင့် automatic HTTPS
4. Health checks စတင်ခြင်း
5. Zero-downtime restarts

## လိုအပ်ချက်များ (Prerequisites)

- Python 3.10 နှင့်အထက်၊ FastAPI framework အခြေခံကို နားလည်ထားခြင်း
- Docker Desktop (docker နှင့် docker compose commands) ထည့်သွင်းပြီးဖြစ်ခြင်း
- ကိုယ်ပိုင် domain name တစ်ခု ရှိခြင်း (HTTPS အတွက် လိုအပ်သည်)
- အခြေခံ shell/command line အသုံးပြုနိုင်စွမ်း

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- AI model inference API တစ်ခုကို အင်တာနက်ပေါ်တွင် လုံခုံစွာ ဝန်ဆောင်မှုပေးလိုသောအခါ
- HTTPS certificate များကို လက်ဖြင့် စီမံစရာ မလိုပဲ automatic လုပ်ဆောင်စေလိုသောအခါ
- Deployment လုပ်စဉ် ဝန်ဆောင်မှု မရပ်တန့်စေဘဲ update လုပ်လိုသောအခါ

## ကိုးကား

- Docker Documentation — https://docs.docker.com/
- Dockerfile reference — https://docs.docker.com/reference/dockerfile/
- Docker Compose overview — https://docs.docker.com/compose/
- Caddy Documentation — https://caddyserver.com/docs/
- Caddy Automatic HTTPS — https://caddyserver.com/docs/automatic-https
