---
layer: 05_ai_engineering
type: index
status: evergreen
created: 2026-03-02
---
# AI Engineering

## Purpose of This Layer

This layer covers the design, integration, evaluation, and operation of large-scale pretrained models (LLMs and other foundation models) as production system components.

It focuses on **system-level AI engineering**, not mathematical derivations (01_foundations) and not general ML infrastructure (04_ml_engineering).

Guiding question:

> How do we design, integrate, evaluate, and operate foundation-model-based systems in real-world settings?

---
# Subdomains

## 1. Foundation Models

**Path:** `foundation_models/`

Covers:

- Model families (GPT, Llama, Mistral, etc.)
- Architecture overviews (Transformer-based LLMs, diffusion models, multimodal models)
- Model selection trade-offs
- Context length considerations
- Cost-performance analysis
- Open vs proprietary model comparisons

Focus:
Understanding models as reusable components in larger systems.

---
## 2. Architectures

**Path:** `architectures/`

System-level compositions of foundation models.
### Includes:

- `rag/` – Retrieval-Augmented Generation systems  
- `agents/` – Agentic workflows and tool-augmented reasoning  
- `tool_use/` – Function calling, external API integration  

Focus:
How models are embedded into larger pipelines and workflows.

---
## 3. Prompting

**Path:** `prompting/`

Covers:

- Prompt design patterns
- Structured outputs
- Chain-of-thought prompting
- Tool invocation prompts
- Guardrails and prompt injection mitigation

Focus:
Controlling model behavior without retraining.

---
## 4. Fine-Tuning

**Path:** `fine_tuning/`

Covers:

- Full fine-tuning
- LoRA / PEFT
- Domain adaptation
- Dataset construction
- Overfitting and catastrophic forgetting risks

Focus:
Adapting foundation models to domain-specific tasks.

---
## 5. Inference

**Path:** `inference/`

Covers:

- Model serving strategies
- Batching and streaming
- Latency optimization
- Quantization
- Hardware considerations
- Caching strategies

Focus:
Running models efficiently and reliably in production.

---
## 6. Evaluation

**Path:** `evaluation/`

Covers:

- Benchmark design
- Automatic evaluation metrics
- Human evaluation
- Hallucination detection
- Domain-specific evaluation (e.g., insurance QA systems)

Focus:
Measuring performance, safety, and business impact.

---
## 7. LLMOps

**Path:** `llmops/`

Covers:

- Monitoring and observability
- Versioning prompts and models
- Cost tracking
- Drift detection
- Safety and compliance considerations

Focus:
Operating LLM systems over time.

---
# Boundaries With Other Layers

- Mathematical derivations of attention, scaling laws → `01_foundations`
- Deep learning architectures as modeling tools → `02_modeling`
- General ML pipelines → `04_ml_engineering`
- Business problem framing → `06_applications`

This layer is strictly about **LLM system engineering**.

---

# Cross-Link Expectations

Notes in this layer should:

- Link to modeling notes (e.g., transformer architecture)
- Link to ML engineering notes (e.g., deployment, monitoring)
- Link to application notes when used in specific domains

This layer integrates, but does not duplicate, core knowledge.