---
layer: 05_ai_engineering
type: index
status: evergreen
created: 2026-03-02
---
# AI Engineering

## Purpose of This Layer

This layer captures the engineering discipline required to build applications on top of pretrained foundation models.

It is aligned with the conceptual framework of modern AI engineering as structured in:
- Foundation models
- Evaluation
- Prompting
- RAG and agents
- Finetuning
- Dataset engineering
- Inference optimization
- System architecture and user feedback

Guiding question:

> How do we design, evaluate, optimize, and operate systems built around foundation models?

This layer does NOT cover:
- General ML pipelines (→ 04_ml_engineering)
- Classical model selection and structured ML (→ 02_modeling)
- Mathematical derivations (→ 01_foundations)

---

## Structure

### 00_foundation_models/

Understanding the underlying models.

Includes:
- Transformer architecture overview
- Scaling laws
- Sampling and decoding
- SFT / RLHF alignment
- Model selection trade-offs

Focus:
Capabilities and constraints of foundation models.

---

### 01_evaluation/

Evaluation methodology for open-ended AI systems.

Includes:
- Language modeling metrics
- Exact vs subjective evaluation
- AI-as-a-judge
- Comparative evaluation
- Evaluation pipeline design
- Benchmark limitations

Focus:
How to systematically evaluate generative systems.

---

### 02_prompt_engineering/

Controlling models via inputs.

Includes:
- Prompt anatomy
- In-context learning
- Structured outputs
- Reasoning prompts
- Prompt injection risks
- Guardrails

Focus:
Human–AI interaction design.

---

### 03_rag_and_agents/

Architectures that extend foundation models.

Includes:
- RAG systems
- Retriever design
- Hybrid search
- Agent planning
- Tool use
- Memory systems

Focus:
System-level augmentation of foundation models.

---

### 04_finetuning/

Modifying model weights.

Includes:
- When to finetune vs RAG
- Full finetuning
- PEFT / LoRA
- Model merging
- Hyperparameter considerations

Focus:
Parameter adaptation strategies.

---

### 05_dataset_engineering/

Designing datasets for adaptation.

Includes:
- Dataset design principles
- Instruction data
- Synthetic data
- Data quality, coverage, quantity
- Dataset evaluation

Focus:
Data as the core adaptation lever.

---

### 06_inference_optimization/

Efficiency and cost optimization.

Includes:
- Latency vs throughput trade-offs
- KV caching
- Quantization
- Distillation
- Batching and parallelism
- Serving trade-offs

Focus:
Making foundation models usable at scale.

---

### 07_architecture_and_feedback/

System-level integration.

Includes:
- AI application architectures
- Model gateways
- Observability for LLM systems
- User feedback loops
- Data flywheel
- Build vs buy decisions

Focus:
Holistic AI system design.

---

## Relationship to Other Layers

02_modeling:
Model choice and evaluation for structured ML systems.

04_ml_engineering:
Production lifecycle for classical ML systems.

05_ai_engineering:
System design around pretrained foundation models.

This layer treats the foundation model as a core system component, not just a predictive model.

---
## Explicit Structure

```
05_ai_engineering/
├── 00_foundation_models/
├── 01_evaluation/
├── 02_prompt_engineering/
├── 03_rag_and_agents/
├── 04_finetuning/
├── 05_dataset_engineering/
├── 06_inference_optimization/
├── 07_architecture_and_feedback/
```
