#  System Design Document

## 1 Architecture Design
The system is designed as a **modular, service-oriented ML pipeline** capable of handling large-scale document processing across multiple sectors (legal, finance, healthcare). Each stage is independently scalable and observable.


┌─────────────────────────────────────────────────────────────────────────┐
│                     CLIENT APPLICATIONS                                  │
│  (Web UI, Mobile Apps, API Integrations, Batch Uploads)                  │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │ HTTPS/API Requests
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     API GATEWAY & LOAD BALANCER                          │
│  • Request Routing & Authentication                                      │
│  • Rate Limiting (1000 RPM/user)                                         │
│  • DDoS Protection                                                        │
└─────────────────────┬──────────────────────┬────────────────────────────┘
                      │                      │
           ┌──────────▼──────────┐  ┌───────▼────────┐
           │   REAL-TIME API     │  │  BATCH QUEUE   │
           │  (FastAPI/Gunicorn) │  │  (Kafka/Rabbit) │
           └──────────┬──────────┘  └───────┬────────┘
                      │                      │
           ┌──────────▼──────────────────────▼──────────┐
           │          DOCUMENT PROCESSING ENGINE         │
           │  (Orchestrator + Workflow Manager)          │
           └─┬─────────┬─────────┬─────────┬──────────┬─┘
             │         │         │         │          │
    ┌────────▼─┐ ┌────▼────┐ ┌──▼───┐ ┌──▼──┐ ┌─────▼────┐
    │  OCR     │ │ Domain  │ │ NER  │ │Privacy│ │Post-     │
    │  Service │ │Classifier│ │Engine │ │Module │ │processing│
    │(Tesseract│ │ (XLM-R)  │ │(Layout│ │      │ │ & Output │
    │ + Custom)│ │         │ │ LMv3) │ │      │ │ Formatting│
    └──────────┘ └─────────┘ └──────┘ └──────┘ └─────────┘
                      │                      │
           ┌──────────▼──────────────────────▼──────────┐
           │          MODEL SERVING LAYER                │
           │  • TorchServe/Triton Inference Server       │
           │  • GPU Pool Management                      │
           │  • Model Versioning & A/B Testing           │
           └─────────────────────────────────────────────┘
                      │                      │
           ┌──────────▼──────────────────────▼──────────┐
           │          DATA & STORAGE LAYER               │
           │  • Raw Documents (S3)                       │
           │  • Processed Results (PostgreSQL)           │
           │  • Vector DB (Pinecone/Weaviate)            │
           │  • Audit Logs (Immutable Storage)           │
           └─────────────────────────────────────────────┘
```


### ML Pipeline Structure

#### 1. Preprocessing

* OCR (e.g. LayoutLM-compatible OCR)
* Language detection
* Layout extraction (bounding boxes, reading order)
* Token alignment

#### 2. Inference

* **NER model**: detects entities (PII, medical, financial)
* **Domain classifier**: determines sector-specific routing
* **Privacy/anonymization model**: redacts or masks sensitive spans

#### 3. Post-processing

* Entity merging across pages
* Sector-specific redaction rules
* Consistent pseudonym generation
* Output formatting (JSON, PDF, masked text)

---

### Model Placement

| Model             | Responsibility           | Location        |
| ----------------- | ------------------------ | --------------- |
| OCR               | Text + layout extraction | Preprocessing   |
| Domain Classifier | Sector routing           | Inference       |
| NER               | Entity detection         | Inference       |
| Anonymization     | Redaction / masking      | Post-processing |


### Multi-Tenancy Strategy

* **Logical isolation per tenant**

  * Separate config, encryption keys, logs
* **Shared core models**

  * Sector-specific heads or adapters
* **Routing based on `tenant_id`**



## 2 GPU Optimization Strategy

### Batch Processing

**Dynamic Batching Algorithm:**
1. **Group by Characteristics:**
   - Similar document types
   - Similar text lengths (buckets: <128, 128-512, >512 tokens)
   - Same tenant/model requirements

2. **Batch Formation Rules:**
   - Max batch size: 32 (balanced memory/throughput)
   - Timeout: 50ms (collect similar requests)
   - Priority override: Premium tenants skip queue

3. **Memory Management:**
   - Gradient checkpointing for large models
   - CPU offloading for less critical layers
   - Shared memory for common embeddings

### Mixed Precision

* FP16 / BF16 inference using AMP
* Reduces memory by ~50% with negligible accuracy loss

### Quantization & Distillation

* INT8 quantized models for inference-only workloads
* Distilled models for real-time endpoints
* Full models retained for batch/high-accuracy jobs

### Multi-GPU Scaling

**Data Parallelism:** (Primary strategy)
- Each GPU gets a batch slice
- Synchronous gradient averaging
- 4-8 GPUs per model instance

**Model Parallelism:** (For very large models)
- Layer splitting across GPUs
- Pipeline parallelism with micro-batches
- Used only for >10B parameter models

**Inference Scaling:**
- Model replicas across GPU nodes
- Smart routing based on load/affinity
- Warm-up periods to load models before traffic


## 3 MLOps & Deployment

### Model Versioning

* Semantic versioning (model + data + config)
* Registry-based deployment (e.g. MLflow)
* Canary and A/B testing per tenant or sector

### Monitoring

**Four Pillars of ML Monitoring:**

1. **Infrastructure Metrics:**
   - GPU Utilization (>85% target, <95% alert)
   - Memory Usage (alert at 90%)
   - Batch processing time (SLA compliance)
   - Queue lengths (alert at >1000)

2. **Model Performance Metrics:**
   - Prediction latency (P95 < SLA)
   - Throughput (docs/second)
   - Error rates (alert at >1%)
   - Confidence score distribution

3. **Data Quality Metrics:**
   - Input data drift (PSI/KL divergence)
   - Concept drift (accuracy degradation)
   - Out-of-distribution detection
   - Label quality monitoring

4. **Business Metrics:**
   - User satisfaction scores
   - API usage patterns
   - Cost per document processed
   - SLA compliance rates

### CI/CD Pipeline

1. **Development:**
   - Code review with ML-specific checks
   - Unit tests for data processing
   - Integration tests with sample data
   - Security vulnerability scanning

2. **Training:**
   - Automated training on new data
   - Hyperparameter optimization (Optuna)
   - Cross-validation with statistical tests
   - Bias/fairness evaluation

3. **Validation:**
   - Performance on holdout test set
   - A/B test against current production
   - Drift detection on recent production data
   - Compliance checks (privacy, regulations)

4. **Deployment:**
   - Canary deployment (1% → 10% → 50% → 100%)
   - Blue-green deployment for zero downtime
   - Automatic rollback on failure detection
   - Post-deployment monitoring

5. **Retirement:**
   - Performance degradation detection
   - Automatic retraining triggers
   - Graceful deprecation with migration

### Containerization

* One container per service (OCR, NER, Privacy)
* GPU-enabled containers
* Immutable images with pinned dependencies


## 4 Privacy & Security

### Track A – Anonymization

**Multi-Layer Anonymization Pipeline:**

1. **Detection Layer:** (Ensemble Approach)
   - **Model 1:** BERT-based PII detector (high recall)
   - **Model 2:** Rule-based pattern matcher (SSN, Credit Cards)
   - **Model 3:** Dictionary lookup (company names, locations)
   - **Model 4:** Context-aware classifier (ambiguous cases)
   - **Consensus Mechanism:** Require 2/4 models for redaction

2. **Redaction Layer:**
   - **Text Redaction:** [REDACTED] placeholder with type tag
   - **Image Redaction:** Black bars on scanned documents
   - **Metadata Removal:** EXIF, document properties
   - **Consistent Hashing:** Same entity → same pseudonym across documents

3. **Validation Layer:**
   - **Re-identification Risk Assessment:** k-anonymity ≥ 3
   - **Context Preservation:** Maintain document readability
   - **Audit Trail Generation:** What was removed, when, why

**Data Flow for Anonymization:**
```
Raw Document → OCR → PII Detection → Risk Assessment → 
                  ↓                     ↓
              Redaction           Audit Logging
                  ↓                     ↓
           Anonymized Doc        Immutable Storage
                  ↓
           Quality Check
                  ↓
           Encrypted Storage
```


## 5 Scalability & Performance

### Target Performance

* **Throughput**: 50–200 documents/sec (batch mode)
* **Latency**:

  * P95 < 500 ms (text)
  * P99 < 2s (OCR-heavy)

### Cost Optimization

* Spot GPUs for batch jobs
* Autoscaling inference pools
* Result caching for repeated documents

### Peak Load Handling

* Queue-based backpressure
* Priority lanes per tenant
* Graceful degradation (reduced model size)

### Failover Strategy

* Multi-region deployment
* Stateless services
* Warm standby GPU pools



## Technology Stack (Example)

| Layer         | Technology           | Justification      |
| ------------- | -------------------- | ------------------ |
| Serving       | FastAPI / gRPC       | Low latency        |
| ML            | PyTorch + HF         | Flexibility        |
| OCR           | Layout-aware OCR     | Document structure |
| Orchestration | Kubernetes           | Scalability        |
| Monitoring    | Prometheus + Grafana | Observability      |
| Registry      | MLflow               | Version control    |

---

## Trade-Off Analysis
**Accuracy vs. Speed Trade-off:**
```
Option A: Maximum Accuracy
- Models: LayoutLMv3-Large + BERT-Large ensemble
- Accuracy: 94% F1
- Latency: 3500ms
- Cost: $0.05/document
- Use Case: Legal contracts, medical records

Option B: Balanced Approach ✓ SELECTED
- Models: LayoutLMv3-Base + Distilled BERT
- Accuracy: 90% F1
- Latency: 1200ms
- Cost: $0.02/document
- Use Case: General business documents

Option C: Maximum Speed
- Models: Rule-based + FastText
- Accuracy: 75% F1
- Latency: 200ms
- Cost: $0.005/document
- Use Case: High-volume, low-risk documents
```

**Privacy vs. Utility Trade-off:**
```
Track A: Strict Anonymization
- Privacy: k=5 anonymity, l=3 diversity
- Utility: Some context loss, readable output
- Compliance: GDPR Article 89, HIPAA Safe Harbor
- Use Case: Healthcare, sensitive personal data

Track B: Differential Privacy
- Privacy: ε=1.0 privacy budget
- Utility: Statistical accuracy preserved
- Compliance: GDPR research exemption
- Use Case: Analytics, research datasets

Hybrid Approach ✓ SELECTED
- Detection: Strict anonymization for PII
- Aggregation: Differential privacy for statistics
- Balance: Privacy protection + business utility
```

**Cost vs. Performance Trade-off:**
```
Tier 1: Premium Performance
- GPU: A100 (80GB)
- Latency: <500ms
- Cost: $3.50/hour
- Use: Real-time, customer-facing applications

Tier 2: Balanced ✓ SELECTED
- GPU: A10 (24GB)
- Latency: <1500ms
- Cost: $1.50/hour
- Use: General processing, mixed workload

Tier 3: Cost Optimized
- GPU: T4 (16GB)
- Latency: <3000ms
- Cost: $0.35/hour
- Use: Batch processing, non-critical workloads


| Aspect           | Accuracy | Speed | Cost   |
| ---------------- | -------- | ----- | ----   |
| Full models      | hight    | meduim| low    |
| Distilled models | meduim   | hight | meduim |
| Quantized models | low      | hight | hight  |




