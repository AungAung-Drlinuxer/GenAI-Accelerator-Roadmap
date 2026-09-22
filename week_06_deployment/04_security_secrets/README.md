# Security Hardening & Secrets Management

AI application တစ်ခုကို production မှာ deploy လုပ်တဲ့အချိန်မှာ secret တွေကို vault ထဲမှာ လုံခြုံစွာသိမ်းဆည်းပြီး database role တွေ၊ network boundary တွေ၊ dependency scanning နဲ့ audit logging တွေနဲ့အတူ security ကို ခိုင်မာအောင်လုပ်နည်းသင်ကြားပါမယ်။

## ဒီ module မှာ ဘာသင်မလဲ

- API key၊ password စတဲ့ secrets တွေကို git repository ထဲ မထည့်ပဲ vault (HashiCorp Vault သို့မဟုတ် Kubernetes Secret sealing) ထဲမှာ သိမ်းနည်း
- Database အတွက် least-privilege role တွေကို သတ်မှတ်ပြီး application တွေရဲ့ ရယူနိုင်တဲ့ အခွင့်အရေးကို ကန့်သတ်နည်း
- Network boundary တွေနဲ့ service တွေကြားမှာ ဆက်သွယ်မှုကို ထိန်းချုပ်နည်း
- Dependency နဲ့ container image တွေကို vulnerability အတွက် scan လုပ်နည်း
- Audit logging နဲ့ API key rotation လုပ်ငန်းစဉ်ကို ပြုလုပ်နည်း
- OWASP LLM Top 10 ထဲက security risk တွေနဲ့ ဒီနည်းလမ်းတွေ ဘယ်လိုဆက်စပ်သလဲ

## သင်ခန်းစာများ

1. Secrets in a Vault — git ထဲမှာ secret မထားပဲ HashiCorp Vault သို့မဟုတ် Sealed Secrets နဲ့ စီမံခြင်း
2. Least-Privilege Database Roles — application အတွက် လိုအပ်တဲ့အပိုင်းပဲ ရယူနိုင်တဲ့ DB role ဖန်တီးခြင်း
3. Network Boundaries — firewall၊ private network နဲ့ ingress rule တွေနဲ့ ခွဲခြားခြင်း
4. Dependency နဲ့ Image Scanning — pip-audit နဲ့ Trivy သုံးပြီး vulnerable package ရှာဖွေခြင်း
5. Audit Logging — ဘယ်သူဘယ်အချိန်မှာ ဘာလုပ်လဲဆိုတာ မှတ်တမ်းတင်ခြင်း
6. Key Rotation — API key တွေကို ပုံမှန်ပြောင်းလဲပေးခြင်းနဲ့ rotation အလိုအလျောက်လုပ်နည်း

## လိုအပ်ချက်များ (Prerequisites)

- Week 5 မှာ deploy လုပ်ခဲ့တဲ့ AI application နမူနာ
- Docker နဲ့ Kubernetes အခြေခံ၊ kubectl command သိဖို့
- SQL database (ဥပမာ PostgreSQL) အသုံးပြုနည်းအခြေခံ
- Python နဲ့ environment variable ဖတ်နည်း နားလည်ဖို့

## ဘယ်အချိန်မှာ အသုံးဝင်လဲ

- LLM API key တွေကို source code ထဲ တိုက်ရိုက်မရေးပဲ production မှာ လုံခြုံစွာ သုံးချင်တဲ့အခါ
- Database ကို application တစ်ခုတည်းရဲ့ ရယူခွင့်နဲ့ ကန့်သတ်ချင်တဲ့အခါ
- Container image ထဲမှာ known vulnerability ရှိမရှိ စစ်ချင်တဲ့အခါ
- Compliance (ဥပမာ audit trail လိုအပ်ချက်) ပြီးမြောက်အောင် လုပ်ဆောင်ရတဲ့အခါ

## ကိုးကား

- OWASP Top 10 for LLM Applications — https://owasp.org/www-project-top-10-for-large-language-model-applications/
- HashiCorp Vault Documentation — https://developer.hashicorp.com/vault/docs
- Kubernetes Secrets — https://kubernetes.io/docs/concepts/configuration/secret/
- Sealed Secrets (kubeseal) — https://github.com/bitnami-labs/sealed-secrets
- Trivy Vulnerability Scanner — https://github.com/aquasecurity/trivy
- pip-audit — https://github.com/pypa/pip-audit
- PostgreSQL Roles နဲ့ Permissions — https://www.postgresql.org/docs/current/user-manag.html
