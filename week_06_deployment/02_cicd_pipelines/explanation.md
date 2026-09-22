# CI/CD Pipelines for AI Services — ရှင်းလင်းချက်

## 1. CI Pipeline အဆောက်အအုံ (build, test, scan)

### ဘာကို ဆိုလိုတာလဲ

CI (Continuous Integration) ဆိုတာ code အသစ် push တိုင်း အော်တိုမက်တစ် ဖြင့် build လုပ်ခြင်း၊ unit test လုပ်ခြင်း၊ လုံခြုံမှု scan လုပ်ခြင်း စသည့် အဆင့်များကို တာဝန်ခံစွာ လုပ်ဆောင်ပေးသည့် စနစ်ဖြစ်သည်။ GitHub Actions တွင် `.github/workflows/` ဖိုလ်ထဲမှ YAML ဖြင့် ဤအဆင့်များကို သတ်မှတ်သည်။

### ဘာကြောင့် လဲ

လက်ဖြင့် လုပ်ဆောင်ပါက မမှန်ကန်နိုင်သလို၊ လူပင်ပန်လည်း ဖြစ်သည်။ AI service များတွင် dependency များစွာရှိ၍ တစ်ခုမှာ မှားယွင်းပါက တစ်ခြားနေရာထိ ထိခိုက်တတ်သည်။ Pipeline က တစ်ပြားမှ မကျဘဲ တစ်သမတ်တည်း စစ်ဆေးပေးသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

1. Developer က code ကို branch တစ်ခုသို့ push လုပ်သည်
2. Workflow "trigger" (ဥပမာ `on: push`) က လည်ပတ်စေသည်
3. Job တစ်ခုချင်းစီမှာ step များကို အစီအစဉ်ဖြင့် လုပ်ဆောင်သည်
4. Test တစ်ခုပျက်လျှင် pipeline ရပ်တန့်ပြီး merge မဖြစ်စေရ

### ဥပမာ

```python
# Minimal check script that a CI job would run:
import subprocess
import sys

def run_check(command):
    # Run a shell command and return True if exit code is 0
    result = subprocess.run(command, shell=True, capture_output=True)
    return result.returncode == 0

checks = [
    ("python -m pytest tests/ -q", "unit tests"),
    ("python -m ruff check .", "lint"),
]
failed = []
for cmd, label in checks:
    if not run_check(cmd):
        failed.append(label)

print("FAILED:", failed if failed else "none")
sys.exit(1 if failed else 0)
# Expected output: FAILED: none
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

AI service တစ်ခုဟာ model file, tokenizer, embedding library စတဲ့ အစိတ်အပိုင်း များစွာ မှီခိုသည်။ တစ်ခုမှာ version လွဲလျှင် ခန့်မှန်းချက်ထွက်တတ်သည်။ CI က dependency lock file, prompt logic, API handler တို့ကို အလိုအလျောက် စစ်ပေးသဖြင့် မတော်တဆ ထိခိုက်မှုကို ကာကွယ်သည်။

## 2. Registry Push နှင့် Immutable Tags/Digests

### ဘာကို ဆိုလိုတာလဲ

Container registry (ဥပမာ GHCR, Docker Hub) ဆိုတာ build ပြီး image များသိမ်းရန်နေရာဖြစ်သည်။ Image တစ်ခုကို ရည်ညွှန်းရန် နည်းလမ်းနှစ်မျိုးရှိသည် — tag (ဥပမာ `myapp:v1.2`) နှင့် digest (`myapp@sha256:abc...`) တို့ဖြစ်သည်။ Tag က ပြောင်းလဲနိုင်သော နာမည်ဖြစ်ပြီး digest က image အကြောင်းအရာ၏ ပြောင်းလဲမရသော fingerprint ဖြစ်သည်။

### ဘာကြောင့် လဲ

တူညီသော tag (ဥပမာ `latest`) ကို အကြိမ်များစွာ push လုပ်နိုင်သည်။ ထို့ကြောင့် `latest` ဆိုတာ အချိန်တိုင်း ကွဲပြားသော code ကို ရည်ညွှန်းနေနိုင်သည်။ Digest ကတော့ အကြောင်းအရာအတိအကျကိုသာ ရည်ညွှန်းသောကြောင့် ဘယ် code တင်လဲဆိုတာ သေချာစွာ သိနိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

1. CI job က Docker image ကို build လုပ်သည်
2. သီးခြားဖြစ်စေရန် commit SHA (ဥပမာ `v1.2-abc1234`) ပါသော tag ပေးသည်
3. Registry သို့ login လုပ်ပြီး push လုပ်သည်
4. Push ပြီးလျှင် digest ကို ထုတ်ယူ၍ နောက်အဆင့်သို့ ဖြတ်သည်

### ဥပမာ

```python
# Simulate generating an immutable tag from a git commit
import subprocess

def get_commit_sha():
    # Return the short git commit hash of HEAD
    result = subprocess.run(
        ["git", "rev-parse", "--short", "HEAD"],
        capture_output=True, text=True, check=True,
    )
    return result.stdout.strip()

sha = get_commit_sha()
tag = f"ai-service:v1.3-{sha}"
print(f"Pushing image with tag: {tag}")
# Expected output: Pushing image with tag: ai-service:v1.3-abc1234
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

အသုံးပြုသူ (client apps, အခြား service) တွေဟာ သင်၏ AI endpoint အပေါ် မှီခိုနေကြသည်။ Tag တစ်ခုတည်းကို ငြိမ်းခံမထားပါက တစ်ရက်နှင့်တစ်ရက် မတူသော model သုံးနေရတာမျိုး ဖြစ်လာနိုင်သည်။ Immutable tag နှင့် digest က "ဒီ deploy မှာ ဘာ version တင်လဲ" ဆိုတာကို စာရွက်စာတမ်းအတိအကျ ဖြစ်စေသည်။

## 3. Digest ဖြင့် Deploy ပြုလုပ်ခြင်း

### ဘာကို ဆိုလိုတာလဲ

Deploy stage မှာ server (သို့) cluster ကို "ဒီ digest ရှိတဲ့ image ကို တင်ပါ" ဟု တိကျစွာ ညွှန်ကြားခြင်းဖြစ်သည်။ Tag ဖြင့် မကြာခဏ ညွှန်းကြသော်လည်း digest ဖြင့် ညွှန်းခြင်းက ပိုယုံကြည်စိတ်ချရသည်။

### ဘာကြောင့် လဲ

Deploy အချိန်နှင့် pull အချိန်ကြားမှာ tag တစ်ခုကို အခြားသူတစ်ယောက်က ထပ်ရေးထားနိုင်သည်။ Digest ဖြင့် deploy လုပ်ပါက registry က အတွင်းပိုင်း အကြောင်းအရာကို hash ဖြင့် တိုက်ဆိုင်စစ်ဆေး၍ ကွဲလွဲမှုရှိလျှင် deploy က အလိုအလျောက် ပျက်စေသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

1. Build stage က digest ကို output အဖြစ် ထုတ်ယူထားသည်
2. Deploy stage (သို့) နောက် job က ထို digest ကို ရယူသည်
3. Deployment manifest တွင် tag အစား digest ထည့်သွင်းသည်
4. Server က image ကို digest အတိအကျ pull လုပ်သည်

### ဥပမာ

```python
# Show how a deploy reference changes when using a digest
def build_deploy_command(image_ref):
    # Build a pull command for the given image reference
    return f"docker pull {image_ref}"

tag_ref = "ghcr.io/org/ai-service:v1.3"
digest_ref = "ghcr.io/org/ai-service@sha256:9f2a7b1c..."
print(build_deploy_command(tag_ref))
print(build_deploy_command(digest_ref))
# Expected output:
# docker pull ghcr.io/org/ai-service:v1.3
# docker pull ghcr.io/org/ai-service@sha256:9f2a7b1c...
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Model swap တစ်ခု လွဲမှားလျှင် AI service ရဲ့ ထွက်လဒ်တွေ တစ်ကယ် ပြောင်းသွားတတ်သည် — token count၊ latency၊ စာပိုဒ်အရည်အသွေး အပါအဝင်။ Digest deploy က "production မှာ တကယ် run နေတဲ့ code ဟာ CI မှာ test လုပ်ခဲ့တဲ့ code ဘက်ပဲ" ဆိုတာကို သက်သေပြနိုင်သည်။

## 4. Smoke Test နှင့် Rollback Strategy

### ဘာကို ဆိုလိုတာလဲ

Smoke test ဆိုတာ deploy ပြီးနောက် "အခြေခံက အလုပ်လုပ်နေလဲ" ဆိုတာကို စစ်သော တိုတောင်းသည့် automated test ဖြစ်သည် (ဥပမာ health endpoint၊ sample prediction တစ်ခုခေါ်ကြည့်ခြင်း)။ Rollback ကတော့ smoke test ပျက်လျှင် အရင် ကောင်းနေသော digest သို့ ပြန်တင်ခြင်း စနစ်ဖြစ်သည်။

### ဘာကြောင့် လဲ

Deploy "အောင်မြင်တယ်" လို့ ဆိုနိုင်ဖို့ အလုပ်လုပ်ကြည့်ရသည်။ Container တက်လာသည် ဟူသော အချက်တင်က မလုံလောက်။ AI service မှာ model load ဖိုင်ပျက်နေခြင်း၊ GPU config လွဲနေခြင်းများ ရှိနိုင်သဖြင့် တကယ့် request တစ်ခု ပို့ကြည့်မှ သိနိုင်သည်။ Rollback က ရှိနေလျှင် ပျက်စီးမှုကို မိနစ်ပိုင်းအတွင်း ကယ်တင်နိုင်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

1. Deploy ပြီးနောက် health endpoint (`/health`) ကို ခေါ်ကြည့်သည်
2. Sample request တစ်ခု (ဥပမာ စာကြောင်းတစ်ကြောင်း embeddings ယူခြင်း) ပို့ကြည့်သည်
3. အဆင့်တိုင်းမှာ ကန့်သတ်ချက် (timeout, response ပုံစံ) စစ်သည်
4. တစ်ခုမှ မှားလျှင် နောက်ဆုံး အောင်မြင်ခဲ့သော digest ကို deploy ပြန်သည်

### ဥပမာ

```python
import requests

def smoke_test(base_url):
    # Run a minimal post-deploy check; return list of failures
    failures = []
    try:
        health = requests.get(f"{base_url}/health", timeout=5)
        if health.status_code != 200:
            failures.append("health check failed")
    except requests.RequestException:
        failures.append("health check unreachable")

    try:
        pred = requests.post(
            f"{base_url}/predict",
            json={"text": "hello world"},
            timeout=10,
        )
        if pred.status_code != 200 or "result" not in pred.json():
            failures.append("predict endpoint malformed response")
    except requests.RequestException:
        failures.append("predict endpoint unreachable")

    return failures

# print(smoke_test("http://localhost:8000"))
# Expected output: []  (empty list when the service is healthy)
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Model တစ်ခု update လုပ်ရာတွင် အဆင့်သင့်မဖြစ်နိုင်သော အခြေအနေများ (config mismatch, incompatible tokenizer) မကြာခဏ တွေ့ရသည်။ Smoke test က ထိုပြဿနာကို အသုံးပြုသူ မသိခင် ဖမ်းဆီးပေးပြီး rollback က ရပ်တန့်ချိန်ကို တစ်ရှူးတစ်တန်း လျှော့ပေးသည်။

## အနှစ်ချုပ်

CI/CD pipeline တစ်ခုက AI service ကို build → test → scan → push (immutable tag) → deploy (digest) → smoke test → rollback ဆိုသော ကွင်းဆက်ဖြင့် လုံခြုံစွာ production တင်ပေးသည်။ Tag က လူများ ဖတ်ရလွယ်သော နာမည်ဖြစ်ပြီး digest က ပြောင်းလဲမရသော အာမခံချက်ဖြစ်သည်။ Smoke test မှာ တစ်ရက်တည်းအလုပ်လုပ်တာ မဟုတ်ဘဲ၊ rollback အစီအစဉ်လည်း လိုက်စဉ်းပါရှိမှ တကယ့် ယုံကြည်စိတ်ချမှု ရရှိမည်ဖြစ်သည်။
