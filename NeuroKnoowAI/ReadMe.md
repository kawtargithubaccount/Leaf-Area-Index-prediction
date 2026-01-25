
#  Document Processing, NLP, and Privacy – Problem Set Project

**Private GitHub repository with code [@moitreebasunkai]**

---

##  Project Overview

This project is a **set of four applied problems** focused on **document processing and natural language understanding**.
Each problem addresses a different real-world challenge involving **OCR, Named Entity Recognition (NER), NLP models, and data privacy**.

The goal of the project is to demonstrate **end-to-end document intelligence pipelines**, from raw document images to structured, privacy-aware information extraction.

---

##  Problem Breakdown

###  Problem 1: OCR-Based Document Text Extraction

**Objective:**
Extract text from scanned or image-based documents.

**Techniques & Tools:**

* Image preprocessing
* Optical Character Recognition (OCR)
* Noise reduction and thresholding

**Outcome:**
Reliable text extraction from document images to enable downstream NLP tasks.

---

###  Problem 2: Document Understanding with Layout-Aware NER

**Objective:**
Understand structured documents (e.g., forms) by identifying semantic regions such as:

* Questions
* Answers
* Headers

**Techniques & Tools:**

* Layout-aware transformers (LayoutLMv3)
* Token-level classification
* Multimodal learning (text + layout + vision)

**Outcome:**
Accurate semantic labeling of document elements using spatial and visual context.

---

###  Problem 3: Multilingual Financial NER

**Objective:**
Detect financial entities in multilingual text documents.

**Techniques & Tools:**

* Transformer-based NER (XLM-RoBERTa)
* Multilingual token classification
* Evaluation using precision, recall, and F1-score

**Outcome:**
Robust recognition of financial entities across multiple languages.

---

###  Problem 4: Privacy-Aware NLP (PII Detection)

**Objective:**
Identify Personally Identifiable Information (PII) in documents to support privacy protection.

**Techniques & Tools:**

* Named Entity Recognition for PII
* Synthetic privacy-preserving datasets
* Secure handling of sensitive information

**Outcome:**
Automated detection of sensitive data to support compliance and privacy-preserving document processing.

---

##  Technologies Used

* **Python**
* **PyTorch**
* **Hugging Face Transformers**
* **OCR (Tesseract / vision-based models)**
* **LayoutLMv3**
* **XLM-RoBERTa**
* **scikit-learn**
* **Datasets (Hugging Face)**

---

##  Evaluation

Models are evaluated using:

* Token-level **precision**
* **Recall**
* **F1-score**
* Qualitative inspection of predictions

Evaluation is performed on **held-out test sets** without retraining.

---

##  Privacy & Data Handling

* No real personal data is stored in this repository
* Synthetic datasets are used for training and evaluation
* The repository is **private** to ensure controlled access

---



##  Project Structure

```
src/
├── prb1.ipynb       
├── prb1.md     
├── prb2.ipynb     
├── prb2.md     
└
```

---

##  Summary

This project demonstrates how **OCR, NLP, NER, and privacy-aware modeling** can be combined to build intelligent document processing systems.
Each problem addresses a different aspect of real-world document understanding, making the project both **practical and comprehensive**.

