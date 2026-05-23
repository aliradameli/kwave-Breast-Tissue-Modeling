# Acoustic and Tissue Simulation for Breast Tumor Detection using k-Wave

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![k-Wave](https://img.shields.io/badge/Simulation-k--Wave-green.svg)](https://github.com/ucl-bug/k-wave-python)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository contains a full computational simulation pipeline designed to model multi-layered biological tissues (specifically human breast anatomy) and perform ultrasonic full-waveform modeling. By computing customized continuous distribution models of standard tissue property bounds, it models sound propagation through heterogeneous structures and lesions.

The pipeline simulates a physical coupling layer, an epidermis layer, a dermis layer, general background fat/subcutaneous tissue matrices, and populates randomly shaped and oriented physiological tumors based on recent deep learning ultrasound reconstruction literature.

---

## 📖 Table of Contents
* [Features](#-features)
* [Mathematical & Tissue Model Specifications](#-mathematical--tissue-model-specifications)
* [Simulation Workflow](#-simulation-workflow)
* [Dependencies & Requirements](#-dependencies--requirements)
* [Usage Guide](#-usage-guide)
* [Visualizing Outputs](#-visualizing-outputs)
* [References](#-references)

---

## ✨ Features

- **Automated Physical-to-Pixel Mapping:** Converts physical layer thicknesses directly from micrometers ($\mu\text{m}$) into custom spatial computational grids matching standard acoustic parameters.
- **Stochastic Heterogeneity Optimization:** Applies normal Gaussian noise distribution models to the standard dynamic limits of tissue parameters to represent real biological porosity and acoustic backscattering texture.
- **Multi-Layer Synthetic Tissue Generation (`BreastMap`):** Pipeline mimics natural medical constraints sequentially:
  1. **Coupling Gel Layer:** Simulates Propylene Glycol layer dynamics.
  2. **Epidermis Layer:** Models thin skin layer bounds based on dermatological clinical benchmarks.
  3. **Dermis Layer:** Simulates dense core skin variations.
  4. **Substance Tissue:** Fills remaining space dynamically with fat/muscle backgrounds.
  5. **Tumor Pathology Injection:** Randomly populates varying quantities of elliptical lesions mimicking solid malignant structures or fluid-filled cysts.
- **Advanced Acoustic Field Modeling:** Built on top of the first-order **k-space** pseudo-spectral numerical method via the `kspaceFirstOrder2D` execution engine.

---

## 🔬 Mathematical & Tissue Model Specifications

The tissue structures generated across the spatial computational grid assign randomized bounds compiled from medical imaging literature:

| Tissue Layer / Component | Physical Thickness Layer Range | Speed of Sound (SoS) Range ($c_0$) | Mass Density Range ($\rho_0$) |
| :--- | :--- | :--- | :--- |
| **Propylene Glycol Gel** | $1 \text{ pixel}$ | $1467.8 \rightarrow 1541.1 \text{ m/s}$ | $1036 \rightarrow 1040 \text{ kg/m}^3$ |
| **Epidermis** | $84.5 \rightarrow 156.7 \ \mu\text{m}$ | $1531.9 \rightarrow 1735.5 \text{ m/s}$ | $1095.1 \rightarrow 1122.9 \text{ kg/m}^3$ |
| **Dermis** | $1005.6 \rightarrow 1670.2 \ \mu\text{m}$ | $1568.9 \rightarrow 1680.5 \text{ m/s}$ | $1095.1 \rightarrow 1122.8 \text{ kg/m}^3$ |
| **Subcutaneous Background** | Dynamic Rest | $1450.0 \rightarrow 1540.0 \text{ m/s}$ | $950.0 \rightarrow 1020.0 \text{ kg/m}^3$ |
| **Lesion (Solid Malignant)** | Elliptical Form | $1550.0 \rightarrow 1700.0 \text{ m/s}$ | $1040.0 \rightarrow 1100.0 \text{ kg/m}^3$ |
| **Lesion (Cyst/Fatty Mass)** | Elliptical Form | $1300.0 \rightarrow 1450.0 \text{ m/s}$ | $900.0 \rightarrow 940.0 \text{ kg/m}^3$ |

### Elliptical Lesion Geometry
The spatial boundary transformations for modeling tumor structures utilize an oriented quadratic matrix condition:

$$\frac{X_{\text{rot}}^2}{r_x^2} + \frac{Y_{\text{rot}}^2}{r_y^2} \le 1$$

Where spatial positions undergo translation and coordinate matrix rotation via an incident angle ($\theta$):

$$X_{\text{rot}} = (X - c_x)\cos(\theta) + (Y - c_y)\sin(\theta)$$

$$Y_{\text{rot}} = -(X - c_x)\sin(\theta) + (Y - c_y)\cos(\theta)$$

---

## ⚙️ Simulation Workflow

The script uses a clear modular method-chaining structure to create datasets step-by-step:

```mermaid
graph LR
    A[Initialize Grid Specs] --> B[BreastMap Instance]
    B --> C[addGel]
    C --> D[addEpidermis]
    D --> E[addDermis]
    E --> F[addSubstance]
    F --> G[addTumors]
    G --> H[Extract SoS & Density Maps]
    H --> I[k-Space Wave Execution]
