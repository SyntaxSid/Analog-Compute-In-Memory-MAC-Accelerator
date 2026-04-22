# ⚡ Analog Compute-in-Memory MAC Accelerator
### Micron Memory Awards 2025 — Device Design & Development

[![Competition](https://img.shields.io/badge/Micron_Memory_Awards-2025-00A3E0?style=flat)](https://www.micron-mimoryawards.org.tw/en/)
[![Category](https://img.shields.io/badge/Category-Device_Design_%26_Development-green?style=flat)]()
[![Technology](https://img.shields.io/badge/Technology-180nm_CMOS-orange?style=flat)]()
[![Tool](https://img.shields.io/badge/Simulation-LTSpice-8B0000?style=flat)]()
[![Team ID](https://img.shields.io/badge/Team_ID-2526__C__0061-lightgrey?style=flat)]()
[![Demo](https://img.shields.io/badge/▶_Watch_Demo-YouTube-FF0000?style=flat&logo=youtube)](https://www.youtube.com/watch?v=S-6IQegoF5s)

> **Reduction of Energy Usage and Effective Area by Statistical Precision Scaling and Analog Layer Cascading in Analog Compute-in-Memory**

**Team:** Siddarth Gupta · Varunan Balasubramanian · Lalitya Marathe · Ananya Dongsarwar  
**Supervisor:** Prof. Arun Tej Mallajosyula  
**Institution:** IIT Guwahati

---

## 🎯 The Problem

In Analog Compute-in-Memory (CIM) architectures, crossbar arrays perform multiply-accumulate (MAC) operations directly in hardware using Ohm's law:

$$I_j = \sum_i G_{ij} V_i$$

This eliminates digital multipliers and drastically cuts data movement energy. However, every layer output must be digitized before the next layer — and this ADC cost dominates everything:

$$E_{ADC} \gg E_{crossbar}$$
$$E_{ADC,\text{total}} = L \cdot E_{ADC}$$

**The more layers, the more ADC conversions. ADC energy scales with depth, not computation.**

---

## 💡 Our Solution

We propose two combined ideas that together halve ADC usage without sacrificing precision:

### 1. Statistical Conductance Splitting
Instead of storing each weight in one flash transistor, we split it across **n parallel devices**:

$$G_{eff} = \sum_{k=1}^{n} G_k, \quad \sigma_{eff} = \frac{\sigma_{single}}{\sqrt{n}}$$

For **4-way splitting** (n = 4):
- Standard deviation reduces by **2×**
- Noise power reduces by **4×**
- SNR improves by **6 dB** ≈ **one additional effective bit of precision**

### 2. Analog Layer Cascading
Using the precision margin gained from splitting, we cascade **two layers entirely in the analog domain** before any digitization — eliminating the ADC/DAC pair between them:

$$E_{ADC,\text{total,new}} = \frac{L}{2} \cdot E_{ADC}$$

### 3. Analog ReLU in Current Domain
To cascade layers without going digital, the activation function must stay analog. We implement ReLU using:
- Op-amp current buffers for isolation
- NMOS current mirror for differential subtraction: $I_{node} = I^+ - I^-$
- A selective mirror stage that passes only positive currents:

$$I_{out} = \begin{cases} I^+ - I^-, & I^+ > I^- \\ 0, & I^+ \leq I^- \end{cases}$$

---

## 📐 Architecture

```
Input Voltages
      │
      ▼
┌─────────────┐
│  MAC Layer 1│  ← Conductance crossbar (4-way split weights)
│  (Crossbar) │
└──────┬──────┘
       │ Analog current (no ADC here!)
       ▼
┌─────────────┐
│  Analog     │  ← Current subtraction + current-mirror ReLU
│  ReLU       │
└──────┬──────┘
       │ Analog current
       ▼
┌─────────────┐
│  MAC Layer 2│  ← Second crossbar layer
│  (Crossbar) │
└──────┬──────┘
       │
       ▼
     [ADC]       ← Only ONE ADC for every TWO layers
       │
       ▼
    Digital Output
```

---

## 🎥 Demo & Simulation

Our simulation demonstrates a fully integrated neural network using a differential MAC architecture with dual memristor crossbars, an analog ReLU stage, and statistical conductance splitting — all validated in LTSpice.

[![Watch Demo](https://img.shields.io/badge/▶_Watch_Simulation_Demo-YouTube-FF0000?style=for-the-badge&logo=youtube)](https://www.youtube.com/watch?v=S-6IQegoF5s)

> Click the badge above or go to: https://www.youtube.com/watch?v=S-6IQegoF5s

---

## 📊 Results

### Power Reduction (3×3 Differential MAC)

| Architecture | Power |
|---|---|
| Conventional (with ADCs) | 240 µW (3 × 80 µW ADCs) |
| Proposed (ADC-free cascade) | 112.134 µW |
| **Reduction** | **127.866 µW → 53.3% lower** |

Since ADCs account for ~85% of power in a conventional analog CIM chip, this translates to approximately **~40% reduction in overall chip power**.

### Statistical Validation

Monte Carlo simulation of the 4-way conductance splitting yielded a measured standard deviation of **3.1%** relative to the nominal programmed conductance — confirming the theoretical √n reduction in variability and validating that cascading two layers stays within the original single-layer noise budget.

---

## 🔬 Implementation Details

| Parameter | Value |
|---|---|
| Technology | 180 nm CMOS |
| Design domain | Fully current-mode (no unnecessary voltage conversions) |
| Simulation tool | LTSpice |
| Test case | 3×3 differential MAC, input vector [3mV, 5mV, 7mV] |
| Validation | Monte Carlo runs for statistical error analysis |
| Conductance splitting | 4-way parallel devices per weight |
| ADC reduction | 50% (1 ADC per 2 layers vs 1 per layer) |

---

## 📁 Repository Structure

```
├── ltspice/
│   └── final_submission_schematic.asc   # LTSpice schematic (full design)
├── report/
│   └── 2526_C_0061-Memery.pdf           # Full competition report
└── README.md
```

---

## 📋 Report

The full competition report documents the theoretical analysis, circuit design, simulation methodology, and results in detail.

[![Report](https://img.shields.io/badge/📄_Full_Report-PDF-red?style=for-the-badge)](report/2526_C_0061-Memery.pdf)

| Field | Details |
|---|---|
| **Team ID** | 2526-C-0061 |
| **Competition** | Micron Memory Awards 2025 |
| **Category** | Device Design & Development |
| **File** | [`report/2526_C_0061-Memery.pdf`](report/2526_C_0061-Memery.pdf) |

> 📌 The report covers: problem formulation, conductance splitting theory, analog ReLU circuit design, LTSpice simulation setup, Monte Carlo validation, and power analysis.

---

## 🔭 Future Work

- **Bit slicing** for higher precision weight representation
- **Technology scaling** to 65 nm / 28 nm for lower parasitics and area
- **Op-amp elimination** using purely transistor-level current steering
- **Weight pruning & sparsity** to reduce crossbar size and ADC calls further
- **Deeper analog cascading** — generalizing to m-layer chains
- **On-chip learning** using conductance update rules for in-memory training

---

## 📄 References

Key references this work builds on:
- Sebastian et al., *Nature Electronics* 2020 — Analog In-Memory Computing
- Shafiee et al., *ISCA 2016* — ISAAC crossbar accelerator
- Chi et al., *ISCA 2016* — PRIME ReRAM architecture
- Han et al., *NeurIPS 2015* — Pruning and sparsity in neural networks

---

## 🔗 Links

- 📹 **Demo Video:** [YouTube — Simulation Results](https://www.youtube.com/watch?v=S-6IQegoF5s)
- 💻 **GitHub Repository:** [SyntaxSid/Analog-Compute-In-Memory-MAC-Accelerator](https://github.com/SyntaxSid/Analog-Compute-In-Memory-MAC-Accelerator)
- 📋 **Full Report:** [`report/2526_C_0061-Memery.pdf`](report/2526_C_0061-Memery.pdf)
- 🏆 **Competition:** [Micron Memory Awards 2025](https://www.micron-mimoryawards.org.tw/en/)

---

<p align="center">Made with ⚡ at IIT Guwahati · Micron Memory Awards 2025</p>