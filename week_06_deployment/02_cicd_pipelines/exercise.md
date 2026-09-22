# လေ့ကျင့်ခန်းများ — CI/CD Pipelines for AI Services

## လေ့ကျင့်ခန်း ၁ — Pipeline stage validator

Pipeline stage အမည်များ (`build`, `test`, `scan`, `push`, `deploy`) ကို လက်ခံပြီး stage တစ်ခုချင်းရှိ/မရှိကို စစ်သော Python function `validate_stages(stages)` ရေးပါ။ ပျက်နေသော stage များကို list ဖြင့် ပြန်ပါ။

**Hints:** required set တစ်ခုနှင့် ပေးထားသော list ကို set အဖြစ် ပြောင်း၍ နှိုင်းယှဉ်ပါ။

**Expected behavior:** `validate_stages(["build", "test"])` → `["scan", "push", "deploy"]`

## လေ့ကျင့်ခန်း ၂ — Immutable tag generator

Git commit SHA တစ်ခုကို လက်ခံ၍ `ai-service:v<version>-<sha>` ပုံစံဖြင့် immutable tag ထုတ်ပေးသော `make_tag(version, sha)` function ရေးပါ။

**Hints:** f-string ဖြင့် စာလုံးများကို ဆက်စပ်ပါ။

**Expected behavior:** `make_tag("1.3", "abc1234")` → `ai-service:v1.3-abc1234`

## လေ့ကျင့်ခန်း ၃ — Tag vs digest reference checker

Image reference string တစ်ခုကို လက်ခံပြီး digest reference (`@sha256:` ပါသည်) လား tag reference လားဆိုတာကို ပြန်သော `classify_ref(ref)` function ရေးပါ။

**Hints:** `"@sha256:" in ref` ဖြင့် စစ်နိုင်ပြီး `"tag"` သို့မဟုတ် `"digest"` ကို return ပါ။

**Expected behavior:** `classify_ref("org/app:v1@sha256:deadbeef")` → `"digest"`၊ `classify_ref("org/app:v1")` → `"tag"`

## လေ့ကျင့်ခန်း ၄ — Smoke test runner

`base_url` လက်ခံ၍ `/health` endpoint ကို ခေါ်ပြီး status code `200` ဖြစ်မဖြစ် စစ်သော `smoke_health(base_url, session)` function ရေးပါ။ `session` က `get` method ပါသော object ဖြစ်သည် (dependency injection အလေ့အကျ)။ အောင်လျှင် `True` ပြန်ပါ။

**Hints:** fake session class တစ်ခု ရေး၍ စမ်းနိုင်သည်။ Exception ကို ဖမ်းပြီး `False` ပြန်ပါ။

**Expected behavior:** 200 ပြန်သော fake session ဖြင့် → `True`၊ 500 ပြန်လျှင် → `False`

## လေ့ကျင့်ခန်း ၅ — Rollback target chooser

လက်ရှိ digest နှင့် ယခင် အောင်မြင်ခဲ့သော digest များ list ကို လက်ခံပြီး rollback လုပ်ရမည့် digest ကို ရွေးပေးသော `choose_rollback(current, history)` function ရေးပါ။ လက်ရှိ digest ပါဝင်ပြီးသား history ထဲက နောက်ဆုံး item ကို ရွေးပါ။ History အလွတ်ဖြစ်လျှင် `None` ပြန်ပါ။

**Hints:** `history[-1]` ကို အသုံးပြုပြီး empty list ကို ဦးစွာ စစ်ပါ။

**Expected behavior:** `choose_rollback("d3", ["d1", "d2"])` → `"d2"`

## လေ့ကျင့်ခန်း ၆ — Pipeline runner simulation

Stages list တစ်ခုကို အစီအစဉ်အတိုင်း လည်ပတ်စေသော `run_pipeline(stages, fail_at=None)` function ရေးပါ။ `fail_at` တွင် stage နာမည်ထည့်ပါက ထို stage မတိုင်ခင် ရပ်ပြီး ဘယ် stage ထိ အောင်မြင်ကြောင်း list ဖြင့် ပြန်ပါ။

**Hints:** loop တစ်ခုဖြင့် stages ကို လည်ပြီး `fail_at` နှင့် တွဲစစ်ပါ။

**Expected behavior:** `run_pipeline(["build", "test", "deploy"], fail_at="deploy")` → `["build", "test"]`
