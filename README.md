# Temporal Hybrid Perceptual Hashing (THPH)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Table of Contents

- [Overview](#overview)
- [Key Contributions](#key-contributions)
- [System Architecture](#system-architecture)
- [Methodology](#methodology)
  - [PCA-based Hashing (KLT)](#pca-based-hashing-klt)
  - [DCT-based Hashing (pHash)](#dct-based-hashing-phash)
  - [Difference Hashing (dHash)](#difference-hashing-dhash)
  - [Temporal Hashing](#temporal-hashing)
  - [Combining into THPH](#combining-into-thph)
- [Experimental Setup](#experimental-setup)
- [Datasets & Operations](#datasets--operations)
- [Results & Analysis](#results--analysis)
  - [Filter Operations](#filter-operations)
  - [Spatial Transformations](#spatial-transformations)
  - [Temporal Operations](#temporal-operations)
  - [Transformation Operations](#transformation-operations)
  - [Comparison with Existing Methods](#comparison-with-existing-methods)
- [How to Reproduce](#how-to-reproduce)
- [Project Structure](#project-structure)
- [Dependencies](#dependencies)
- [Authors & Acknowledgements](#authors--acknowledgements)
- [License](#license)

---

## Overview

**Temporal Hybrid Perceptual Hashing (THPH)** is a novel, lightweight framework designed for robust video similarity assessment and piracy detection. By fusing spatial hashing techniques (PCA / DCT / Difference Hash) with a frame-to-frame temporal analysis, THPH achieves:

- **High robustness** against common edits (blur, sharpen, compression, resizing).  
- **Sensitivity** to meaningful alterations (color filters, geometric transforms, cropping).  
- **Real-time capability**, suitable for live surveillance, digital forensics, and media management.  
- **Scalability** across high-resolution or large video archives.  

---

## Key Contributions

1. **Hybrid Spatial–Temporal Hash**  
   Combines PCA-based (KLT), DCT-based (pHash), and Difference Hash (dHash) features with a temporal XOR-based differential hash.  

2. **Real-Time Performance**  
   Optimized for immediate per-frame processing; tested at 30 FPS on multi-core systems.  

3. **Comprehensive Evaluation**  
   Benchmarked across 20+ video operations (filtering, spatial, temporal, geometric) and compared against pHash, wHash, aHash, dHash, and KL Hash.  

4. **Open Architecture**  
   Modular design with clear APIs for ingestion, hashing, comparison, and alerting.  

---

## System Architecture

![System Framework](System_Framework _1.drawio.png)

1. **Video Capture & Preprocessing**  
   - Frame extraction at configurable FPS  
   - Grayscale conversion, resizing, noise reduction  

2. **Perceptual Hashing Module**  
   - PCA-based (KLT) → captures dominant components  
   - DCT-based (pHash) → captures low-frequency patterns  
   - Difference Hash (dHash) → captures local gradients  

3. **Temporal Analysis**  
   - XOR of consecutive dHash outputs to detect inter-frame changes  

4. **Hash Comparison & Alerting**  
   - Hamming-distance based similarity score  
   - Thresholding for anomaly/tamper detection  
   - Dashboard & API integration  

---

## Methodology

### PCA-based Hashing (KLT)

1. Flatten each 64×64 frame into a vector **x**  
2. Compute covariance matrix and its top-k eigenvectors **P**  
3. Project **x** onto **P** and binarize  
4. **Output**:  
H_PCA(F_t) = bin(Pᵀ · x)

markdown
Copy
Edit

### DCT-based Hashing (pHash)

1. Apply 2D DCT to the preprocessed frame  
2. Extract top-left n×n low-frequency block  
3. Compute its median μ  
4. Binarize coefficients against μ  
5. **Output**:  
H_DCT(F_t) = bin(D[1:n,1:n] > μ)

markdown
Copy
Edit

### Difference Hashing (dHash)

1. Resize to (n+1)×n, convert to grayscale  
2. Compute horizontal pixel differences Δ  
3. Binarize:  
H_dHash(F_t) = bin(Δ > 0)

shell
Copy
Edit

### Temporal Hashing

Compute bitwise XOR of consecutive dHash outputs:  
H_temporal(F_t) = H_dHash(F_t) ⊕ H_dHash(F_{t-1})

sql
Copy
Edit

### Combining into THPH

Final per-frame hash:  
H_THPH(F_t) = [ H_DCT(F_t) | H_temporal(F_t) ]

yaml
Copy
Edit
Converted to hexadecimal for compact storage.

---

## Experimental Setup

- **Hardware**: Multi-core CPU, 16 GB RAM  
- **Software**: Python 3.8, OpenCV, NumPy, SciPy, scikit-learn  
- **Frame Rate**: 30 FPS sampling  
- **Similarity Metric**:  
s_t = 1 – (HammingDistance / HashLength)

ruby
Copy
Edit

---

## Datasets & Operations

We curated a diverse video set covering:

- **Filter Edits**: Blur, Sharpen, Smooth, Edge-detect, Color Filter  
- **Spatial Transforms**: Crop, Resize, Translate, Flip, Zoom, Affine, Perspective  
- **Temporal Edits**: Frame-drop, Frame-rate change, Speed change, Loop  
- **Others**: Compression, Rotation  

---

## Results & Analysis

### Filter Operations

![Filter Operations](Filter_Comparison (1).png)

| Operation      | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|---------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Blurred**        | 0.9989  | 0.9973  | 0.9805  | 0.9984  | 0.9712  | **0.9991** |
| **Sharpened**      | 0.9977  | 0.9962  | 0.9797  | 0.9975  | 0.9563  | **0.9985** |
| **Smoothed**       | 0.9982  | 0.9969  | **0.9989** | 0.9980  | 0.9739  | 0.9891   |
| **Edges**          | 0.8874  | 0.8966  | 0.8957  | 0.8942  | 0.8841  | **0.9531** |
| **Color-Filtered** | 0.8816  | 0.8833  | 0.8853  | 0.8900  | 0.8775  | **0.8961** |

### Spatial Transformations

![Spatial Operations](Filter Operations Diagram.drawio (1).png)

| Operation   | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Translated**  | 0.8927  | 0.9330  | 0.8644  | 0.9211  | 0.8838  | **0.9342** |
| **Zoomed**      | 0.8910  | 0.9241  | 0.8683  | 0.8980  | 0.8913  | **0.9293** |
| **Affine**      | 0.8941  | 0.9207  | 0.8618  | 0.9025  | 0.8817  | **0.9242** |

### Temporal Operations

![Temporal Operations](Temporal_Comparison.png)

| Operation             | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|----------------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Slowed Down**         | 0.9988  | 0.9972  | 0.9891  | 0.9984  | 0.9838  | **0.9991** |
| **Frame-Rate Converted**| 0.9988  | 0.9972  | 0.9891  | 0.9984  | 0.9838  | **0.9991** |
| **Frame Dropped**       | —       | —       | —       | —       | —       | **0.9954** |
| **Looped**              | —       | —       | —       | —       | —       | **0.9891** |

### Transformation Operations

![Transformation Operations](Transformation_Comparison.png)

| Operation        | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|-----------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Rotated**      | 0.8815  | 0.8876  | 0.8927  | 0.8918  | 0.8826  | **0.9539** |
| **Perspective**  | 0.8901  | 0.9078  | 0.9114  | 0.9068  | 0.8805  | **0.9477** |

### Comparison with Existing Methods

| Framework / Paper      | Methodology                       | Real-Time | Geometry Robustness | Notes               |
|------------------------|-----------------------------------|-----------|---------------------|---------------------|
| Yang *et al.* (2024)   | SURF+KLT Graph + Min-Cut          | 26 FPS    | Moderate            | Deep-learning light |
| Mendes *et al.* (2024) | Structural Tensor + GA            | N/A       | Low                 | No speed data       |
| **This Work (THPH)**   | PCA + DCT + dHash + Temporal XOR  | Yes       | **High**            | No deep nets        |

---

## How to Reproduce

1. **Clone the repository**  
 ```bash
 git clone https://github.com/your-org/THPH.git
 cd THPH
Install dependencies

bash
Copy
Edit
pip install -r requirements.txt
Prepare your videos

Place raw videos in data/original/

Place edited videos in data/edited/

Run the hashing pipeline

bash
Copy
Edit
python run_thph.py \
  --input_dir data/original \
  --output_hashes hashes/original.json
python run_thph.py \
  --input_dir data/edited \
  --output_hashes hashes/edited.json
Compute similarity scores

bash
Copy
Edit
python compute_similarity.py \
  --orig hashes/original.json \
  --edit hashes/edited.json \
  --out results/similarity.csv
Visualize results

bash
Copy
Edit
python plot_results.py --input results/similarity.csv
Project Structure
graphql
Copy
Edit
├── data/
│   ├── original/          # Raw video sequences
│   └── edited/            # Edited test videos
├── images/                # Plots & diagrams
├── src/
│   ├── run_thph.py        # Main THPH hashing script
│   ├── compute_similarity.py
│   ├── plot_results.py
│   └── utils/             # Helper modules
├── results/
├── requirements.txt
├── README.md
└── LICENSE
Dependencies
Python ≥ 3.7

OpenCV

NumPy

SciPy

scikit-learn

matplotlib

Install with:

bash
Copy
Edit
pip install -r requirements.txt
##Authors & Acknowledgements
Stuti Pandey — NIT Meghalaya — Lead author

Akhilendra Pratap Singh — NIT Meghalaya

Dharmender Singh Kushwaha — MNNIT Allahabad

We thank the Computer Science departments of NIT Meghalaya and MNNIT Allahabad for computational resources.

##License
This project is licensed under the MIT License.
