# Multimodal Document Understanding

# 0 Dataset
we choosed this dataset [FUNSD](https://huggingface.co/datasets/nielsr/funsd) it's a finance dataset that contain images of invoicement and infomations about positions of words, labels.....


# 1️ Model / System Architecture Diagram
## Model N°1

### Architecture of the Model (OCR Pipeline)

     
     Analyzing document structure (Analyze document layout based on bounding boxes)
        │
        ▼

     Extracting entities (Extract entities from FUNSD-style BIO labels Keeps text + bounding boxes)
        │
        ▼

     Extracting key-value pairs (Extract question-answer pairs from the document)
        │
        ▼

     Building structured output (Build final structured output from all extracted information)

### Explaination 
this model analzse all steps of the second parrt of tho problem :
""" 2. Information Extraction """ where we extract pairs of question/answer, entites ..... from the images of the invoices in our dataset without using anz pretrained models 

### Results
the results was not satisfing cause of the missunderstanding of the content of the images aand also the problem of distinging between labels HEADER. QUESTION...... (you can find the results in the nootbook ith details) 

## Model N°2: LayoutLMv3 Token Classification Model
### Architecture of the Model 


Dataset (FUNSD)
(image, words, bounding boxes, NER tags)
        │
        ▼
Encoding & Alignment
(LayoutLMv3Processor)
 ├─ Image → pixel_values
 ├─ Text → input_ids
 ├─ Layout → bounding boxes
 ├─ Token–label alignment
        │
        ▼
Multimodal Transformer
(LayoutLMv3)
 ├─ Visual embeddings
 ├─ Text embeddings
 ├─ Layout embeddings
        │
        ▼
Token Classification Head
(Linear layer)
        │
        ▼
Predicted Labels
(O, QUESTION, ANSWER, HEADER)

### Explaination 
Each training example contains: Image: scanned document page, Words: OCR-extracted text, Bounding boxes: word-level spatial coordinates, NER tags: semantic labels then we go to The processor  that simultaneously Tokenizes text, Encodes layout information, Extracts visual embeddings, Pads/truncates sequences
FOR TRAINNING we used these params 
| Parameter     | Value |
| ------------- | ----- |
| Optimizer     | AdamW |
| Learning Rate | 5e-5  |
| Epochs        | 10    |
| Batch Size    | 2     |
| Dataset       | FUNSD |

### Results

LOSS: 
| Epoch | Avg Loss   |
| ----- | ---------- |
| 1     | 1.2296     |
| 5     | 0.3501     |
| 10    | **0.1196** |
METRICS:
| Metric      | Value      |
| ----------- | ---------- |
| Accuracy    | **77.53%** |
| Macro F1    | **0.7410** |
| Weighted F1 | **0.7799** |
F1 SCORE for each label
| Label          | F1-Score   | Interpretation                   |
| -------------- | ---------- | -------------------------------- |
| **B-ANSWER**   | **0.8871** | Very strong semantic recognition |
| **B-HEADER**   | **0.8788** | Layout cues well learned         |
| **I-HEADER**   | **0.8285** | Good sequence consistency        |
| **O**          | 0.7223     | Background tokens harder         |
| **I-QUESTION** | **0.5118** | Most difficult class             |
 
The proposed system fine-tunes LayoutLMv3 for token-level classification on structured document images. By jointly encoding textual content, spatial layout, and visual information, the model effectively learns semantic roles such as questions, answers, and headers. Training on the FUNSD dataset demonstrates stable convergence and strong performance, achieving a weighted F1-score of 0.78. Results indicate that visually and spatially distinctive classes such as headers and answers are recognized with high accuracy, while more ambiguous token spans such as in-question text remain challenging. Overall, the model confirms the effectiveness of multimodal transformers for document understanding tasks.

# 3 Performance bottlenecks and optimization strategies
## performance botleneck:
### Bottleneck 1: Multimodal Transformer Complexity
Cause
LayoutLMv3 processes:
    Text tokens
    Bounding boxes
    Image patches (vision encoder)
This significantly increases:
    Number of embeddings
    Self-attention computation
    GPU memory usage

Symptoms
Slow training (~2.2 iterations/sec)
    Small batch size (2)
    High VRAM consumption
Optimization Strategies
    Use gradient accumulation to simulate larger batch sizes
    Enable mixed-precision training (FP16)
    Reduce max_length if documents are short

### Bottleneck 2: Small Batch Size
Cause
High memory footprint per example
GPU VRAM limitations

Impact
Noisy gradients
Longer convergence time

Optimization Strategies
Gradient accumulation
Reduce image resolution
Switch to layoutlmv3-small (if available)

### Bottleneck 3: Class Imbalance
Cause
Dominance of O and I-HEADER
Rare labels (e.g., B-QUESTION)

Impact
Lower recall for minority classes
Reduced macro F1-score

Optimization Strategies
Class-weighted loss
Focal loss
Oversampling rare classes

## Inference-Time Bottlenecks
Bounding Box Density
Many small words → long sequences
Increases attention complexity

Optimization
Merge adjacent tokens
Filter low-confidence OCR tokens