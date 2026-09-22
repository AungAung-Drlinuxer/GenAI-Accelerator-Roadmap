# CI/CD Pipelines for AI Services

AI service တစ်ခုကို production မှာ ယုံကြည်စိတ်ချစွာ တင်ရန်အတွက CI/CD pipeline တည်ဆောက်ပုံကို သင်ကြားစေသည့် module ဖြစ်သည်။

## ဒီ module မှာ ဘာသင်မလဲ

- Build / test / scan stage တွေကို GitHub Actions နဲ့ တစ်ဆက်တည်း စီမံပုံ
- Container registry ထဲကို immutable tag နှင့် digest အသုံးပြု၍ push ပြုလုပ်ပုံ
- Image digest ဖြင့် deploy လုပ်ခြင်း၏ အနှစ်ချုပ်
- Deploy ပြီးနောက် smoke test လုပ်ဆောင်ခြင်း
- ပျက်စီးခဲ့ပါက ရှေ့ version သို့ ပြန်လည် rollback စနစ်

## သင်ခန်းစာများ

1. CI Pipeline အဆောက်အအုံ (build, test, scan)
2. Registry push နှင့် immutable tags/digests
3. Digest ဖြင့် deploy ပြုလုပ်ခြင်း
4. Smoke test နှင့် rollback strategy

## လိုအပ်ချက်များ (Prerequisites)

- Python basics နှင့် Docker အခြေခံ
- Git နှင့် GitHub repository တစ်ခု
- Week 5 container/module များ ပြီးမြောက်ထားခြင်း

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- AI service (model API, chatbot backend, vector search service) ကို production server, cloud သို့မဟုတ် Kubernetes ပေါ်တင်ရန် pipeline လိုအပ်ချိန်
- "server ပေါ်တင်ပြီးရင် ဘာဖြစ်မလဲ မသိဘူး" ဟူသော စိုးရိမ်မှုကို ဖျောက်လိုသောအခါ

## ကိုးကား

- GitHub Actions documentation: https://docs.github.com/actions
- Publishing Docker images: https://docs.github.com/actions/publishing-packages/publishing-docker-images
- Docker registry digests: https://docs.docker.com/reference/cli/image/pull/
- Trivy scanner docs: https://trivy.dev
