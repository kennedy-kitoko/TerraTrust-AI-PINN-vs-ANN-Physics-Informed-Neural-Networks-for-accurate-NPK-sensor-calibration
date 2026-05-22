# TerraTrust AI — PINN vs ANN: Physics-Informed Neural Networks for accurate NPK sensor calibration in Smart Agriculture

> **A new way to calibrate low-cost NPK soil sensors.** Combining Physics-Informed Neural Networks with Bayesian uncertainty quantification to turn a $50 soil probe into a continuous, drift-aware NPK estimation tool for African smallholder agriculture.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Project Status](https://img.shields.io/badge/Status-Class%20Project-blue.svg)]()
[![Languages](https://img.shields.io/badge/Languages-FR%20%7C%20EN%20%7C%20中文-orange.svg)]()
[![YouTube](https://img.shields.io/badge/YouTube-Watch%20demo-red.svg)](https://www.youtube.com/watch?v=KE-JiOSihdw)

---

## 📺 5-minute video walkthrough

**▶️ [Watch on YouTube](https://www.youtube.com/watch?v=KE-JiOSihdw)**

A complete walkthrough of the problem, the proposed solution, the simulation results (honest findings included), and the real-world application.

---

## 🌾 What this project solves

Off-the-shelf NPK soil sensors (VEMSEE SN-3002, Rainbow All-in-One, generic ModBus probes priced around $50) display nitrogen, phosphorus, and potassium values derived from a **generic empirical formula — identical for every soil type on Earth**. The readings are often off by a factor of two or more, with no mechanism to detect when soil chemistry has drifted between calibrations.

For smallholder farmers in Africa, the alternatives are out of reach:

- **Laboratory analysis**: $30–80 per measurement, 1–2 week delay — unaffordable for continuous monitoring
- **Memory-based sensors (VEMSEE v2)**: lab values can be written into sensor memory, but they freeze while soil keeps evolving
- **Standard ANN calibration**: improves accuracy but ignores soil physics and gives no warning when conditions change

Between two lab analyses, the farmer flies blind. The result is over-fertilization (root burn, nitrate leaching, wasted money) or under-fertilization (yellow leaves, poor yields).

## 🧠 The proposed solution

**TerraTrust AI** is a Physics-Informed Neural Network (PINN) combined with Bayesian uncertainty quantification, deployed on a $30 ESP32-S3 microcontroller. The architecture:

1. **PINN with Kohlrausch's law as physical constraint** — the network is trained to respect `EC = λ_N·[N] + λ_P·[P] + λ_K·[K] + EC_background`, which removes entire classes of physically impossible NPK predictions a standard ANN would generate.

2. **Monte Carlo Dropout for drift detection** — 10 forward passes with active dropout estimate prediction variance. When soil chemistry leaves the training distribution, variance rises **28 days before** predictions become dangerously wrong (>20 mg/kg error in our simulation).

3. **Edge-first deployment** — quantized to int8, the full model fits in ~5 KB on an ESP32-S3. Inference: 3 ms. Full uncertainty quantification: 30 ms. Solar-powered, IP67, works completely offline. Cloud is optional and used only for monthly retraining when new lab analyses arrive.

**Total hardware cost: ~$120. ROI: one growing season.**

## 📊 Honest simulation findings

Two simulations were run with full methodological transparency. Both showed real benefits AND real limitations.

### Simulation 1 — PINN vs ANN, head to head

800 synthetic samples with realistic NPK distributions, EC computed via Kohlrausch + 7% sensor noise, three data regimes tested (5%, 20%, 50% lab data).

| Finding | Result |
|---------|--------|
| ✅ **Phosphorus (weakest physical signal)** | PINN beats ANN by **+8 R² points** at 5% lab data (0.37 vs 0.29) |
| ⚖️ **Nitrogen and potassium (strong signal)** | Both models tie at R² ≈ 0.78 even with just 30 lab measurements |
| ✅ **Physical consistency** | Kohlrausch violation reduced by 13% in mid-regime |
| ❌ **"ANN collapses without data"** narrative | **FALSE** — ANN performs well on N and K even at 5% lab data |

**Conclusion:** PINN advantage is real but specific — it helps most where data signal is weakest. The marketing claim that "PINN rescues a collapsing ANN" is exaggerated.

### Simulation 2 — Soil drift detection over 180 days

PINN trained on first 30 days only. Then continuous prediction over 150 more days while soil drifts (rain leaches N, fertilizer added day 90, plants absorb K from day 120).

| Mechanism | First alert | Verdict |
|-----------|-------------|---------|
| **Kohlrausch residual** (physics-only) | Day 100 | ❌ **22 days too late** — under-determined problem |
| **Bayesian uncertainty** (MC Dropout) | Day 49 | ✅ **28 days early warning** before dangerous error (day 77) |

**Key technical insight:** Kohlrausch's law alone CANNOT detect drift, because 1 equation with 3 unknowns is mathematically under-determined (infinite NPK triplets satisfy the same EC). The PINN-BNN architecture (Khan et al., IEEE OJ-CS 2025) — combining physics constraint AND Bayesian inference — is mathematically necessary, not just convenient.

## 🛠 Architecture: Edge + Opportunistic Cloud

```
┌─────────────────────────────────────────────────┐
│      ESP32-S3 (autonomous, no internet)         │
│                                                 │
│  VEMSEE sensor → PINN inference (3 ms)          │
│                → Kohlrausch consistency audit   │
│                → MC Dropout x10 (30 ms)         │
│                → Local alerts (LED/OLED/BLE)    │
│                                                 │
│  Storage: 30 days flash buffer (offline mode)   │
└──────────────┬──────────────────────────────────┘
               │ LoRa / WiFi (opportunistic)
               │ ~few KB/day if connected
┌──────────────▼──────────────────────────────────┐
│            Cloud (optional)                     │
│                                                 │
│  • Monthly PINN retraining (~30 new lab pts)   │
│  • OTA model delivery (~10 KB quantized)        │
│  • Extended MC Dropout validation (100 passes)  │
│  • Federated multi-farm learning                │
│  • SMS alerts when farmer is far from box       │
└─────────────────────────────────────────────────┘
```

## 📚 Scientific validation — 3 peer-reviewed publications (2025)

This project doesn't reinvent the wheel — it transposes a scientifically validated methodology to the African agriculture context.

1. **Rizqan, Hole, Gretton (2025)** — *Evaluation and Verification of Physics-Informed Neural Models of the Grad-Shafranov Equation* — Australian National University. Validates that PINNs generalize well in-domain (R²≈0.96) but drift in extrapolation (R²≈0.17) → justifies our recalibration alert mechanism.

2. **Khan, Rahman, Mahmud et al. (2025)** — *A Physics-Guided Bayesian Neural Network for Sensor Fault Detection in Wind Turbines* — IEEE Open Journal of the Computer Society. **Main architectural reference.** Demonstrates PINN-BNN achieves 97.6% accuracy on real wind turbine sensor data. Combining physics-informed learning + Bayesian inference improves generalization on sparse and noisy data.

3. **Venianakis, Theodoropoulos, Kavousanakis (2025)** — *A Physics Informed Machine Learning Framework for Optimal Sensor Placement and Parameter Estimation* — Elsevier. Shows PINNs can estimate unknown physical parameters from sparse point sensors with Gaussian noise robustness.

**Agricultural domain reference:** Ennaji et al. (2025), *Machine learning-based optimization of site-specific NPK fertilizer recommendation*, Smart Agricultural Technology (Elsevier/IEEE Xplore). Closest study to NPK agriculture — demonstrates R²=0.96 in interpolation and simulated +20% yield gain under environmental constraints.

## ⚠️ Honest limitations

Scientific honesty requires stating what this project does NOT prove:

- **Mathematical under-determination:** 1 EC measurement vs 3 NPK unknowns. Future work should use ion-selective electrodes (ISE) or multi-frequency impedance to fully resolve the inverse problem.
- **Modest precision:** ~±12 mg/kg typical error on N and K, more on P. Useful for coarse fertilization decisions ("add fertilizer or not"), not for millimeter-precision agriculture.
- **No field validation yet:** all simulations are synthetic. Field validation on real African soils is the next step, not yet performed.
- **Yield figures from literature:** the +20% yield gain comes from published simulations (Ennaji et al. 2025), not from TerraTrust AI measurements in field.

## 📂 Repository structure

```
.
├── index.html                          ← Main project page (FR/EN/中文)
├── README.md                           ← This file
├── LICENSE                             ← MIT License
│
├── images/                             ← All visuals referenced in index.html
│   ├── 01_problem_overview.png         ← Section 01: The Problem infographic
│   ├── 02_existing_solutions_limits.png ← Section 02: Limits of existing solutions
│   ├── 03_pinn_solution_diagram.png    ← Section 03: PINN approach diagram
│   ├── 04_simulation_pinn_vs_ann.png   ← Section 04: Simulation graph (PINN vs ANN)
│   ├── 05_drift_detection_180days.png  ← Section 05: Drift detection graph
│   ├── 06_terra_ai_box_in_field.png    ← Section 07: TERRA AI Box photo
│   └── 07_terra_ai_dashboard_interface.png ← Section 07: Dashboard + agent chat
│
├── simulations/                        ← Python source code for simulations
│   ├── simulation_pinn_vs_ann.py       ← Static benchmark on synthetic data
│   ├── simulation_drift_detection.py   ← 180-day drift detection
│   └── requirements.txt                ← torch, numpy, matplotlib, scikit-learn
│
├── docs/                               ← Detailed reports
│   ├── honest_simulation_report.md     ← Full PINN vs ANN findings
│   └── drift_detection_report.md       ← Drift detection technical analysis
│
└── presentation/                       ← Class deliverables
    ├── TerraTrust_AI_Final_5min.pptx   ← Final 5-minute presentation
    └── oral_script_5min.md             ← Time-stamped speaking notes
```

## 🖼️ Required image files

The `index.html` page references **7 image files** (located in `images/` folder). If you have existing visuals, **rename and convert them to PNG** with the following exact names:

| File name | Content | Where it appears |
|-----------|---------|------------------|
| `01_problem_overview.png` | Infographic showing under/over-fertilization problem, African farmer context | Section 01 — The Problem |
| `02_existing_solutions_limits.png` | Infographic comparing lab analysis, VEMSEE sensor, ANN, etc. | Section 02 — Existing solutions |
| `03_pinn_solution_diagram.png` | Diagram of the PINN architecture with Kohlrausch's law as constraint | Section 03 — Our approach |
| `04_simulation_pinn_vs_ann.png` | Multi-panel graph from `simulation_pinn_vs_ann.py` — R² bars, scatter plots, MAE | Section 04 — Honest simulation |
| `05_drift_detection_180days.png` | 5-panel temporal graph from `simulation_drift_detection.py` — NPK prediction, Kohlrausch residual, Bayesian uncertainty, real error, synthesis | Section 05 — Drift detection |
| `06_terra_ai_box_in_field.png` | Photo or infographic of the TERRA AI Box deployed in a corn field, with farmer using smartphone | Section 07 — Products |
| `07_terra_ai_dashboard_interface.png` | Screenshot of Terra AI v3.0 web dashboard (sensor readouts + agent chat) | Section 07 — Products |

**Image format requirements:**
- Format: **PNG** (recommended) or JPG
- Aspect ratio: landscape preferred (16:9 or 3:2)
- Min resolution: 1200px wide
- Max file size: 500 KB each (compress with [TinyPNG](https://tinypng.com) if needed)
- The HTML applies `width: 100%; border-radius: 10px; box-shadow` automatically

## 🚀 Quick start

### View the project page locally

```bash
git clone https://github.com/<your-username>/terratrust-ai.git
cd terratrust-ai
# Just open index.html in your browser, or serve locally:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

The page is fully self-contained — only inline CSS and JavaScript, no external dependencies except Google Fonts (loaded over CDN).

### Deploy on GitHub Pages

1. Push this repository to GitHub
2. Go to `Settings → Pages`
3. Select branch `main` and folder `/ (root)`
4. Your site will be live at `https://<your-username>.github.io/terratrust-ai/`

### Run the simulations

```bash
cd simulations/
pip install -r requirements.txt
python simulation_pinn_vs_ann.py       # ~5 minutes runtime
python simulation_drift_detection.py   # ~3 minutes runtime
```

## 🌐 Multilingual support

The page is available in **three languages** via the floating language switcher in the top-right corner:

- 🇫🇷 **Français** (default)
- 🇬🇧 **English**
- 🇨🇳 **中文 (Simplified Chinese)**

Language switching is instant (no page reload). All content blocks (sections, callouts, references) are translated.

## 🔗 Related resources

- **Sensor hardware:** [Rainbow All-in-One Soil Sensor NPK pH EC](https://fr.made-in-china.com/co_rainbowsensor/product_All-in-One-Soil-Sensor-NPK-pH-Ec-Sensor-for-Agriculture-Monitoring_uoynuyhrey.html) (~$50)
- **PINN theory:** [MathWorks — Physics-Informed Neural Networks](https://fr.mathworks.com/discovery/physics-informed-neural-networks.html)
- **Video walkthrough:** [YouTube — 5-min presentation](https://www.youtube.com/watch?v=KE-JiOSihdw)

## 👤 Author

**Kennedy Kitoko Muyunga**

🎓 3rd-year student, Bachelor of Mechanical & Electrical Engineering
📍 Beijing Institute of Technology (BIT)
📅 Born: July 24, 2003

**Contact:**
- 𝕏 (Twitter): [@kennedykitoko13](https://x.com/kennedykitoko13)
- YouTube: [TerraTrust AI demo](https://www.youtube.com/watch?v=KE-JiOSihdw)

Technical discussions, collaboration proposals, and constructive feedback are all welcome.

## 🤝 Contributing

This is a class project, but contributions and discussions are welcome — especially from:

- Researchers in PINN / Physics-Informed ML applications
- AgriTech engineers working on low-cost sensor calibration
- Embedded ML / Edge AI specialists with ESP32 experience
- African agricultural research institutions interested in field validation

Open an issue or reach out on X.

## 📄 License

This project is released under the [MIT License](LICENSE) — code, simulations, and documentation can be freely reused with attribution. Hardware specs and methodology are intended to be open and reproducible.

## 🙏 Acknowledgments

- Beijing Institute of Technology — AI & Simulation Science course (Spring 2026)
- The three IEEE/Elsevier 2025 publications cited above for methodological grounding
- The open-source PyTorch and PptxGenJS communities

---

*"TerraTrust AI doesn't replace the laboratory. It tells you when to go back."*
