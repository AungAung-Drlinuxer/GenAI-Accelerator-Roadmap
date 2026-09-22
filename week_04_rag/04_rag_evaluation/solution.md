# solution.md — Week 4: RAG Evaluation & Grounding

## လေ့ကျင့်ခန်း ၁ — Golden Question Set တည်ဆောက်ခြင်း

```python
# golden_set.py
# A minimal golden dataset: question, expected answer keywords, and relevant source ids.
golden_set = [
    {
        "id": "q001",
        "question": "What is the default chunk size in our ingestion pipeline?",
        "expected_keywords": ["chunk_size", "512"],
        "relevant_doc_ids": ["doc_12", "doc_45"],
    },
    {
        "id": "q002",
        "question": "How do you configure the vector database connection string?",
        "expected_keywords": ["VECTOR_DB_URI", "environment", "variable"],
        "relevant_doc_ids": ["doc_07"],
    },
]

# Validate the set before using it: every entry needs the required fields.
def validate_golden_set(dataset):
    required = {"id", "question", "expected_keywords", "relevant_doc_ids"}
    for item in dataset:
        missing = required - set(item.keys())
        if missing:
            raise ValueError(f"Item {item.get('id')} missing fields: {missing}")
    return True

if __name__ == "__main__":
    validate_golden_set(golden_set)
    print(f"Golden set OK: {len(golden_set)} questions")
```

**အဓိကအယူအဆ** — Golden question set သည် မေးခွန်း၊ မျှော်လင့်သောအဖြေ၏ သော့ချက်စကားလုံးများနှင့် ဆက်စပ်ရှိရမည့် document အိုင်ဒီများ ပါဝင်သော စံနှုန်းသတ်မှတ်ထားသည့် စမ်းသပ်ဒေတာအစုံဖြစ်ပြီး၊ အသုံးမပြုမီ ဖိုင်းအားလုံး ပြည့်စုံကြောင်း စစ်ဆေးရမည်။

## လေ့ကျင့်ခန်း ၂ — Recall@k, MRR, nDCG တွက်ချက်ခြင်း

```python
# retrieval_metrics.py
# Compute recall@k, MRR, and nDCG for a single query against golden labels.

def recall_at_k(retrieved_ids, relevant_ids, k):
    # Fraction of relevant documents found within the top-k results.
    top_k = retrieved_ids[:k]
    found = set(top_k) & set(relevant_ids)
    return len(found) / len(relevant_ids) if relevant_ids else 0.0

def reciprocal_rank(retrieved_ids, relevant_ids):
    # 1 / rank of the first relevant document; 0 if none is retrieved.
    for rank, doc_id in enumerate(retrieved_ids, start=1):
        if doc_id in relevant_ids:
            return 1.0 / rank
    return 0.0

def ndcg_at_k(retrieved_ids, relevant_ids, k):
    # Ideal DCG vs actual DCG, using binary relevance.
    import math
    def dcg(ids):
        total = 0.0
        for rank, doc_id in enumerate(ids[:k], start=1):
            rel = 1.0 if doc_id in relevant_ids else 0.0
            total += rel / math.log2(rank + 1)
        return total

    actual = dcg(retrieved_ids)
    ideal_ids = list(relevant_ids)
    ideal = dcg(ideal_ids)
    return actual / ideal if ideal > 0 else 0.0

if __name__ == "__main__":
    retrieved = ["doc_01", "doc_07", "doc_12", "doc_45", "doc_99"]
    relevant = ["doc_12", "doc_45"]
    print("recall@3 =", recall_at_k(retrieved, relevant, 3))
    print("MRR      =", reciprocal_rank(retrieved, relevant))
    print("nDCG@5   =", ndcg_at_k(retrieved, relevant, 5))
```

**အဓိကအယူအဆ** — recall@k သည် ဆက်စပ် document များအနက် top-k အတွင်း ဘယ်နှစ်ခု ရှာတွေ့သည်ကို တိုင်းတာပြီး၊ MRR သည် ပထမဆုံးဆက်စပ် document ၏ အဆင့်နေရာ၊ nDCG သည် ဆက်စပ် document များ အပေါ်ဘက်အဆင့်များတွင် ရှိနေမှုကို အာရုံစိုက်သည့် retrieval တိုင်းတာမှုသုံးမျိုးဖြစ်သည်။

## လေ့ကျင့်ခန်း ၃ — Faithfulness နှင့် Citation Check

```python
# faithfulness.py
# Check that every claim in the answer is supported by the retrieved context
# and that every citation points to a real retrieved chunk.

def simple_faithfulness_check(answer, contexts):
    # Naive approach: split the answer into sentences, then verify each
    # sentence shares at least one meaningful token with some context chunk.
    import re
    sentences = [s.strip() for s in re.split(r"[။.!?\n]", answer) if s.strip()]  # intentional: Burmese sentence punctuation
    supported = 0
    for sentence in sentences:
        tokens = set(re.findall(r"\w+", sentence.lower()))
        matched = any(tokens & set(re.findall(r"\w+", ctx.lower())) for ctx in contexts)
        if matched:
            supported += 1
    return supported / len(sentences) if sentences else 0.0

def citation_check(cited_ids, retrieved_ids):
    # Every cited document must actually exist in the retrieved set.
    invalid = [cid for cid in cited_ids if cid not in retrieved_ids]
    return {"valid": len(invalid) == 0, "invalid_citations": invalid}

if __name__ == "__main__":
    contexts = [
        "The chunk_size parameter defaults to 512 tokens in the pipeline.",
        "Set VECTOR_DB_URI as an environment variable to configure the connection.",
    ]
    answer = "The default chunk_size is 512. You must restart the cluster nightly."
    cited = ["doc_12"]
    retrieved = ["doc_12", "doc_45"]
    print("faithfulness:", simple_faithfulness_check(answer, contexts))
    print("citations:", citation_check(cited, retrieved))
```

**အဓိကအယူအဆ** — Faithfulness ဆိုသည်မှာ ဖြေကြားချက်ထဲရှိ ထောက်ခံချက်တိုင်းသည် retrieved context တွင် အခြေခံရှိကြောင်း စစ်ဆေးခြင်းဖြစ်ပြီး၊ citation check သည် ဖြေကြားချက်က ကိုးကားသော document များ အမှန်တကယ် retrieve ရရှိထားသည့်အစုံတွင် ပါဝင်ကြောင်း အတည်ပြုခြင်းဖြစ်သည်။

## လေ့ကျင့်ခန်း ၄ — CI Regression Gate

```python
# regression_gate.py
# Run evaluation on the golden set and fail (exit code 1) if any metric
# drops below its threshold. Suitable for a CI pipeline step.

import sys

THRESHOLDS = {"recall_at_3": 0.80, "mrr": 0.70, "faithfulness": 0.85}

def run_rag_pipeline(question):
    # Placeholder: replace with a real retriever + generator call.
    return {
        "retrieved_ids": ["doc_01", "doc_07", "doc_12"],
        "answer": "The chunk_size defaults to 512 tokens.",
        "cited_ids": ["doc_12"],
    }

def evaluate(golden_set):
    from retrieval_metrics import recall_at_k, reciprocal_rank
    from faithfulness import simple_faithfulness_check, citation_check
    scores = {"recall_at_3": [], "mrr": [], "faithfulness": []}
    for item in golden_set:
        result = run_rag_pipeline(item["question"])
        scores["recall_at_3"].append(
            recall_at_k(result["retrieved_ids"], item["relevant_doc_ids"], 3))
        scores["mrr"].append(
            reciprocal_rank(result["retrieved_ids"], item["relevant_doc_ids"]))
        # Faithfulness assumes contexts come from the retriever in production.
        scores["faithfulness"].append(1.0 if citation_check(
            result["cited_ids"], result["retrieved_ids"])["valid"] else 0.0)
    return {name: sum(vals) / len(vals) for name, vals in scores.items()}

if __name__ == "__main__":
    from golden_set import golden_set
    results = evaluate(golden_set)
    failed = False
    for name, value in results.items():
        status = "PASS" if value >= THRESHOLDS[name] else "FAIL"
        if status == "FAIL":
            failed = True
        print(f"{name}: {value:.3f} (threshold {THRESHOLDS[name]:.2f}) -> {status}")
    sys.exit(1 if failed else 0)
```

**အဓိကအယူအဆ** — Regression gate ဆိုသည်မှာ CI အတွင်းတွင် golden set အပေါ် RAG pipeline ကို အလိုအလျောက်အကဲဖြတ်ပြီး၊ recall@k သို့မဟုတ် faithfulness ကဲ့သို့သော မက်ထရစ်တစ်ခုခု သတ်မှတ်ထားသော အနိမ့်ဆုံးတန်ဖိုးအောက် ကျဆင်းပါက build ကို ပျက်စေခြင်းဖြင့် ထည့်သွင်းမှုအရည်အသွေး ကျဆင်းသွားမှုကို အမြန်ဖမ်းဆုပ်ပေးသည့် စက်ရုပ်စနစ်ဖြစ်သည်။

## လေ့ကျင့်ခန်း ၅ — Hallucination Mitigation

```python
# hallucination_mitigation.py
# Mitigation layers: context-first prompting, citation enforcement,
# and a refusal rule when no relevant context is retrieved.

def build_prompt(question, contexts):
    # Instruct the model to answer ONLY from the provided context.
    context_text = "\n".join(f"[{i}] {c}" for i, c in enumerate(contexts, start=1))
    return (
        "Answer the question using ONLY the context below.\n"
        "Cite the bracket number of each source you use.\n"
        "If the context does not contain the answer, reply exactly: I don't know.\n\n"
        f"Context:\n{context_text}\n\nQuestion: {question}"
    )

def postprocess_answer(answer, contexts):
    # Guard 1: refuse if the model admits it cannot answer.
    if "i don't know" in answer.lower():
        return {"answer": "I don't know.", "citations": []}
    # Guard 2: strip any sentence with no citation marker and no context overlap.
    import re
    kept = []
    for sentence in re.split(r"(?<=[.!?])\s+", answer):
        has_citation = bool(re.search(r"\[\d+\]", sentence))
        overlaps = any(
            set(re.findall(r"\w+", sentence.lower()))
            & set(re.findall(r"\w+", ctx.lower()))
            for ctx in contexts)
        if has_citation and overlaps:
            kept.append(sentence)
    return {
        "answer": " ".join(kept) if kept else "I don't know.",
        "citations": sorted(set(re.findall(r"\[(\d+)\]", " ".join(kept)))),
    }

if __name__ == "__main__":
    prompt = build_prompt(
        "What is the default chunk size?",
        ["The chunk_size parameter defaults to 512 tokens."],
    )
    print(prompt)
    print(postprocess_answer(
        "The default chunk_size is 512 [1]. Unicorns also store vectors [2].",
        ["The chunk_size parameter defaults to 512 tokens."]))
```

**အဓိကအယူအဆ** — Hallucination လျှော့ချရန်အတွက် ဖြေကြားချက်ကို context အပေါ်သာ အခြေခံစေသော prompt၊ မဖြစ်မနေ citation တပ်စေခြင်း၊ ဆက်စပ် context မရှိပါက ငြင်းပယ်စေသော စည်းကမ်းနှင့် ကိုးကားချက်မရှိသော ဝါကျများကို ဖယ်ရှားသော post-processing စသည့် ကာကွယ်မှုအလွှာများ ပေါင်းစပ်အသုံးပြုရသည်။

## လေ့ကျင့်ခန်း ၆ — CI Regression Gate တည်ဆောက်ခြင်း

```python
import sys
import json

def load_scores(path):
    # Load current evaluation scores from a JSON file produced by the eval step
    with open(path, "r", encoding="utf-8") as f:
        return json.load(f)

def evaluate_and_gate(scores, thresholds):
    # scores: dict of metric name -> current score
    # thresholds: dict of metric name -> minimum acceptable score
    failures = []
    for metric, threshold in thresholds.items():
        if scores.get(metric, 0.0) < threshold:
            failures.append(
                f"FAIL: {metric}={scores.get(metric, 0.0):.2f} < {threshold:.2f}"
            )
    return failures

if __name__ == "__main__":
    # Current scores could come from an eval artifact written earlier in the pipeline
    current_scores = {
        "recall_at_5": 0.85,
        "mrr": 0.72,
        "faithfulness": 0.90
    }
    # Thresholds are set from measured baselines, not from guesses
    thresholds = {"recall_at_5": 0.80, "mrr": 0.70, "faithfulness": 0.85}

    failures = evaluate_and_gate(current_scores, thresholds)
    if failures:
        for msg in failures:
            print(msg)
        sys.exit(1)  # Non-zero exit code fails the CI pipeline
    print("All quality gates passed.")
```

**အဓိကအယူအဆ** — Golden set အပေါ် တိုင်းတာထားသော baseline နှုန်းများမှ သတ်မှတ်ထားသည့် threshold များအောက် အမှတ်များ ကျဆင်လာပါက non-zero exit code ထုတ်ပေးခြင်းဖြင့် CI pipeline အလိုအလျောက် ရပ်တန့်ပြီး regression များကို production သို့ မရောက်မီ ဖမ်းဆီးနိုင်သည်။
