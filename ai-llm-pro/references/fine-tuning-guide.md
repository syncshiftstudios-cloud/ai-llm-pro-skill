# Fine-tuning Guide

## What is Fine-tuning?

Fine-tuning = taking a **pretrained LLM** and training it further on **your
specific data** to make it behave in a specific way.

```
Base Model (GPT-4o, Llama, Gemini)
        +
Your Custom Dataset (1000s of examples)
        =
Fine-tuned Model (specialized for your use case)
```

---

## Fine-tuning vs Prompt Engineering

> ⚠️ **Always try prompt engineering first!**
> Fine-tuning is expensive, slow, and hard to update.

| Factor | Prompt Engineering | Fine-tuning |
|--------|-------------------|-------------|
| **Cost** | Low (just API calls) | High (GPU training) |
| **Speed to implement** | Minutes | Days/weeks |
| **Data needed** | 0 examples | 100s to 1000s |
| **Easy to update** | ✅ Yes | ❌ Retrain needed |
| **Knowledge cutoff** | Limited | Can teach new facts |
| **Style control** | Good | Excellent |
| **Consistency** | Moderate | Very high |

---

## When SHOULD You Fine-tune?

Fine-tune when:

1. **Style/Tone consistency** — You need the model to always sound like your brand
2. **Proprietary format** — Output must match a very specific format (e.g., your internal JSON schema)
3. **Latency** — You need to move instructions from prompt → weights to reduce token usage
4. **Privacy** — You can't send sensitive examples in every prompt
5. **Prompt engineering maxed out** — You've tried everything and still getting inconsistent results
6. **Specific domain** — Medical, legal, coding in niche language — where base model is weak

---

## Fine-tuning Data Requirements

### Dataset Format (OpenAI)
```jsonl
{"messages": [{"role": "system", "content": "You are a customer support agent for TechCorp."}, {"role": "user", "content": "My order hasn't arrived"}, {"role": "assistant", "content": "I'm sorry to hear that! Let me look into your order right away. Could you please share your order ID?"}]}
{"messages": [{"role": "system", "content": "You are a customer support agent for TechCorp."}, {"role": "user", "content": "How do I reset my password?"}, {"role": "assistant", "content": "To reset your password, go to Settings → Security → Reset Password. You'll receive an email within 2 minutes."}]}
```

### How Many Examples Do You Need?

| Use Case | Minimum | Recommended |
|----------|---------|-------------|
| Style/tone change | 50-100 | 200-500 |
| New task type | 100-200 | 500-1000 |
| Domain specialization | 500+ | 1000-5000 |
| Replacing RAG | 1000+ | 5000-10000+ |

### Data Quality Rules
- ✅ Quality > Quantity — 100 perfect examples > 1000 bad ones
- ✅ Diverse — cover all edge cases
- ✅ Consistent — same style/format throughout
- ✅ Representative — matches real production inputs
- ❌ No duplicates
- ❌ No hallucinations in training data

---

## Fine-tuning Platforms

### OpenAI Fine-tuning
**Models:** GPT-4o-mini, GPT-3.5-turbo
**Best for:** Production apps, easy setup

```python
from openai import OpenAI

client = OpenAI()

# Upload training file
with open("training_data.jsonl", "rb") as f:
    file = client.files.create(file=f, purpose="fine-tune")

# Start fine-tuning job
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-4o-mini"
)

print(f"Job ID: {job.id}")

# Check status
job_status = client.fine_tuning.jobs.retrieve(job.id)
print(f"Status: {job_status.status}")
print(f"Fine-tuned model: {job_status.fine_tuned_model}")
```

**Cost:** ~\$3-8 per million tokens for training

---

### Hugging Face + PEFT (LoRA)
**Models:** Any open-source model (Llama, Mistral, Phi)
**Best for:** Full control, privacy, one-time training cost

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType
from trl import SFTTrainer, SFTConfig

# Load base model
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.1-8B")

# LoRA config — trains only small adapter layers (memory efficient)
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,           # Rank — higher = more capacity but more memory
    lora_alpha=32,  # Scaling factor
    target_modules=["q_proj", "v_proj"],  # Which layers to train
    lora_dropout=0.1
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Output: trainable params: 6,815,744 || all params: 8,037,421,312 || trainable%: 0.08%

# Train
trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=your_dataset,
    args=SFTConfig(
        output_dir="./fine-tuned-model",
        num_train_epochs=3,
        per_device_train_batch_size=4,
        learning_rate=2e-4,
    )
)

trainer.train()
trainer.save_model()
```

---

### Google Vertex AI Fine-tuning
**Models:** Gemini 1.5 Flash, PaLM 2
**Best for:** Google Cloud users

```python
from google.cloud import aiplatform

aiplatform.init(project="your-project", location="us-central1")

job = aiplatform.CustomJob.from_local_script(
    display_name="gemini-fine-tune",
    script_path="fine_tune.py",
    container_uri="gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest",
    requirements=["google-cloud-aiplatform[vertex]"],
    machine_type="n1-standard-8",
    accelerator_type="NVIDIA_TESLA_V100",
    accelerator_count=1,
)
job.run()
```

---

## LoRA vs Full Fine-tuning

| Method | Memory | Speed | Quality | When to Use |
|--------|--------|-------|---------|-------------|
| **Full Fine-tuning** | Very High (80GB+ GPU) | Slow | Best | You have 100B+ param model & budget |
| **LoRA** | Medium (16-24GB GPU) | Fast | Great | Most use cases ✅ |
| **QLoRA** | Low (8-12GB GPU) | Medium | Good | Limited GPU memory |
| **Prefix Tuning** | Low | Fast | Moderate | Style/tone changes |

**Recommendation for most people:** Use **LoRA** or **QLoRA** with Llama/Mistral

---

## Evaluation — How to Know if Fine-tuning Worked?

### Automatic Metrics
```python
# ROUGE score for text generation
from rouge_score import rouge_scorer

scorer = rouge_scorer.RougeScorer(['rouge1', 'rougeL'])
scores = scorer.score(
    target="Expected output text",
    prediction="Model generated text"
)

# For classification tasks
from sklearn.metrics import accuracy_score, f1_score
accuracy = accuracy_score(y_true, y_pred)
```

### Human Evaluation (Most Important)
Create an eval set of 50-100 examples that were NOT in training data:
- [ ] Is the output format correct?
- [ ] Is the tone/style matching?
- [ ] Are there hallucinations?
- [ ] Does it handle edge cases?
- [ ] Is it better than the base model?

### A/B Testing in Production
- Run base model and fine-tuned model in parallel
- Compare user satisfaction, task completion rate
- Use shadow mode — log both outputs, evaluate offline

---

## Fine-tuning Checklist

**Before starting:**
- [ ] Tried prompt engineering and it's not enough
- [ ] Have 100+ high quality examples (500+ recommended)
- [ ] Data is cleaned and deduplicated
- [ ] Eval set is separate from training set (80/20 split)
- [ ] Budget approved (compute costs)

**During training:**
- [ ] Monitor training loss — should decrease steadily
- [ ] Check validation loss — if it diverges, you're overfitting
- [ ] Save checkpoints every few epochs

**After training:**
- [ ] Run eval set → compare with base model
- [ ] Test edge cases manually
- [ ] Check for regressions on general tasks
- [ ] Deploy gradually (canary release — 5% traffic first)

---

## Common Fine-tuning Mistakes

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Too little data | Add more examples or use few-shot prompting instead |
| Low quality data | Clean data manually, remove bad examples |
| Overfitting | Reduce epochs, add dropout, more diverse data |
| Catastrophic forgetting | Use LoRA (preserves base model weights) |
| Wrong metric | Align eval metric with actual use case |
| No baseline | Always compare with base model + best prompt |
