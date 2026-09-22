# Week 6 — Deploying AI Applications

## ဒီပတ်မှာ ဘာသင်မလဲ

- AI application တစ်ခုကို container image အဖြစ် pack လုပ်ပြီး production server တစ်ခုပေါ်မှာ deploy တက်နည်း
- Caddy reverse proxy နဲ့ automatic HTTPS (TLS certificate) ထားနည်း၊ health check နဲ့ zero-downtime restart လုပ်နည်း
- GitHub Actions သုံးပြီး CI/CD pipeline တည်ဆောက်ကာ build → test → scan → registry push → deploy အဆင့်ဆင့် အလိုအလျောက် လုပ်နည်း
- Structured logging၊ Sentry error tracking၊ metrics/dashboards နဲ့ SLO (p95 latency, error rate) သတ်မှတ်ပြီး စောင့်ကြည့်နည်း
- Secrets များကို vault ထဲမှာသိမ်းခြင်း၊ least-privilege DB roles၊ dependency/image scanning နဲ့ key rotation အပါအဝင် security hardening လုပ်နည်း

## Modules

- `01_container_deploy/` — **Container Deploy & Reverse Proxy with TLS** — Docker image ship လုပ်ခြင်း၊ env/config ခွဲခြားခြင်း၊ Caddy နဲ့ HTTPS၊ health check နဲ့ downtime မရှိ restart လုပ်ပုံ သင်ပါတယ်။
- `02_cicd_pipelines/` — **CI/CD Pipelines for AI Services** — build/test/scan stages၊ immutable tags/digests နဲ့ registry push၊ digest အားဖြင့် deploy၊ smoke test နဲ့ rollback strategy သင်ပါတယ်။
- `03_observability_alerts/` — **Observability, Errors & Alerts** — structured logging၊ Sentry နဲ့ error tracking၊ metrics/dashboards၊ SLO နဲ့ on-call playbook သင်ပါတယ်။
- `04_security_secrets/` — **Security Hardening & Secrets Management** — vault ထဲ secrets သိမ်းခြင်း၊ least-privilege DB roles၊ scanning၊ audit logging၊ key rotation သင်ပါတယ်။

## ဒီပတ်ရဲ့ ရည်မှန်းချက်

ဒီပတ်ပြီးဆုံးရင် သင်ဟာ —

- AI service တစ်ခုကို container ထဲကနေ production deployment တစ်ခုအထိ ကိုယ့်ဖာသာ deploy တက်နိုင်ပါမယ်
- CI/CD pipeline တစ်ခုကို ကိုယ်တိုင်ရေးပြီး deploy လုပ်တိုင်းး smoke test စစ်ပြီး မအောင်ရင် rollback လုပ်နိုင်ပါမယ်
- production မှာ error တက်ရင် alert ချက်ဖမ်းပြီး SLO အရ ဆုံးရှုံးမှုကို တိုင်းတာနိုင်ပါမယ်
- secrets တွေကို git ထဲမထည့်ဘဲ vault နဲ့ စီမံပြီး security အခြေခံများကို အသုံးချနိုင်ပါမယ်

## လေ့လာရန် အစီအစဉ်

| နေ့ | Module |
|------|--------|
| Day 1–2 | `01_container_deploy/` — image တည်ဆောက်ပြီး server ပေါ် deploy၊ Caddy နဲ့ HTTPS ထား |
| Day 3–4 | `02_cicd_pipelines/` — GitHub Actions pipeline ရေးပြီး digest deploy လုပ် |
| Day 5 | `03_observability_alerts/` — logging, Sentry, metrics, SLO ချိတ်ကြည့် |
| Day 6 | `04_security_secrets/` — vault နဲ့ secrets စီမံ၊ hardening checklist လုပ် |
| Day 7 | Checkpoint အောင်မြင်မှု စစ်ပြီး ဒီပတ် project အား review |

## Checkpoint

1. Secrets တွေကို `.env` file အနေနဲ့ git repository ထဲ တိုက်ရိုက် commit လုပ်တာက ဘာကြောင့် အန္တရာယ်ရှိပြီး အစားထိုးနည်းက ဘာလဲ။
2. Immutable tag (digest) နဲ့ deploy လုပ်တာက mutable tag (ဥပမာ `:latest`) ထက် ဘာကြောင့် ပိုစိတ်ချရပါသလဲ။
3. p95 latency ဆိုတာကို ဘယ်လို ဖွင့်ပြပြီး SLO တစ်ခုအဖြစ် ဘာကြောင့် သတ်မှတ်ကြပါသလဲ။
4. Zero-downtime restart ရှိဖို့ health check ဟာ deployment ထဲမှာ ဘယ်နေရာမှာ ပါဝင်ပါသလဲ၊ ဘာကြောင့်လဲ။
