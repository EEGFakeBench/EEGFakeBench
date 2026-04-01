# EEGFakeBench

<p align="center">
  <img src="assets/mm_seed_arch.png" alt="MM-SEED Architecture" width="800"/>
</p>

---

> **EEGFakeBench: A Multi-Generator Benchmark for Synthetic EEG Detection**
>

---

## Overview

**EEGFakeBench** is the first comprehensive multi-generator benchmark dataset for 
**Synthetic EEG Detection (SynED)** — the task of detecting fully generative spoofing 
in EEG signals. Modern generative models can now synthesize EEG signals that are 
perceptually indistinguishable from genuine neural recordings, posing a serious threat 
to EEG-based systems such as biometric authentication, extended reality (XR), and 
neural gaming.

EEGFakeBench comprises **89,586 EEG recordings** — real EEG paired with synthetic 
signals generated across **8 distinct generative model families**, enabling systematic 
evaluation of detector generalization under realistic conditions where the synthesis 
method is unknown at inference time.

---

## Dataset Statistics

| Split | Real EEG | Synthetic EEG | Total |
|---|---|---|---|
| BCI IV-2a | 5,057 | 40,456 | 45,513 |
| EEGMMIDB | 4,897 | 39,176 | 44,073 |
| **Total** | **9,954** | **79,632** | **89,586** |

### Generative Model Families

| Family | Models |
|---|---|
| **GAN-based** | WGAN, TimeGAN |
| **Diffusion-based** | DS-DDPM |
| **Score-based** | Score-SDE |
| **Neural Audio Codecs (NACs)** | EnCodec, DAC, SoundStream, SNAC |

Each generative model produces **9,954 synthetic samples** (22 channels × 1000 timepoints at 250 Hz).

---

## Signal Specifications

| Property | Value |
|---|---|
| Channels | 22 EEG channels |
| Sampling Rate | 250 Hz |
| Segment Length | 1000 timepoints (4 seconds) |
| Format | NumPy arrays (.npy) |
| Preprocessing | Bandpass filtered 4–40 Hz |

---

## Benchmark Tasks

EEGFakeBench supports three evaluation paradigms:

### 1. Seen Evaluation
Models are trained and tested on all 8 generators with a **70/10/20** train/validation/test split.

### 2. Unseen Evaluation
Models are trained on **seen generators** and tested exclusively on **unseen generators**.

| Seen Generators | Unseen Generators |
|---|---|
| WGAN, DDPM, EnCodec, DAC | TimeGAN, SDE, SoundStream, SNAC |

Each generative paradigm is represented in both splits, ensuring generalization is evaluated within and across paradigms.

### 3. Leave-One-Generator-Out (LOGO)
Each of the 8 generators is held out in turn as the sole test generator. The model is trained on real EEG paired with fake samples from the remaining 7 generators. Repeated 8 times.

---

## Baselines

We provide implementations of the following baselines:

| Category | Model |
|---|---|
| EEG-Specific | EEGNet |
| Transformer | EEG Conformer |
| Audio Deepfake | AASIST |
| Unimodal Pretrained | REVE-only, AST-only |
| Multimodal | Late Fusion (REVE + AST) |
| **Ours** | **MM-SEED** |

### MM-SEED Architecture

**MM-SEED** (Multimodal Synthetic EEG Detection) is a bidirectional cross-modal 
attention framework that jointly models temporal and spectral EEG representations:

- **Temporal Branch:** Frozen REVE encoder (pretrained on 60,000 hours of EEG) 
  extracts spatiotemporal embeddings from the raw EEG waveform
- **Spectral Branch:** Frozen AST encoder extracts frequency-domain patch embeddings 
  from the EEG spectrogram
- **Bidirectional Cross-Modal Attention:** Allows each branch to query the other for 
  complementary forensic evidence
- **Fusion:** Global average pooling + concatenation + FCN classifier

---

## Main Results

| Model | Seen AUC ↑ | Seen EER ↓ | Unseen AUC ↑ | Unseen EER ↓ |
|---|---|---|---|---|
| EEGNet | 65.2 | 37.4 | 52.8 | 47.6 |
| EEG Conformer | 68.7 | 34.1 | 54.3 | 46.2 |
| AASIST | 62.4 | 40.2 | 51.3 | 48.9 |
| REVE-only | 74.1 | 29.3 | 57.2 | 44.1 |
| AST-only | 71.8 | 31.7 | 55.6 | 45.3 |
| Late Fusion | 76.9 | 26.8 | 59.4 | 42.3 |
| **MM-SEED (Ours)** | **79.0** | **24.2** | **62.7** | **39.2** |

All models exhibit unseen AUC within 51.3%–62.7%, confirming that **SynED remains 
a fundamentally open and challenging problem**. Extended LOGO results are available 
in the paper.

---

## Installation

```bash
git clone https://github.com/XXXXXXX/EEGFakeBench.git
cd EEGFakeBench
pip install -r requirements.txt
```

### Requirements

```
torch>=2.0.0
numpy>=1.24.0
scipy>=1.10.0
scikit-learn>=1.2.0
mne>=1.4.0
transformers>=4.30.0
pandas>=2.0.0
matplotlib>=3.7.0
```

---

## Dataset Download

The full EEGFakeBench dataset is available on HuggingFace:

```python
from datasets import load_dataset
dataset = load_dataset("XXXXXXX/EEGFakeBench")
```

Or download directly from [Zenodo](https://zenodo.org/record/XXXXXXX).

### Dataset Structure

```
EEGFakeBench/
├── real/
│   ├── BCICIV_2a/          # 5,057 samples
│   └── EEGMMIDB/           # 4,897 samples
├── synthetic/
│   ├── wgan/               # 9,954 samples
│   ├── timegan/            # 9,954 samples
│   ├── ddpm/               # 9,954 samples
│   ├── score_sde/          # 9,954 samples
│   ├── encodec/            # 9,954 samples
│   ├── dac/                # 9,954 samples
│   ├── soundstream/        # 9,954 samples
│   └── snac/               # 9,954 samples
└── splits/
    ├── seen_train.csv
    ├── seen_val.csv
    ├── seen_test.csv
    ├── unseen_test.csv
    └── logo/
        ├── logo_wgan.csv
        ├── logo_timegan.csv
        └── ...
```

Each sample is stored as a NumPy array of shape `(22, 1000)` — 22 channels × 1000 timepoints.

---

## Usage

### Loading a Sample

```python
import numpy as np

# Load a real EEG sample
real_sample = np.load('real/BCICIV_2a/sample_0001.npy')  # shape: (22, 1000)

# Load a synthetic EEG sample
fake_sample = np.load('synthetic/encodec/sample_0001.npy')  # shape: (22, 1000)
```

### Running MM-SEED

```python
from models.mm_seed import MMSEED

model = MMSEED(
    reve_checkpoint='pretrained/reve',
    ast_checkpoint='pretrained/ast',
    d_model=256,
    n_heads=8,
    dropout=0.4
)

# Input: raw EEG (B, 22, 1000) and spectrogram (B, 1, 128, 128)
logits = model(raw_eeg, spectrogram)
```

### Running Evaluation

```python
# Seen evaluation
python evaluate.py --paradigm seen --model mm_seed

# Unseen evaluation  
python evaluate.py --paradigm unseen --model mm_seed

# LOGO evaluation
python evaluate.py --paradigm logo --model mm_seed --generator encodec
```

---

## Forensic Analysis

Our forensic analysis reveals that synthetic EEG signals closely match authentic 
recordings across temporal, spectral, and time-frequency domains:

- **Score-SDE** achieves near-perfect signal fidelity — visually indistinguishable 
  from real EEG in all three domains
- **EnCodec** preserves the motor imagery band (0–40 Hz) but introduces high-frequency 
  quantization artifacts above 40 Hz via residual vector quantization (RVQ)

Extended forensic analysis across all 8 generators is available on the 
[project website](https://XXXXXXX.github.io/EEGFakeBench).

---

## Real EEG Data Sources

EEGFakeBench uses the following publicly available real EEG datasets as ground truth:

| Dataset | Subjects | Recordings | Channels | Sampling Rate |
|---|---|---|---|---|
| [BCI Competition IV-2a](https://www.bbci.de/competition/iv/) | 9 | 5,057 | 22 | 250 Hz |
| [EEGMMIDB (PhysioNet)](https://physionet.org/content/eegmmidb/1.0.0/) | 109 | 4,897 | 22 | 250 Hz |

---

## Citation

If you use EEGFakeBench in your research, please cite:

```bibtex
@inproceedings{XXXX2026eegfakebench,
  title     = {EEGFakeBench: A Multi-Generator Benchmark for Synthetic EEG Detection},
  author    = {XXXX},
  booktitle = {Proceedings of the 34th ACM International Conference on Multimedia},
  year      = {2026},
  publisher = {ACM},
  address   = {New York, NY, USA}
}
```

---

## License

EEGFakeBench is released under the [Creative Commons Attribution 4.0 International License](LICENSE).

The real EEG data sources are subject to their original licenses:
- BCI Competition IV-2a: Available for research use
- EEGMMIDB: Available under PhysioNet Credentialed Health Data License

---

## Acknowledgements

We thank the creators of the BCI Competition IV-2a and EEGMMIDB datasets, and the 
developers of REVE and AST for making their pretrained models publicly available.

---

## Contact

For questions about the dataset or benchmark, please open a GitHub issue or contact 
the authors at [XXXX@XXXX.edu].
