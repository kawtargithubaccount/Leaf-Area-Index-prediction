# "NeuroKnow AI" - ML Engineer Problem Set

Duration: 5 working days
Submission: Private GitHub repository with code [@moitreebasunkai] + comprehensive documentation
Contact: moitree.basu@inria.fr

---

## Deliverables Checklist
- [ ] Code repository with clear structure
- [ ] README.md with setup instructions
- [ ] Technical documentation (architecture, design decisions)
- [ ] Results report with metrics and visualizations
- [ ] System design document (see Problem 4)
- [ ] Provide collaboration access to the repository to user [@moitreebasunkai]

---

## Motivations

1. **Why do you want to work with NeuroKnow AI within Inria?**

2. **What aspects of privacy-preserving AI excite you most, and how does this align with NeuroKnow AI's mission?**


---

## Problem 1: Multimodal Document Understanding

### Context
"NeuroKnow AI" must process documents containing text, tables, and images (scanned forms, invoices, reports).

### Task: OCR + Layout Analysis + Information Extraction

Choose **ONE** document type to process:
- Medical lab reports
- Legal contracts
- Financial statements/invoices

**Dataset suggestions:**
- [RVL-CDIP](https://huggingface.co/datasets/chainyo/rvl-cdip) for document classification
- [FUNSD](https://huggingface.co/datasets/nielsr/funsd) for form understanding
- Create synthetic documents using real templates

**Requirements:**
1. **Document Layout Analysis:**
   - Segment document into: text blocks, tables, images, headers
   - Use LayoutLM, Donut, or similar multimodal models

2. **Information Extraction:**
   - Extract key-value pairs from structured sections
   - Parse tables into structured format
   - Link extracted information to document sections

3. **Evaluation:**
   - Report extraction accuracy for 50-100 documents
   - Show examples of successful and failed extractions
   - Measure processing time per document

**Bonus:** Handle documents in multiple languages (English + French) or implement rotation/skew correction for scanned documents.

### Documentation Required:
- Model architecture diagram
- Preprocessing pipeline explanation
- Performance bottlenecks and optimization strategies

---

## Problem 2: Multi-Sector Document NER & Classification

### Context
"NeuroKnow AI" processes sensitive documents across healthcare, legal, and financial sectors. Each domain requires accurate entity extraction and classification.

### Task A: Fine-tune a Domain-Specific NER

Choose **ONE** of these multilingual (English + French) dataset options:
- **Healthcare:** Combine **[NCBI Disease](https://huggingface.co/datasets/ncbi/ncbi_disease)** (English) + **[Quaero French Med](https://huggingface.co/datasets/mnaguib/QuaeroFrenchMed)** (French)
- **Legal:** **[Text Anonymization Benchmark (MAPA)](https://huggingface.co/datasets/mattmdjaga/text-anonymization-benchmark-train)** (European Court Documents in EN/FR/etc.)
- **Financial:** **[Gretel Synthetic PII Finance](https://huggingface.co/datasets/gretelai/synthetic_pii_finance_multilingual)** (Multilingual Financial Documents)

**Requirements:**
1. Fine-tune a **multilingual** Transformer (e.g., XLM-RoBERTa, mBERT, or CamemBERT) using **PyTorch or TensorFlow**
2. Train on a combined corpus (e.g., 500 English + 500 French examples)
3. Implement and report:
   - Token-level F1, Precision, Recall for each entity type (separately for EN and FR)
   - Analysis of cross-lingual transfer (e.g., train on EN, zero-shot test on FR)
4. Discuss challenges in aligning entity taxonomies across different languages/datasets.

**Bonus:** Implement zero-shot NER using an instruction-tuned LLM (Mistral, Ollama) and compare with your fine-tuned model.

### Task B: Cross-Domain Entity Detection & Classification

Create a system that can detect when a document contains entities from multiple sectors (e.g., a medical-legal document) and classify it accordingly.

**Requirements:**
- **Data:** Use small subsets (e.g., 50 docs each) from the other two datasets listed in Task A to build/test your classifier.
- Classify documents into: Healthcare, Legal, Finance, or Multi-domain
- Report classification accuracy and confusion matrix
- Return the classification results and extracted entities in a JSON format.
- Discuss how this impacts entity extraction strategy

### Documentation Required:
- Model selection rationale
- Hyperparameter choices
- Performance analysis with visualizations
- Error analysis with concrete examples

---

## Problem 3: Privacy-Preserving Document Processing

### Context
"NeuroKnow AI" must anonymize sensitive information while preserving document utility. **Choose ONE of the following tracks based on your background.**

### Track A: Applied Anonymization Strategy (No DP knowledge required)
**Goal:** Build a robust multilingual PII detection and redaction system, and evaluate its impact on downstream tasks.

**Test on this sample:**
```
Dossier Médical: Patient Marie Dubois (née le 15/03/1978)
Sécurité Sociale: 2 78 03 75 116 025 43
Adresse: 42 Rue de la République, 59000 Lille
Tél: +33 6 12 34 56 78
Email: marie.dubois@example.fr
Diagnostic: Hypertension (ICD-10: I10)
Médecin: Dr. Jean Martin, ID médical: FR789456123
```

**Requirements:**
1. **Detection:** Identify Names, Dates, SSNs, Addresses, Phones, Emails, and Medical IDs in both English and French.
2. **Redaction Strategies:** Implement three modes:
   - *Strict:* Full masking `[REDACTED]`
   - *Typed:* Entity type masking `[NAME]`, `[SSN]`
   - *Surrogate:* Replace with synthetic but realistic data (e.g., `Marie Dubois` → `Alice Smith`) to preserve readability.
3. **Consistency:** Ensure the same entity is redacted consistently across the document (e.g., "Marie Dubois" is always replaced by the same surrogate).
4. **Evaluation:** Report Precision/Recall for each PII type on a held-out test set.
5. **Downstream Utility (Crucial):** Run your **Problem 2 (NER or Classification)** pipeline on the redacted documents. Compare the performance (F1 Score/Accuracy) before vs. after redaction for each strategy (Strict vs. Surrogate).

**Documentation for Track A:**
- Regex vs. Model-based approach trade-offs.
- Strategy for handling false positives in medical/legal contexts.
- How you handle multilingual PII formats (e.g., US vs. French phone numbers).
- Analysis of performance degradation on downstream tasks.

### Track B: Differential Privacy for Embeddings (Advanced - Optional, try only if you are comfortable with DP)
**Goal:** Implement a differentially private text embedding system and evaluate its utility.

**Requirements:**
1. **Dataset:** Use a corpus of 200-500 sensitive documents (from Problem 1 or 2).
2. **Mechanism:** Generate embeddings (e.g., SBERT) and apply **Differential Privacy** using:
   - Gaussian/Laplace mechanism on the embedding vectors, OR
   - DP-SGD during fine-tuning.
3. **Budget:** Implement privacy budget tracking (ε, δ parameters).
4. **Trade-off Analysis:**
   - Measure **Utility:** Train a simple classifier (from Problem 2 Task B) using the private embeddings. Report accuracy drop compared to non-private embeddings.
   - Measure **Privacy:** Distance to original embedding.
   - Plot the Utility vs. Privacy frontier for ε = [0.1, 1.0, 5.0, 10.0].

**Documentation for Track B:**
- Mathematical explanation of the chosen DP mechanism.
- Proof/Explanation of how ε guarantees are satisfied.
- Analysis of embedding degradation impacts on downstream tasks.

---

## Problem 4: System Design Document

### Task: Design Decisions & Production Architecture

Write a technical design document for deploying your solutions in production.

**Address these aspects:**

### 4.1 Architecture Design
- How would you structure the ML pipeline? (preprocessing → inference → post-processing)
- Where does each model (NER, privacy/anonymization, OCR) fit in the architecture?
- How do you handle multi-tenancy for different sectors?

### 4.2 GPU Optimization Strategy
- Batch processing strategy for high-throughput
- Mixed precision training/inference
- Model quantization or distillation for deployment
- Multi-GPU scaling approach

### 4.3 MLOps & Deployment
- Model versioning and A/B testing strategy
- Monitoring: What metrics track model performance in production?
- CI/CD pipeline for model updates
- Docker containerization strategy

### 4.4 Privacy & Security
- **Track A (Anonymization):**
  - How do you ensure consistent redaction across millions of documents?
  - How do you handle potential re-identification risks?
  - Data handling: Storage, processing, and deletion policies
  - Audit logging for GDPR compliance
  - Secure model serving (authentication, encryption)
- **Track B (DP):**
  - How do you manage privacy budget (ε) consumption in a continuous release cycle?
  - Data handling: Storage, processing, and deletion policies
  - Audit logging for GDPR compliance
  - Secure model serving (authentication, encryption)

### 4.5 Scalability & Performance
- Expected throughput: X documents/second
- Latency requirements: P95, P99 response times
- Cost optimization strategies (GPU utilization, caching)
- Handling peak loads and failover strategies

**Include:**
- Architecture diagrams (use draw.io, Lucidchart, or ASCII)
- Technology stack choices with justification
- Trade-off analysis (accuracy vs. speed vs. cost)

---


## Tips for submission

✅ **Do:**
- Start with Problem 1 (foundational)
- Use pre-trained models (don't train from scratch)
- Focus on depth over breadth (better to nail the core problems than rush all tasks)
- Show your thought process in documentation
- Be honest about limitations and trade-offs
- Don't use ChatGPT/Claude or Other LLMs for your work.

❌ **Don't:**
- **NEVER** use proprietary datasets without permission [**WILL RESULT IN IMMEDIATE REJECTION**]
- **NEVER** copy code from tutorials without understanding [**WILL RESULT IN IMMEDIATE REJECTION**]
- **NEVER** skip documentation to save time
- **NEVER** ignore privacy considerations
- **NEVER** over-engineer solutions

- **Regarding LLMs:** You may use LLMs for brainstorming or debugging, but **do not** generate your core solution code with them. You will be asked to explain your implementation in detail during the interview.

---

## Questions?

Email: moitree.basu@inria.fr  
Subject: "ML Engineer Problem Set - [Your Name]"

**Good luck! We're excited to see your approach to privacy-preserving document intelligence. 🚀**
