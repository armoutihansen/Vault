# Vault Conventions  
  
## 1. Purpose of This Vault  
  
This vault is a structured professional knowledge system supporting:  
  
- Mathematical and statistical foundations  
- Machine learning modeling  
- Software engineering depth  
- ML engineering (production systems)  
- AI engineering (LLM systems)  
- Insurance domain expertise  
- Active and completed projects  
  
The vault is organized by **epistemic layer and abstraction level**, not by buzzwords or temporary interests.  
  
Top-level structure:
```
00_meta  
01_foundations  
02_modeling  
03_software_engineering  
04_ml_engineering  
05_ai_engineering  
06_applications  
07_projects  
08_reading  
09_logs  
99_archive
```

The top-level structure is stable. Changes require explicit revision of this document.  
  
---  
# 2. Layer Definitions  
  
## 01_foundations  

### Purpose  
Timeless theoretical foundations.  
### Contains  
- Mathematics (calculus, linear algebra, differential equations, real analysis)  
- Optimization theory  
- Probability theory  
- Statistical inference  
- Time series theory  
- Actuarial mathematics  
- Theoretical deep learning  
### Does NOT Contain  
- Framework-specific code  
- Deployment instructions  
- Business context  
- Tool configuration  
### Guiding Question  
**Why does this work mathematically?**  
  
If a note contains derivations, formal definitions, or proofs → it belongs here.  
  
---  
## 02_modeling    
### Purpose  
Model classes and modeling strategies, framework-agnostic.  
### Contains  
- Supervised learning  
- Unsupervised learning  
- Deep learning architectures  
- Probabilistic models  
- Feature engineering  
- Evaluation metrics  
- Insurance modeling techniques (e.g., GLMs for pricing)  
### Does NOT Contain  
- Docker, CI/CD, orchestration  
- MLflow, Airflow  
- API deployment instructions  
- Pure mathematical derivations  
### Guiding Question  
**How do we model this problem?**  
  
Modeling notes describe algorithms, inductive biases, trade-offs, and modeling logic.  
  
---  
## 03_software_engineering  

### Purpose  
General software engineering knowledge supporting all technical work.    
### Contains  
- Programming languages (Python, TypeScript, Go)  
- Data structures & algorithms  
- System design  
- API design  
- Testing strategies  
- Docker  
- Kubernetes  
- GitHub Actions  
- Framework usage (e.g., PyTorch API patterns, FastAPI usage)  
### Does NOT Contain  
- Statistical theory  
- Business domain reasoning  
- ML evaluation frameworks (unless purely tooling-related)  
### Guiding Question  
**How do we build robust software systems?**  
  
If the note is about implementation patterns or tooling → it belongs here.  
  
---  
## 04_ml_engineering  

### Purpose  
Productionization of machine learning systems (model-agnostic).  
### Contains  
- Data pipelines  
- ETL/ELT architectures  
- Experiment tracking (MLflow, etc.)  
- Feature stores  
- Monitoring  
- Reproducibility  
- CI/CD for ML systems  
- Model deployment (non-LLM specific)  
### Does NOT Contain  
- Deep learning architecture theory  
- LLM-specific system integration  
- Business problem framing  
### Guiding Question  
**How do we productionize ML systems?**  
  
Infrastructure and lifecycle management live here.  
  
---  
## 05_ai_engineering  
  
### Purpose  
Productionization and integration of foundation models and LLM systems.  
### Contains  
- Foundation model integration  
- RAG architectures  
- LLMOps  
- Fine-tuning pipelines  
- Inference optimization  
- Quantization  
- LLM evaluation frameworks  
- Safety and monitoring for LLM systems  
### Does NOT Contain  
- Transformer mathematical derivations  
- General ML infrastructure (unless LLM-specific)  
- Pure business problem framing  
### Guiding Question  
**How do we integrate and operate large-scale pretrained models?**  
  
Foundation models are deep learning models conceptually, but treated here as system components.  
  
---  
## 06_applications  

### Purpose  
Domain-specific and business-oriented knowledge.    
### Contains  
- Insurance industry structure  
- Regulatory frameworks  
- Underwriting processes  
- Risk modeling case studies  
- Fraud detection problem framing  
- Experimentation design in business context  
- Stakeholder and deployment constraints  
### Does NOT Contain  
- Core mathematical derivations  
- Algorithmic definitions  
- Tool-specific instructions  
### Guiding Question  
**Why are we solving this problem and under which constraints?**  
  
Applications integrate knowledge from all other layers.  
  
---  
# 3. Projects (07_projects)  

## Purpose  
  
Projects are time-bound execution instances.  
  
They integrate:  
- Foundations  
- Modeling  
- Engineering  
- Applications  
  
Projects are not abstraction layers.  
  
---    
## Structure

```
07_projects/  
_active/  
_completed/  
_experimental/
```

Each project has its own folder:
```
07_projects/_active/project_name/
```

Recommended internal structure:  
  ```
- overview.md  
- problem_definition.md  
- data_pipeline.md  
- modeling.md  
- evaluation.md  
- deployment.md  
- lessons_learned.md  
  ```
---  
## Project Rules  
  
1. Projects may reference any layer.  
2. Projects must not duplicate core knowledge.  
3. Reusable insights must be extracted to the appropriate layer.  
4. Temporary experimentation is allowed inside projects.  
5. Projects are integration hubs, not theory repositories.  
  
---  
## Extraction Rule  
  
When content becomes:  
- Generalizable  
- Reusable across projects  
- Conceptually stable  
  
Move it to:  
- Foundations  
- Modeling  
- Software engineering  
- ML engineering  
- AI engineering  
- Applications  
  
Replace the project content with a link.  
  
---  
# 4. Reading (08_reading)  

## Purpose  
  
Structured intake of external material.  
  
Contains notes derived from:  
- Papers  
- Books  
- Articles  
- Documentation  
- Industry reports  
  
---    
## Structure

```
08_reading/  
papers/  
books/  
articles/  
reports/
```
---  
  
## Reading Rules  
  
1. Reading notes summarize and interpret external content.  
2. They must link to relevant core layers.  
3. If a concept becomes central → extract to core layer.  
4. Avoid storing raw highlights without synthesis.  
  
Reading is intake.  
Core layers are synthesis.  
  
---  
# 5. Logs (09_logs)  

## Purpose  
  
Operational and temporary memory.  
  
Contains:  
- Daily notes  
- Weekly reviews  
- Meeting summaries  
- Brain dumps  
- Reflections  
  
---  
## Structure

```
09_logs/  
daily/  
weekly/
```
  
---  
## Log Rules  
  
1. Logs are not permanent knowledge.  
2. Valuable insights must be extracted to core layers.  
3. Logs may contain unstructured content.  
4. Logs are reviewed weekly for extraction.  
  
---  
# 6. Abstraction Placement Rule  
  
When unsure where a note belongs, ask:  
  
- Why does this work mathematically? → Foundations  
- How do we model this? → Modeling  
- How do we implement this? → Software engineering  
- How do we productionize this? → ML engineering  
- How do we integrate LLM systems? → AI engineering  
- Why are we solving this problem? → Applications  
- Is this a concrete execution instance? → Projects  
- Is this intake from external material? → Reading  
- Is this temporary thought or reflection? → Logs  
  
---  
# 7. Naming Conventions  

## File Names  
  
- Lowercase only  
- Underscores instead of spaces  
- Singular nouns preferred  
  
Examples:    
- logistic_regression.md  
- gradient_boosting.md  
- docker_networking.md  
- insurance_pricing.md  
  
Avoid:  
- CamelCase  
- Spaces  
- Generic names (e.g., "notes", "stuff")  
  
---  
## Folder Names  
  
- Lowercase  
- Descriptive  
- No vague categories (e.g., misc, advanced_topics)  
  
---  
# 8. Note Structure Standards  
  
## Conceptual / Modeling Notes

```
## Definition

## Intuition

## Formal Description

## Applications

## Trade-offs

## Links
```
---  
## Engineering Notes

```
## Purpose

## Architecture

## Implementation Notes

## Trade-offs

## References
```

---    
## Application Notes

```
## Problem Definition

## Domain Context

## Data Requirements

## Modeling Options

## Deployment Constraints

## Risks

## Links
```

---
# 9. Cross-Link Governance

To prevent silos:

1. Every application note must link to:
   - At least one modeling note
   - At least one engineering note

2. Every modeling note should link to:
   - At least one foundational note

3. Engineering notes support outward but should not contain theory.

4. Avoid duplicate conceptual notes.
   If overlap occurs → merge.

---
# 10. Knowledge Lifecycle

Information flows as follows:

Reading → Logs → Projects → Core Layers

Core layers are stable.
Everything else is transitional.

---
# 11. Stability Principle

The top-level folder structure is stable.

Any structural change requires:
- Clear justification
- Update to this document

This vault is a professional knowledge architecture, not a casual note collection.