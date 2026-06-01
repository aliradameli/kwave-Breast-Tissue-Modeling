# k-Wave Breast Tissue Modeling

**Procedural 2D ultrasound simulation of multi-layered human breast tissue using k-Wave**

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![Simulation: k-Wave](https://img.shields.io/badge/Simulation-k--Wave-green.svg)](https://github.com/ucl-bug/k-wave-python)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/aliradameli/kwave-Breast-Tissue-Modeling/blob/main/Tissue_Simulation.ipynb)

---

## Overview

This project implements a full computational pipeline for simulating ultrasound wave propagation through a synthetic, anatomically-grounded model of human breast tissue. It is designed to support deep learning research in ultrasound reconstruction and medical image synthesis by generating realistic, physics-based training data.

The pipeline builds a spatially heterogeneous medium layer by layer — coupling gel, epidermis, dermis, subcutaneous fat — and populates it with stochastically generated tumors (solid malignant or cystic). Acoustic properties at each point are drawn from clinical literature distributions, not constant values, making the resulting maps representative of real biological variability. Full-waveform simulation is then executed via the k-space pseudospectral method (`kspaceFirstOrder2D`).

---

## Features

- **Anatomically grounded tissue geometry** — five sequential layers derived from dermatological and acoustic measurement literature, with physical thicknesses mapped to grid pixels.
- **Stochastic property distributions** — speed of sound and mass density at each layer are sampled from Gaussian-bounded clinical ranges, capturing biological variability and acoustic backscattering texture.
- **Elliptical tumor injection** — randomly oriented, randomly sized ellipsoidal lesions are placed in the subcutaneous background, modelling both solid malignant masses and fluid-filled cysts.
- **Fluent builder API (`BreastMap`)** — chainable method interface (`addGel → addEpidermis → addDermis → addSubstance → addTumors`) keeps the simulation setup readable and modular.
- **k-space wave execution** — built on `k-wave-python` (`kspaceFirstOrder2D`), a well-validated pseudospectral solver for heterogeneous acoustic media.
- **Colab-ready** — the notebook runs end-to-end in Google Colab with no local installation required.

---

## Tissue Model

Each layer's acoustic properties are sampled from clinically measured bounds:

| Layer | Thickness | Speed of Sound (m/s) | Density (kg/m³) |
|---|---|---|---|
| Coupling Gel (Propylene Glycol) | 1 px | 1467.8 – 1541.1 | 1036 – 1040 |
| Epidermis | 84.5 – 156.7 µm | 1531.9 – 1735.5 | 1095.1 – 1122.9 |
| Dermis | 1005.6 – 1670.2 µm | 1568.9 – 1680.5 | 1095.1 – 1122.8 |
| Subcutaneous Fat / Background | Remaining grid | 1450.0 – 1540.0 | 950.0 – 1020.0 |
| Solid Malignant Lesion | Elliptical | 1550.0 – 1700.0 | 1040.0 – 1100.0 |
| Cystic / Fatty Lesion | Elliptical | 1300.0 – 1450.0 | 900.0 – 940.0 |

### Elliptical Tumor Geometry

Tumor boundaries are defined by a rotated ellipse condition:

$$\frac{X_{\text{rot}}^2}{r_x^2} + \frac{Y_{\text{rot}}^2}{r_y^2} \leq 1$$

with spatial coordinates rotated by angle $\theta$ around center $(c_x, c_y)$:

$$X_{\text{rot}} = (X - c_x)\cos\theta + (Y - c_y)\sin\theta$$
$$Y_{\text{rot}} = -(X - c_x)\sin\theta + (Y - c_y)\cos\theta$$

This allows tumors of arbitrary orientation and aspect ratio to be injected anywhere in the subcutaneous layer.

---

## Simulation Pipeline

```
Grid Parameters
     │
     ▼
BreastMap()
     │
     ├─ .addGel()          ← coupling layer (1 px, propylene glycol)
     ├─ .addEpidermis()    ← thin outer skin
     ├─ .addDermis()       ← dense inner skin
     ├─ .addSubstance()    ← fat / subcutaneous background
     └─ .addTumors()       ← stochastic elliptical lesions
          │
          ▼
    SoS map + Density map
          │
          ▼
   kspaceFirstOrder2D
          │
          ▼
    Pressure field output
```

---

## Repository Contents

```
kwave-Breast-Tissue-Modeling/
├── Tissue_Simulation.ipynb          # Main simulation notebook
├── Acoustic and Physical Characterization...pdf   # Supporting technical note
└── README.md
```

---

## Getting Started

### Run in Google Colab (recommended)

Click the badge at the top of this README, or open:

```
https://colab.research.google.com/github/aliradameli/kwave-Breast-Tissue-Modeling/blob/main/Tissue_Simulation.ipynb
```

No local setup is needed. All dependencies are installed within the notebook.

### Run Locally

**Requirements:** Python 3.8+

```bash
git clone https://github.com/aliradameli/kwave-Breast-Tissue-Modeling.git
cd kwave-Breast-Tissue-Modeling
pip install k-wave-python numpy matplotlib
jupyter notebook Tissue_Simulation.ipynb
```

---

## Background & Motivation

Generating ground-truth acoustic maps for training deep learning ultrasound reconstruction models requires simulated data that respects real tissue physics. Simple homogeneous or hand-crafted phantoms fail to capture the spatial variability that neural networks will encounter in clinical data. This pipeline addresses that gap by:

- Drawing layer thicknesses and acoustic properties from published clinical measurements rather than single nominal values.
- Using procedural randomization so each simulation run produces a unique but physically plausible phantom.
- Encoding lesion geometry as parametric ellipses with random position, scale, and orientation, matching the morphological diversity seen in clinical B-mode imaging.

---

## References

1. Griffiths et al. (Eds.), *Rook's Textbook of Dermatology*, 10th ed., Wiley-Blackwell, 2024.
2. Moran, C. M., Bush, N. L., & Bamber, J. C. — Ultrasonic propagation properties of excised human skin. *Ultrasound in Med. & Biol.*, 21(9), 1177–1190, 1995.
3. Oltulu, P. et al. — Measurement of epidermis, dermis, and total skin thicknesses from six body regions. *Turk J Plast Surg*, 2018.
4. Hwang et al. — Skin thickness of Korean adults. *Surgical and Radiologic Anatomy*, 24, 183–189, 2002.
5. IT'IS Foundation — Tissue Properties Database V5.
6. Thol, M. et al. — Speed-of-sound measurements and fundamental equation of state for propylene glycol. *J. Phys. Chem. Ref. Data*, 50, 023105, 2021.
7. REMPEC MIDSIS — Propylene Glycol Chemical Data.
8. Treeby, B. E. & Cox, B. T. — k-Wave: MATLAB toolbox for simulation and reconstruction of photoacoustic wave-fields. *J. Biomed. Opt.*, 15(2), 021314, 2010.
9. Treeby, B. E. et al. — Modeling nonlinear ultrasound propagation in heterogeneous media with power law absorption using a k-space pseudospectral method. *J. Acoust. Soc. Am.*, 131(6), 4324–4336, 2012.
10. Treeby, B. E. et al. — Modelling elastic wave propagation using the k-Wave MATLAB toolbox. *IEEE IUS*, pp. 146–149, 2014.
11. Treeby, B. E. et al. — Rapid calculation of acoustic fields from arbitrary continuous-wave sources. *J. Acoust. Soc. Am.*, 143(1), 529–537, 2018.

---

## License

[MIT](https://opensource.org/licenses/MIT)
