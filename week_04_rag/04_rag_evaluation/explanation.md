# Week 4 — RAG Evaluation & Grounding

## Golden Question Sets

### ဘာကို ဆိုလိုတာလဲ

Golden question set ဆိုသည်မှာ သင့် RAG system အတွက် လူသားများက ကြိုတင်ပြင်ဆင်ထားသော "မေးခွန်း + မှန်ကန်သောအဖြေ + မှန်ကန်စွာ cite လုပ်သင့်သော source document" စုစည်းမှုဖြစ်သည်။ ဥပမာ — သင့် company documentation အပေါ် အခြေခံ၍ "Product X ရဲ့ refund policy က ဘာလဲ" ဟူသောမေးခွန်းနှင့် အဖြေပါဝင်သည့် document chunk တို့ကို ဖန်တီးထားခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ

RAG system ကို "ဖြေပြီးရင် ကောင်းနေလား" ဟူ၍ ချင့်ချိန်ရန် စံနှုန်းတစ်ခု လိုအပ်သည်။ Golden set မရှိပါက retrieval က ကောင်းပြီး၊ မကောင်းပြီးဆိုသည်ကို ဆိုရာရောက်စွာ တိုင်းတာရန် မဖြစ်နိုင်ပေ။ Langfuse ကဲ့သို့သော public evaluation guides များတွင်လည်း evaluation ကို စတင်ရန် ပထမဆုံး လိုအပ်သည်မှာ curated dataset ဖြစ်သည်ဟု ဖော်ပြထားသည်။

### ဘယ်လို အလုပ်လုပ်လဲ

၁။ အရေးကြီးသောမေးခွန်းများကို စုဆောင်းရန် (user logs, FAQ, domain experts များထံမှ)။
၂။ တစ်ချင့်ချင့်အတွက် မှန်ကန်သောအဖြေနှင့် ထိုအဖြေရှိသင့်သော document ID များကို လက်ဖြင့် ရေးသွင်းရန်။
၃။ JSON/CSV ပုံစံဖြင့် သိမ်းဆည်းပြီး CI pipeline ထဲတွင် အလိုအလျောက် တစ်ဆင့်ချိန်းနိုင်ရန် ပြင်ဆင်ရန်။ Golden set ကို ပုံမှန် စစ်ဆေးပြီး နောက်ဆုံး အချက်အလက်များနှင့် ကိုက်ညီနေစေရန် လိုအပ်သည်။

### ဥပမာ

```python
import json

# A small golden question set stored as a JSON structure
golden_set = [
    {
        "question": "What is the refund window for Product X?",
        "expected_answer": "30 days from purchase date",
        "relevant_doc_ids": ["doc_12", "doc_15"],
    },
    {
        "question": "How do I enable two-factor authentication?",
        "expected_answer": "Enable it in Settings > Security",
        "relevant_doc_ids": ["doc_42"],
    },
]

# Save to disk so the evaluation pipeline can load it
with open("golden_set.json", "w", encoding="utf-8") as f:
    json.dump(golden_set, f, indent=2, ensure_ascii=False)

print(f"Saved {len(golden_set)} golden questions.")
# Expected output: Saved 2 golden questions.
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Production တွင် model, embedding, chunking strategy တို့ ပြောင်းလိုက်တိုင်း system ၏ အပြုအမူ ပြောင်းသွားနိုင်သည်။ Golden set ရှိပါက ပြောင်းလဲမှုတိုင်းကို တူညီသော စံနှုန်းဖြင့် နှိုင်းယှဉ်နိုင်သဖြင့် "ဒါပိုကောင်းသွားလား ပိုဆိုးသွားလား" ဆိုသည်ကို အချက်အလက်ဖြင့် ဆုံးဖြတ်နိုင်သည်။ ဤအဆင့်ကို ကျော်လွန်ပါက evaluation များကို ခံစားချက်အပေါ် မှုတည်ကြည့်ရမည် ဖြစ်သည်။

## Recall@k, MRR, nDCG

### ဘာကို ဆိုလိုတာလဲ

ဤသုံးခုက retrieval quality ကို တိုင်းတာသော metrics များဖြစ်သည်။ **Recall@k** — top-k retrieved documents ထဲတွင် မှန်ကန်သော document ဘယ်လောက်ပါဝင်လဲ။ **MRR (Mean Reciprocal Rank)** — ပထမဆုံး မှန်ကန်သော document ၏ အဆင့်အတန်းကို ၁/rank ဖြင့် တွက်ပြီး ပျမ်းမျှယူခြင်း။ **nDCG** — relevant document များကို အပေါ်ဆုံး၌ ထားရှိမှုကို အဆင့်အလိုက် အမှတ်ပေးခြင်း။

### ဘာကြောင့် လဲ

RAG ၌ အဖြေကောင်းရန် အရင်ဆုံး မှန်ကန်သော document ကို ရှာတွေ့ရန် လိုသည်။ Generator model မည်မျှကောင်းနေစေကာမူ မှားယွင်းသော context သာ ရရှိပါက အဖြေလည်း မှားနိုင်သည်။ ထို့ကြောင့် retrieval ကို သီးသန့် တိုင်းတာနိုင်ရန် ဤ metrics များ လိုအပ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

Recall@k သည် "မှန်တာ ရှိသလား" ကို ကြည့်ပြီး၊ MRR နှင့် nDCG တို့က "မှန်တာ ဘယ်နေရာမှာရှိလဲ" ကို ကြည့်သည်။ nDCG တွင် ideal ranking နှင့် နှိုင်းယှဉ်၍ 0 မှ 1 ကြား တန်ဖိုး ထုတ်ပေးသည် — 1 ဖြစ်ပါက perfect ranking ဟူ၍ အဓိပ္ပာယ်ရသည်။

### ဥပမာ

```python
def recall_at_k(retrieved_ids, relevant_ids, k):
    # Fraction of relevant documents found within the top-k results
    top_k = set(retrieved_ids[:k])
    hits = top_k & set(relevant_ids)
    return len(hits) / len(relevant_ids)

def reciprocal_rank(retrieved_ids, relevant_ids):
    # 1/rank of the first relevant document, 0 if none found
    for rank, doc_id in enumerate(retrieved_ids, start=1):
        if doc_id in relevant_ids:
            return 1.0 / rank
    return 0.0

retrieved = ["doc_99", "doc_42", "doc_12", "doc_15"]
relevant = ["doc_12", "doc_42"]

print("Recall@2:", recall_at_k(retrieved, relevant, 2))
print("Reciprocal rank:", reciprocal_rank(retrieved, relevant))
# Expected output:
# Recall@2: 0.5
# Reciprocal rank: 0.5
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Embedding model ကို ပြောင်းလိုက်သောအခါ top-1 hit က ကျသွားပြီး recall က မတက်ဘဲ ရှိနိုင်သည်။ Recall@k တစ်ခုတည်းဖြင့် ဖမ်းပါက ထိုကွာခြားမှုကို မမြင်ရပေ။ ထို့ကြောင့် metrics များစွာကို တွဲ၍ကြည့်ခြင်းဖြင့် retrieval layer ၏ အားနည်းချက်ကို အတိအကျ ရှာဖွေနိုင်သည်။

## Faithfulness နှင့် Citation Checks

### ဘာကို ဆိုလိုတာလဲ

Faithfulness (groundedness) ဆိုသည်မှာ generator ၏ အဖြေသည် retrieved context အပေါ်တွင်သာ အခြေခံထားခြင်းကို ဆိုလိုသည်။ Citation check က အဖြေထဲရှိ အချက်အလက်တိုင်းကို မည်သည့် source document က ထောက်ပံ့ထားသည်ကို စစ်ဆေးခြင်းဖြစ်သည်။ ဥပမာ — အဖြေတွင် "30 days" ဟူ၍ ပါပါက `doc_12` ထဲတွင် ထိုအချက်အလက် အမှန်ရှိရမည်။

### ဘာကြောင့် လဲ

LLM သည် retrieved document များထဲ မပါဝင်သော အချက်အလက်များကို ဖန်တီးထုတ်ပေးနိုင်သည် (hallucination)။ Faithfulness score နိမ့်ပါက အဖြေများသည် မှုယောင်ဆောင်မှု ရှိနေနိုင်ခြင်း ဖြစ်သည်။ ထို့ကြောင့် အဖြေတိုင်းကို context ဖြင့် ချိန်ညှိစစ်ဆေးခြင်းသည် ယုံကြည်မှုအတွက် မဖြစ်မနေ လိုအပ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

အဖြေကို အချက်အလက်ငယ်များ (claims) အဖြစ် ခွဲထုတ်ပြီး တစ်ချင့်ချင့်ကို retrieved context တွင် ပါဝင်မှုရှိ/မရှိ စစ်ဆေးသည်။ LLM-as-a-judge နည်းဖြင့် သော်လည်းကောင်း၊ ရိုးရိုး string/keyword matching ဖြင့် သော်လည်းကောင်း အလုပ်လုပ်နိုင်သည်။ Langfuse docs တွင် evaluation template များသည် ဤပုံစံအတိုင်း အချက်အလက်ကို context ဖြင့် နှိုင်းယှဉ်စစ်ဆေးရန် အကြံပြုထားသည်။

### ဥပမာ

```python
def simple_faithfulness(answer, contexts):
    # Simple keyword-based check: each answer sentence
    # must be supported by at least one retrieved context.
    sentences = [s.strip() for s in answer.split(".") if s.strip()]
    supported = 0
    for sentence in sentences:
        # Take the words of the sentence as claims (simple heuristic)
        words = set(sentence.lower().split())
        for context in contexts:
            context_words = set(context.lower().split())
            # Overlap ratio between the sentence and the context
            if words and len(words & context_words) / len(words) >= 0.6:
                supported += 1
                break
    return supported / len(sentences) if sentences else 0.0

contexts = [
    "The refund window for Product X is 30 days from purchase date.",
    "Two-factor authentication is enabled in Settings > Security.",
]
answer = "You can get a refund within 30 days. Two-factor authentication is enabled in Settings > Security."
print("Faithfulness:", simple_faithfulness(answer, contexts))
# Expected output: Faithfulness: 1.0
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Finance, legal, medical ကဲ့သို့သော အထူးအရေးကြီးရုံ့ရုံ့သော domain များတွင် hallucinated အဖြေတစ်ခုက စီးပွားရေးနှင့် ယုံကြည်မှုဆုံးရှုံးမှု ဖြစ်စေနိုင်သည်။ Faithfulness monitoring ရှိပါက hallucination များကို production တွင် သတိပြုမိနိုင်ပြီး၊ citation check က အသုံးပြုသူအား အဖြေ၏ အခြေခံကို စစ်ဆေးနိုင်စေသည်။

## Regression Gates in CI

### ဘာကို ဆိုလိုတာလဲ

Regression gate ဆိုသည်မှာ CI pipeline (GitHub Actions စသည်) ထဲတွင် golden set အပေါ် evaluation များ အလိုအလျောက် လည်ပတ်စေပြီး၊ metrics များက သတ်မှတ်ထားသော threshold အောက် ကျဆင်းပါက build ကို ကိုင်တွယ်ခြင်းဖြစ်သည်။

### ဘာကြောင့် လဲ

Prompt, model version, retrieval parameter တစ်ခု ပြောင်းလိုက်တာနှင့် အရင်က အလုပ်လုပ်ခဲ့သော မေးခွန်းများ ပျက်စီးသွားနိုင်သည် (regression)။ CI gate မရှိပါက ယင်းပျက်စီးမှုများကို user များ တွေ့မိမှ သိရမည် ဖြစ်သည်။

### ဘယ်လို အလုပ်လုပ်လဲ

၁။ သင့် golden set အပေါ် recall@k, faithfulness စသည့် metrics များကို တွက်သော script တစ်ခု ရေးရန်။
၂။ Threshold များ သတ်မှတ်ရန် (ဥပမာ — faithfulness ≥ 0.85)။
၃။ CI တွင် script ကို လည်ပတ်စေပြီး threshold မပြည့်ပါက non-zero exit code ထုတ်၍ build ကို ရပ်တန့်စေရန်။ Langfuse ကဲ့သို့သော evaluation platforms တွင် ဤသို့ CI မှ ချိတ်ဆက်စစ်ဆေးနည်းကို ဖော်ပြထားသည်။

### ဥပမာ

```python
import json, sys

def run_eval_and_gate():
    # Load results produced by the RAG evaluation script
    with open("eval_results.json", encoding="utf-8") as f:
        results = json.load(f)

    # Define minimum quality thresholds for the release gate
    thresholds = {"recall_at_5": 0.80, "faithfulness": 0.85}
    failures = []
    for metric, threshold in thresholds.items():
        score = results.get(metric, 0.0)
        print(f"{metric}: {score:.3f} (threshold: {threshold})")
        if score < threshold:
            failures.append(metric)

    if failures:
        print(f"FAILED metrics: {', '.join(failures)}")
        sys.exit(1)  # non-zero exit code stops the CI pipeline
    print("All regression gates passed.")

results = {"recall_at_5": 0.82, "faithfulness": 0.84}
with open("eval_results.json", "w") as f:
    json.dump(results, f)
run_eval_and_gate()
# Expected output:
# recall_at_5: 0.820 (threshold: 0.8)
# faithfulness: 0.840 (threshold: 0.85)
# FAILED metrics: faithfulness
```

### လက်တွေ့မှာ ဘာကြောင့် အရေးကြီးလဲ

Quality gate များရှိပါက RAG system တွင် ပြောင်းလဲမှုတိုင်းကို confidence ဖြင့် release လုပ်နိုင်သည်။ သင့် team အတွက် "ကျွန်ုပ်တို့ရဲ့ chatbot က production မှာ ဘယ်လောက်ထိ ယုံကြည်ရလဲ" ဆိုသောမေးခွန်းကို အချက်အလက်ဖြင့် ဖြေဆိုနိုင်သည်။

## Hallucination Mitigation

### ဘာကို ဆိုလိုတာလဲ
