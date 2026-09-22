## လေ့ကျင့်ခန်း ၁ — Golden Question Set တည်ဆောက်ခြင်း

RAG စနစ်တစ်ခုကို အမှတ်မထားတိုင်း တိုးတက်သည်ဟု မဆိုနိုင်ရန်အတွက် စံနှုန်းမေးခွန်းများ (golden questions) လိုအပ်သည်။ အောက်ပါ schema အတိုင်း JSON ဖိုင်တစ်ခုဖန်တီးပြီး မေးခွန်း ၁၀ ခု ရေးသွင်းပါ။

```python
import json

golden_set = [
    {
        "id": "q001",
        "question": "What is the default port for HTTPS?",
        "expected_answer": "443",
        "expected_sources": ["docs/networking.md"]
    }
]

with open("golden_set.json", "w", encoding="utf-8") as f:
    # Save the golden set as JSON for reuse in later exercises
    json.dump(golden_set, f, indent=2, ensure_ascii=False)
```

**Hints:** မေးခွန်းများကို သင်၏ knowledge base ထဲရှိ စာရွက်စာတမ်းအတိအကျ ဖြေဆိုနိုင်သော အကြောင်းအရာများမှ ရွေးချယ်ပါ။ `expected_sources` သည် နောက်ပိုင်း retrieval တိုင်းတာမှုများအတွက် အရေးကြီးသည်။

**Expected behavior:** `golden_set.json` ဖိုင် တည်ဆောက်ပြီး JSON format မှန်ကန်စွာ ဖွင့်ဖတ်နိုင်သည်။

## လေ့ကျင့်ခန်း ၂ — Recall@k တွက်ချက်ခြင်း

Retrieval ရလဒ်များကို golden sources နှင့် နှိုင်းယှဉ်၍ recall@k တွက်ပါ။

```python
def recall_at_k(retrieved_docs, relevant_docs, k):
    # retrieved_docs: list of doc names returned by the retriever
    # relevant_docs: list of doc names that should be retrieved
    top_k = retrieved_docs[:k]
    hits = sum(1 for doc in relevant_docs if doc in top_k)
    return hits / len(relevant_docs) if relevant_docs else 0.0

retrieved = ["networking.md", "security.md", "api.md", "intro.md"]
relevant = ["networking.md", "security.md"]
print(recall_at_k(retrieved, relevant, 3))
```

**Hints:** k = 3 ထဲတွင် relevant documents ၂ ခုလုံး ပါဝင်ပါက ရလဒ်မှာ 1.0 ဖြစ်မည်။ k တန်ဖိုးပြောင်း၍ ကွာခြားချက်ကို လေ့လာပါ။

**Expected behavior:** Function သည် 0.0 မှ 1.0 ကြားရှိ float တစ်ခု ပြန်ပေးပြီး ဖမ်းမိမှုနှုန်းကို တိကစွာ ဖော်ပြသည်။

## လေ့ကျင့်ခန်း ၃ — MRR တွက်ချက်ခြင်း

Mean Reciprocal Rank (MRR) ဖြင့် ပထမဆုံး မှန်ကန်သော ရလဒ်၏ အနိမ့်အမြင့်ကို တိုင်းတာပါ။

```python
def reciprocal_rank(ranked_docs, relevant_doc, k):
    # Find the rank position of the first relevant document
    for i, doc in enumerate(ranked_docs[:k], start=1):
        if doc == relevant_doc:
            return 1.0 / i
    return 0.0

def mrr(queries, k=5):
    # queries: list of (ranked_docs, relevant_doc) tuples
    scores = [reciprocal_rank(docs, rel, k) for docs, rel in queries]
    return sum(scores) / len(scores) if scores else 0.0
```

**Hints:** မှန်ကန်သော document သည် ပထမနေရာတွင် ပါဝင်လျှင် RR = 1.0၊ ဒုတိယနေရာတွင် ပါဝင်လျှင် RR = 0.5 ဖြစ်သည်။ MRR သည် ranking quality ကို recall ထက် ပိုမှန်းဆိုပြသည်။

**Expected behavior:** MRR တန်ဖိုးသည် 0 မှ 1 ကြားတွင်ရှိပြီး ကွာခြားသော queries များကို ပျမ်းမျှကာ ပြန်ပေးသည်။

## လေ့ကျင့်ခန်း ၄ — Faithfulness စစ်ဆေးခြင်း (Claim ခွဲခြားခြင်း)

LLM ၏ အဖြေတစ်ခုကို သီးသန့် claims များအဖြစ် ခွဲခြားပြီး ရှာဖွေရလဒ်များထဲတွင် အထောက်အထားရှိမရှိ စစ်ဆေးပါ။

```python
def split_claims(answer):
    # Naive claim splitting by sentence for this exercise
    sentences = [s.strip() for s in answer.split(".") if s.strip()]
    return sentences

def check_grounded(claim, retrieved_chunks):
    # Check if any retrieved chunk shares enough keyword overlap
    claim_words = set(claim.lower().split())
    for chunk in retrieved_chunks:
        chunk_words = set(chunk.lower().split())
        overlap = claim_words & chunk_words
        if len(overlap) >= 3:
            return True
    return False

faithful_claims = sum(
    1 for c in claims if check_grounded(c, chunks)
)
score = faithful_claims / len(claims)
```

**Hints:** Keyword overlap သည် ရိုးရိုးစပ်စပ်နည်းလမ်းသာ ဖြစ်သည်။ တကယ့် production တွင် LLM-as-judge သို့မဟုတ် embedding similarity ကို အသုံးပြုနိုင်သည် (langfuse.com/docs ၏ RAG evaluation လမ်းညွှန်တွင် ကြည့်ပါ)။

**Expected behavior:** Faithfulness score သည် 0.0 မှ 1.0 ကြားရှိပြီး အထောက်အထားမရှိသော claims များကို ဖော်ပြနိုင်သည်။

## လေ့ကျင့်ခန်း ၅ — Citation စစ်ဆေးခြင်းနှင့် Hallucination ဖမ်းရှာခြင်း

အဖြေတွင် ကိုးကားထားသော sources များ တကယ်ရှိမရှိ၊ အသုံးပြုခဲ့သလားကို စစ်ဆေးပါ။

```python
def validate_citations(answer_citations, retrieved_sources):
    # answer_citations: sources the model claims to have used
    # retrieved_sources: sources actually provided to the model
    valid = [c for c in answer_citations if c in retrieved_sources]
    invalid = [c for c in answer_citations if c not in retrieved_sources]
    return valid, invalid

def hallucination_risk(answer, valid_citations):
    # Flag answers that lack any valid citation
    if len(answer) > 100 and not valid_citations:
        return "HIGH"
    return "LOW"
```

**Hints:** Model က မရှိလောက်သော ဖိုင်နာမည်တစ်ခုကို ကိုးကားနိုင်သည် — ၎င်းသည် hallucination ၏ ပုံစံတစ်မျိုးဖြစ်သည်။ `invalid` list ကို မှတ်တမ်းတင်၍ ပြင်ဆင်ရန် signal အဖြစ် အသုံးပြုပါ။

**Expected behavior:** မှားယွင်းသော citations များကို `invalid` list တွင် ဖမ်းမိပြီး ရှည်လျားသော ကိုးကားမှုကင်းသော အဖြေများကို HIGH risk အဖြစ် အမှတ်အသားပြုသည်။

## လေ့ကျင့်ခန်း ၆ — CI Regression Gate တည်ဆောက်ခြင်း

Golden set အပေါ် အမှတ်များ ကျဆင်းလာပါက pipeline အတွင်း အလိုအလျောက် အသိပေးသည့် regression gate တစ်ခု ရေးပါ။

```python
import sys

def evaluate_and_gate(scores, thresholds):
    # scores: dict of metric name -> current score
    # thresholds: dict of metric name -> minimum acceptable score
    failures = []
    for metric, threshold in thresholds.items():
        if scores.get(metric, 0.0) < threshold:
            failures.append(
                f"FAIL: {metric}={scores[metric]:.2f} < {threshold:.2f}"
            )
    return failures

if __name__ == "__main__":
    current_scores = {
        "recall_at_5": 0.85,
        "mrr": 0.72,
        "faithfulness": 0.90
    }
    thresholds = {"recall_at_5": 0.80, "mrr": 0.70, "faithfulness": 0.85}
    failures = evaluate_and_gate(current_scores, thresholds)
    if failures:
        for msg in failures:
            print(msg)
        sys.exit(1)  # Non-zero exit code fails the CI pipeline
    print("All quality gates passed.")
```

**Hints:** Non-zero exit code သည် CI စနစ် (GitHub Actions ကဲ့သို့) အတွင်း step ကို အောင်မြင်မှုမရှိဟု သတ်မှတ်စေသည်။ Thresholds များကို baseline တိုင်းတာမှုများ ရရှိပြီးမှ သတ်မှတ်ပါ — ကြိုတင်ယူဆောင်းထားသော နှုန်းထားများ မသတ်မှတ်ပါနှင့်။

**Expected behavior:** အမှတ်အားလုံး threshold ထက် မြင့်လျှင် "All quality gates passed." ပြပြီး exit code 0 ပြန်သည်၊ တစ်ခုခု ကျဆင်းလျှင် FAIL message များ ပြပြီး exit code 1 ပြန်သည်။
