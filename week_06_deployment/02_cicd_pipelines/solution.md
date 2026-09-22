# Solutions — CI/CD Pipelines for AI Services

## လေ့ကျင့်ခန်း ၁ — Pipeline stage validator

```python
def validate_stages(stages):
    # Compare required stages against the provided list
    required = ["build", "test", "scan", "push", "deploy"]
    provided = set(stages)
    return [s for s in required if s not in provided]

print(validate_stages(["build", "test"]))
print(validate_stages(["build", "test", "scan", "push", "deploy"]))
# Expected output:
# ['scan', 'push', 'deploy']
# []
```

**အဓိကအယူအဆ** — Pipeline တစ်ခု ပြည့်စုံဖို့ လိုအပ်သော stage တိုင်းကို မစစ်မနေ ရှိနေဖို့ အရေးကြီးသည်။

## လေ့ကျင့်ခန်း ၂ — Immutable tag generator

```python
def make_tag(version, sha):
    # Build a unique, traceable tag from version and commit SHA
    return f"ai-service:v{version}-{sha}"

print(make_tag("1.3", "abc1234"))
print(make_tag("2.0", "ff0099a"))
# Expected output:
# ai-service:v1.3-abc1234
# ai-service:v2.0-ff0099a
```

**အဓိကအယူအဆ** — Commit SHA ပါသော tag တစ်ခုစီက code နှင့် image ကို တစ်ဆူတည်း ချိတ်ဆက်ပေးသည်။

## လေ့ကျင့်ခန်း ၃ — Tag vs digest reference checker

```python
def classify_ref(ref):
    # Detect whether an image reference pins a digest
    if "@sha256:" in ref:
        return "digest"
    return "tag"

print(classify_ref("org/app:v1@sha256:deadbeef"))
print(classify_ref("org/app:v1"))
# Expected output:
# digest
# tag
```

**အဓိကအယူအဆ** — Digest reference ဖြင့် deploy လုပ်ခြင်းက tag ထက် ပိုတိကျပြီး အကြောင်းအရာ အတိအကျကို သေချာစေသည်။

## လေ့ကျင့်ခန်း ၄ — Smoke test runner

```python
class FakeSession:
    # Minimal test double that mimics an HTTP session
    def __init__(self, status_code):
        self.status_code = status_code

    def get(self, url):
        return self

class FakeResponse:
    pass

def smoke_health(base_url, session):
    # Return True only if the health endpoint responds with 200
    try:
        resp = session.get(f"{base_url}/health")
        return resp.status_code == 200
    except Exception:
        return False

ok_session = FakeSession(200)
bad_session = FakeSession(500)

def make_session(status):
    s = FakeSession(status)
    return s

print(smoke_health("http://localhost:8000", make_session(200)))
print(smoke_health("http://localhost:8000", make_session(500)))
# Expected output:
# True
# False
```

**အဓိကအယူအဆ** — Smoke test ကို dependency injection ဖြင့် ရေးခြင်းက network မလိုဘဲ စမ်းသပ်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၅ — Rollback target chooser

```python
def choose_rollback(current, history):
    # Pick the most recent previously deployed digest
    if not history:
        return None
    return history[-1]

print(choose_rollback("d3", ["d1", "d2"]))
print(choose_rollback("d1", []))
# Expected output:
# d2
# None
```

**အဓိကအယူအဆ** — ယခင် အောင်မြင်ခဲ့သော digest များကို မှတ်တမ်းတင်ထားခြင်းက rollback ကို မိနစ်ပိုင်းအတွင်း လုပ်ဆောင်နိုင်စေသည်။

## လေ့ကျင့်ခန်း ၆ — Pipeline runner simulation

```python
def run_pipeline(stages, fail_at=None):
    # Execute stages in order; stop before the failing stage
    completed = []
    for stage in stages:
        if stage == fail_at:
            break
        completed.append(stage)
    return completed

print(run_pipeline(["build", "test", "deploy"], fail_at="deploy"))
print(run_pipeline(["build", "test", "deploy"]))
# Expected output:
# ['build', 'test']
# ['build', 'test', 'deploy']
```

**အဓိကအယူအဆ** — Stage တစ်ခု ပျက်လျှင် နောက် stages တွေ ဆက်မလည်ပတ်ရန် ရပ်တန့်တာဝန်ခံစွာ ရပ်ခြင်းက အရာမှားများ ပိုမိုကူးစက်ခြင်းကို ကာကွယ်သည်။
