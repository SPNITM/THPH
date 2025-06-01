# Temporal Hybrid Perceptual Hashing (THPH)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

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

### DCT-based Hashing (pHash)

1. Apply 2D DCT to the preprocessed frame  
2. Extract top-left n×n low-frequency block  
3. Compute its median μ  
4. Binarize coefficients against μ  
5. **Output**:  
H_DCT(F_t) = bin(D[1:n,1:n] > μ)

### Difference Hashing (dHash)

1. Resize to (n+1)×n, convert to grayscale  
2. Compute horizontal pixel differences Δ  
3. Binarize:  
H_dHash(F_t) = bin(Δ > 0)

### Temporal Hashing

Compute bitwise XOR of consecutive dHash outputs:  
H_temporal(F_t) = H_dHash(F_t) ⊕ H_dHash(F_{t-1})


### Combining into THPH

Final per-frame hash:  
H_THPH(F_t) = [ H_DCT(F_t) | H_temporal(F_t) ]

Converted to hexadecimal for compact storage.

---

## Experimental Setup

- **Hardware**: Multi-core CPU, 16 GB RAM  
- **Software**: Python 3.8, OpenCV, NumPy, SciPy, scikit-learn  
- **Frame Rate**: 30 FPS sampling  
- **Similarity Metric**:  
s_t = 1 – (HammingDistance / HashLength)


## Datasets & Operations

We curated a diverse video set covering:

- **Filter Edits**: Blur, Sharpen, Smooth, Edge-Detect, Color Filter  
- **Transformation Edits**: Compression, Rotation, Resize, Crop, Flip, Perspective Transform
- **Temporal Edits**: Speed Change, Frame-Rate Conversion, Frame-Drop, Loop  
- **Spatial Edits**: Translation, Zoom, Affine-Transform

## Results & Analysis

### Filter Operations

![Filter Operations](Figure%204%20Original%20Video%20Frame%20with%20Filter%20Operations.png)


| Operation      | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|---------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Blurred**        | 0.9989  | 0.9973  | 0.9805  | 0.9984  | 0.9712  | **0.9991** |
| **Sharpened**      | 0.9977  | 0.9962  | 0.9797  | 0.9975  | 0.9563  | **0.9985** |
| **Smoothed**       | 0.9982  | 0.9969  | **0.9989** | 0.9980  | 0.9739  | 0.9891   |
| **Edges**          | 0.8874  | 0.8966  | 0.8957  | 0.8942  | 0.8841  | **0.9531** |
| **Color-Filtered** | 0.8816  | 0.8833  | 0.8853  | 0.8900  | 0.8775  | **0.8961** |

### Spatial Transformations

![Spatial Operations](Figure%207%20Original%20Video%20Frame%20with%20Spatial%20Operations.png)


| Operation   | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Translated**  | 0.8927  | 0.9330  | 0.8644  | 0.9211  | 0.8838  | **0.9342** |
| **Zoomed**      | 0.8910  | 0.9241  | 0.8683  | 0.8980  | 0.8913  | **0.9293** |
| **Affine**      | 0.8941  | 0.9207  | 0.8618  | 0.9025  | 0.8817  | **0.9242** |

### Temporal Operations

![Temporal Operations](Figure%206%20Original%20Video%20Frame%20with%20Temporal%20Operations.png)


| Operation             | pHash   | wHash   | aHash   | dHash   | KL Hash | **THPH** |
|----------------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| **Slowed Down**         | 0.9988  | 0.9972  | 0.9891  | 0.9984  | 0.9838  | **0.9991** |
| **Frame-Rate Converted**| 0.9988  | 0.9972  | 0.9891  | 0.9984  | 0.9838  | **0.9991** |
| **Frame Dropped**       | —       | —       | —       | —       | —       | **0.9954** |
| **Looped**              | —       | —       | —       | —       | —       | **0.9891** |

### Transformation Operations

![Transformation Operations](Figure%205%20Original%20Video%20Frame%20with%20Transformation%20Operations.png)

### Transformation Operations

| Video Operation         | pHash   | wHash   | aHash     | dHash   | KL Hash | THPH      |
|-------------------------|:-------:|:-------:|:---------:|:-------:|:-------:|:---------:|
| **Compressed**              | 0.9985  | 0.9966  | **0.9988** | 0.9980  | 0.9779  | 0.9930    |
| **Rotated**                 | 0.8815  | 0.8876  | 0.8927    | 0.8918  | 0.8826  | **0.9539** |
| **Resized**                 | 0.9986  | 0.9968  | **0.9989** | 0.9983  | 0.9766  | 0.9867    |
| **Cropped**                 | 0.8853  | 0.8991  | 0.9034    | 0.8841  | 0.8886  | **0.9281** |
| **Flipped Horizontal**      | 0.8792  | 0.8979  | 0.9051    | 0.8947  | 0.9838  | **0.9844** |
| **Flipped Vertical**        | 0.8793  | 0.9097  | 0.9124    | **0.9160** | 0.8657  | 0.9000    |
| **Perspective Transformed** | 0.8901  | 0.9078  | 0.9114    | 0.9068  | 0.8805  | **0.9477** |


### Comparison with Existing Methods

| Framework / Paper      | Methodology                       | Real-Time | Geometry Robustness | Notes               |
|------------------------|-----------------------------------|-----------|---------------------|---------------------|
| Yang *et al.* (2024)   | SURF+KLT Graph + Min-Cut          | 26 FPS    | Moderate            | Deep-learning light |
| Mendes *et al.* (2024) | Structural Tensor + GA            | N/A       | Low                 | No speed data       |
| **This Work (THPH)**   | PCA + DCT + dHash + Temporal XOR  | Yes       | **High**            | No deep nets        |

---

# THPH: Temporal Hashing for Video Processing

## 📥 Clone the repository

```bash
git clone https://github.com/SPNITM/THPH.git
cd THPH
```

## 📦 Install dependencies

Make sure you are using **Python ≥ 3.7**, then run:

```bash
pip install -r requirements.txt
```

**Dependencies in `requirements.txt`:**
- `opencv-python`
- `Pillow`
- `ImageHash`
- `numpy`
- `scipy`
- `scikit-learn`

## 🗂️ Project Structure

```
├── Dataset/                              # Contains input video files
│   ├── original/                         # Raw video sequences
├── Comparison.ipynb                      # Notebook for comparing techniques
├── Figure 4–7 Original Video Frames      # Notebooks with different operations
├── README.md                             # Project documentation
├── THPH Final.ipynb                      # Final notebook version
├── Video Frame.ipynb                     # Frame extraction/processing
├── requirements.txt                      # List of dependencies
├── run_thph.py                           # Main THPH hashing script (expected)
├── compute_similarity.py                 # Computes similarity from hashes
├── plot_results.py                       # Plots similarity results
└── LICENSE                               # License file
```

## 👥 Authors & Acknowledgements

- **Stuti Pandey** — NIT Meghalaya — Lead author  
- **Akhilendra Pratap Singh** — NIT Meghalaya  
- **Dharmender Singh Kushwaha** — MNNIT Allahabad  

_Thanks to the Computer Science departments of **NIT Meghalaya** and **MNNIT Allahabad** for their support._

## 📄 License

This project is licensed under the **MIT License**.

##License
This project is licensed under the MIT License.
