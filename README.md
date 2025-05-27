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

![System Framework](images/System_Framework.png)

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

### DCT-based Hashing (pHash)

1. Apply 2D DCT to the preprocessed frame  
2. Extract top-left n×n low-frequency block  
3. Compute its median μ  
4. Binarize coefficients against μ  
5. **Output**:  

### Difference Hashing (dHash)

1. Resize to (n+1)×n, convert to grayscale  
2. Compute horizontal pixel differences Δ  
3. Binarize:  

### Temporal Hashing

Compute bitwise XOR of consecutive dHash outputs:  

### Combining into THPH

Final per-frame hash:  
Converted to hexadecimal for compact storage.

---

## Experimental Setup

- **Hardware**: Multi-core CPU, 16 GB RAM  
- **Software**: Python 3.8, OpenCV, NumPy, SciPy, scikit-learn  
- **Frame Rate**: 30 FPS sampling  
- **Similarity Metric**:  
pip install -r requirements.txt
python run_thph.py \
  --input_dir data/original \
  --output_hashes hashes/original.json
python run_thph.py \
  --input_dir data/edited \
  --output_hashes hashes/edited.json
python compute_similarity.py \
  --orig hashes/original.json \
  --edit hashes/edited.json \
  --out results/similarity.csv
python plot_results.py --input results/similarity.csv
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
