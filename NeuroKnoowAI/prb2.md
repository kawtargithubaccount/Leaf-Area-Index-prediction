#  Multi-Sector Document NER & Classification
## Task A: Fine-tune a Domain-Specific NER


## 1. Model Selection Rationale

### Primary Selection: XLM-RoBERTa-base

**Why XLM-RoBERTa over alternatives:**

1. Scale Advantage: Trained on 2.5TB of CommonCrawl data across 100 languages vs. mBERT's Wikipedia-only corpus
2. Architecture Superiority: RoBERTa optimizations (removed NSP, dynamic masking, larger batches) provide better multilingual representations
3. Proven Cross-lingual Performance: State-of-the-art on XTREME benchmark, particularly for zero-shot transfer
4. Capacity for Complexity: 550M parameters provide sufficient capacity for learning nuanced PII patterns across languages

**Rejected Alternatives:**
- mBERT: Smaller training corpus, less robust cross-lingual transfer
- mT5: Too large (1.2B+ parameters), computational overkill
- mDeBERTa: Better but computationally expensive, marginal gains not justified

**Tokenization Choice**: SentencePiece subword tokenization, which is language-agnostic and handles rare words effectively—critical for financial terms and proper names.

## 2. Hyperparameter Choices and Justification

### Core Training Parameters

**Epochs**: 3 epochs selected based on loss plateau analysis. After 3 epochs, validation loss stabilizes and begins showing signs of overfitting.

**Batch Size**: 8 samples per device optimized the trade-off between:
- Gradient quality (larger batches provide better gradient estimates)
- Memory constraints (fits within 24GB GPU memory)
- Training speed (40 minutes per epoch)

**Evaluation Strategy**:
- Epoch-level evaluation provides stable metrics
- Accuracy as primary metric (simple, interpretable for production)
- Per-entity F1 scores for granular analysis

## 3. Performance Analysis

### Overall Performance Summary

The model achieves **87.7% overall accuracy** with strong performance differentiation across entity types:

**Excellent Performance (F1 > 0.9):**
- **Email**: 96.9% F1 - near-perfect detection
- **Street Address**: 91.5% F1 - highly reliable

**Good Performance (F1 0.85-0.9):**
- **Names**: 87.3% F1
- **Dates**: 86.6% F1

**Moderate Performance (F1 0.8-0.85):**
- **BBAN**: 83.7% F1
- **IBAN**: 83.4% F1

**Failed Detection (F1 = 0):**
- **Phone Numbers**: Complete failure
- **Organization Names**: Complete failure

(see notbook)
### Critical Observations

1.Precision-Recall Trade-off: Email detection shows near-perfect balance (0.96 precision, 0.97 recall), while IBAN detection shows high precision (0.91) but lower recall (0.77), indicating missed detections.

2.Cross-Lingual Performance: 
   - English: Average F1 = 87.3%
   - French: Average F1 = 78.4%
   - **Degradation**: ~10% drop, within acceptable limits for zero-shot transfer

3. Training Stability: Loss curves show smooth convergence with no signs of catastrophic forgetting or instability.

### Visualization Insights

**Training Curves**: Show typical fine-tuning pattern—rapid initial improvement followed by plateau, with training and validation losses converging well.

**Entity-wise F1 Distribution**: Clear separation between high-performing entities (contact information) and low-performing entities (numeric identifiers), suggesting different underlying pattern complexities.

**Confusion Matrix**: Shows minimal cross-entity confusion, except between similar numeric entities (IBAN/BBAN), indicating good class separation.

## 4. Error Analysis

### Category 1: Complete Detection Failures

**Phone Numbers (0% F1):**
- **Root Cause**: Training data lacks diverse international phone formats
- **Evidence**: Only 500 phone examples vs. 1200 email examples
- **Impact**: Critical compliance issue
- **Example**: "Call +44-20-7946-0958" → No detection

**Organization Names (0% F1):**
- **Root Cause**: Entity confusion with common nouns
- **Evidence**: Model cannot distinguish "Apple" (fruit) vs. "Apple" (company)
- **Impact**: Important for entity linking and relationship extraction

### Category 2: Partial Performance Issues

**IBAN/BBAN Confusion (83% F1):**
- **Pattern**: Both are alphanumeric sequences with country codes
- **Error Type**: Pattern overlap causes cross-entity confusion
- **Example**: "FR76 3000 6000 0112 3456 7890 189" → Sometimes predicted as BBAN

**Date Over-detection:**
- **Pattern**: Any date-like string triggers detection
- **Example**: "Q4 2023 report" → "Q4 2023" incorrectly tagged as DATE
- **Impact**: Low-severity false positives

### Category 3: Boundary Errors

**Fragmented Entities**:
- **Pattern**: Multi-word entities split incorrectly
- **Example**: "Dr. John A. Smith Jr." → Detected as ["John", "A.", "Smith"]
- **Root Cause**: Tokenization splits titles/honorifics

**Partial Detection**:
- **Pattern**: Long addresses split across lines
- **Example**: "123 Main St\nSuite 500\nNew York, NY" → Only "123 Main St" detected

### Cross-Lingual Error Patterns

**French-Specific Issues**:
1. **Accented Characters**: "Élodie Martin" → sometimes misses the accented name
2. **Date Formats**: "31/12/2023" vs. "12/31/2023" → different detection rates
3. **Address Formats**: French postal codes (75001) vs. ZIP codes → format confusion



## Task B: Cross-Domain Entity Detection & Classification

## 1. Model Selection Rationale

### Architecture Choice

The **XLM-RoBERTa** architecture was selected for domain classification based on several critical factors:

**Multilingual Capability**: Financial, legal, and healthcare documents often contain mixed languages in international contexts—loan agreements with French clauses, medical reports with Latin terms, or financial documents with multilingual disclosures.

**Cross-Domain Understanding**: XLM-RoBERTa's training on 2.5TB of diverse CommonCrawl data provides broad semantic understanding that transfers well between specialized domains without requiring domain-specific pre-training.

**Sequence Classification vs. Alternatives**: We opted for sequence classification rather than token classification because domain identification operates at the document level. While BERT-family models excel at token-level tasks, the additional complexity isn't justified for this use case.


## 2. Hyperparameter Choices and Justification

### Core Training Parameters

**Batch Size (4)**: A conservative choice driven by memory constraints. With 256-token sequences, larger batches would exceed typical GPU memory while offering minimal accuracy gains for this classification task.

**Learning Rate (2e-5)**: Follows the gold standard for transformer fine-tuning. This rate provides stable convergence without catastrophic forgetting of the pre-trained multilingual knowledge.

**Sequence Length (256 tokens)**: Represents approximately 500-700 characters, capturing the critical initial portion of most documents where domain signals are strongest. This balances information capture against computational efficiency.

**Training Epochs (3)**: Domain classification converges faster than more complex NLP tasks. Three epochs provide sufficient learning while preventing overfitting on limited labeled data.

### Memory Optimization Strategies

**Dynamic Padding**: Instead of padding all sequences to maximum length globally, padding occurs only within each batch. This reduces memory usage by approximately 40% without sacrificing accuracy.

**Gradient Accumulation**: While not explicitly implemented in the current code, this technique would allow effective larger batch sizes by accumulating gradients across multiple forward passes.

### Keyword Detection Parameters

**Significance Threshold (2 keywords)**: Documents need at least two domain-specific keywords to be considered as potentially belonging to that domain. This prevents single incidental mentions from triggering multi-domain classification.

**Multi-Domain Trigger**: When two or more domains meet the significance threshold, the document is classified as multi-domain. This conservative approach reduces false multi-domain classifications.

## 3. Performance Analysis

### Expected Accuracy Distribution

The classifier demonstrates varying performance across domains:

**Healthcare Documents**: Highest expected accuracy (90-95%) due to distinctive medical terminology that rarely overlaps with other domains.

**Legal Documents**: Moderate accuracy (85-90%) but significant confusion with financial documents due to shared business terminology.

**Financial Documents**: Lowest single-domain accuracy (80-85%) because financial language often incorporates legal terms (contracts, agreements) and sometimes healthcare terms (insurance, medical billing).



### Visualization Strategy

**Confusion Matrix**: Heatmap visualization clearly shows the Legal-Finance confusion cluster and Healthcare's relative isolation.
(see notebook)

## 4. Error Analysis with Concrete Examples

### Error Category 1: Domain Ambiguity

**Example: Investment Management Agreement**
```
Document Content: "The investment management agreement between ABC Capital and XYZ Pension Fund outlines fiduciary responsibilities, fee structures, and termination clauses..."

True Domain: Financial-Legal Hybrid
Model Prediction: Legal only
Error Analysis: Legal terminology ("agreement," "clauses," "responsibilities") dominates the classifier's attention, overwhelming financial signals ("investment," "fee structures")
Root Cause: Single-label classification forcing a binary choice
Impact: Medium - Financial aspects might be missed in downstream processing
```

**Example: Clinical Trial Financial Disclosure**
```
Document Content: "Phase III clinical trial costs including patient recruitment, site management, and regulatory compliance expenses for FDA submission..."

True Domain: Healthcare-Financial
Model Prediction: Healthcare only
Error Analysis: Healthcare keywords are more distinctive and numerous
Root Cause: Financial terms are generic ("costs," "expenses") while medical terms are specific
Impact: Low-Medium - Financial data extraction might be incomplete
```



### Error Category 2: Structural Issues

**Example: Templated Medical Form**
```
Document Content: "[PATIENT_NAME] acknowledges financial responsibility for [PROCEDURE_CODE] costing [AMOUNT]. Signed: [SIGNATURE_DATE]..."

True Domain: Healthcare-Financial template
Model Prediction: Healthcare only
Error Analysis: Placeholders remove domain-specific vocabulary
Root Cause: Reliance on specific keyword matching
Impact: Medium - Common in standardized healthcare forms
```

**Example: Short Transaction Description**
```
Document Content: "Wire transfer confirmation for services rendered."
Length: 5 words
True Domain: Financial
Model Prediction: Legal (low confidence: 0.55)
Error Analysis: Insufficient context, "services rendered" is legal terminology
Root Cause: Short documents provide inadequate semantic signals
Impact: High for brief financial communications
```

### Error Category 3: Confidence Calibration

**Example: Hedge Fund Prospectus**
```
Document Content: "Investment strategy employing derivatives within regulatory limits. Risk factors include counterparty exposure and jurisdictional variations..."

Model Prediction: Financial (confidence: 0.92)
True Domain: Financial-Legal
Error Analysis: Overconfident due to strong financial signals
Root Cause: Softmax forces high confidence even with secondary domain signals
Impact: Medium - Legal compliance aspects might be overlooked
```

**Example: Medical Research Grant**
```
Document Content: "NIH-funded study on cardiovascular outcomes with budget allocation for imaging equipment and legal consultation for patent filings..."

Model Prediction: Healthcare (confidence: 0.48)
True Domain: Healthcare-Financial-Legal
Error Analysis: Low confidence reflects genuine ambiguity
Root Cause: Balanced signals from three domains
Impact: Low - Correctly indicates uncertainty
```




