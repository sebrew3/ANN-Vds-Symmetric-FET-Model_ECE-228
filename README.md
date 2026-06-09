# ANN-Vds-Symmetric-FET-Model_ECE-228
# ANN-Based Drain-Source Symmetric FET Model
## Reproduction + Improvement: Multi-Gate-Length Extension

**Based on:** Xu et al., *"Drain-Source Symmetric Artificial Neural Network-Based FET Model with Robust Extrapolation Beyond Training Data"*, IEEE MTT-S 2007.

---

### README:
#### Overview

This notebook reproduces and extends the symmetric ANN-based FET model from Xu et al. (2007). The model learns to predict drain current Id from gate and drain voltages (Vgs, Vds) using a shared-weight neural network architecture that enforces exact drain-source symmetry by construction — meaning it accurately predicts Id-Vds behavior in both the positive and negative Vds regions while being trained only on Vds ≥ 0 data.

The main features of this notebook are as follows:

1. **Processes** Id-Vds data from the public MESD-MOSFET-Electrical-Simulation-Dataset (https://github.com/SJTU-YONGFU-RESEARCH-GRP/MESD-MOSFET-Electrical-Simulation-Dataset)
2. **Reproduces** Xu et al's symmetric ANN architecture using symmetric and anti-symmetric neural network formulations with applied symmetry constraints (Eq. 4-5 in the paper)
3. **Improves** the model by adding **multiple gate lengths (Lg)** as additional inputs; physics-based penalties that account for nanoscale short-channel effects are implemented in the training loop to capture how device scaling affects Id-Vds characteristics
4. **Validates** accurate symmetric device performance via Id-Vds family-of-curves plots, symmetry checks, and error analyses

#### Key Results

| | Paper (Xu et al. 2007) | This Notebook |
|---|---|---|
| **Architecture** | Symmetric + anti-symmetric ANN (MLP3/MLP4) | ✅ Symmetric ANN (tanh MLP, shared weights) |
| **Symmetry enforcement** | By construction via Eq. 4-5 | ✅ `ds_exchange()` + weight sharing |
| **Training data** | Measured GaAs pHEMT (Vds ≥ 0) | ✅ MESD BSIM-simulated MOSFET data (Vds ≥ 0 only) |
| **Symmetry error** | Guaranteed by construction | ✅ < 1e-5 mA (machine precision) |
| **Gate lengths** | Single (0.25µm) | 🆕 Multiple gate lengths (continuous Lg input) |
| **Saturation shaping** | Not discussed | 🆕 Physics augmentation + weighted loss |

#### Key Takeaways
1. **Symmetry** is guaranteed mathematically by the shared-weight ANN decomposition — the error is at floating-point precision (~1e-6), with no need for explicit symmetry constraints in the data.
2. **Gate-length extension** demonstrates that the ANN framework naturally accommodates an additional physical parameter as a third input, enabling a unified model across a device family — a practical improvement over single-device models.
3. **Physics-based augmentation and loss shaping** are effective tools for improving the fidelity of the mirrored negative-Vds curves without violating the paper's core constraint of training only on Vds ≥ 0.

#### Steps to Run Code and Reproduce Main Results

1. **Run Section 0** (Imports & Setup) to install and import all dependencies.
2. **Run Section 1** (Load MESD Dataset from GitHub) to download and unzip the MESD dataset directly from GitHub. Under the **Configuration** block in this section, you can customize:
   - `DEVICE_TYPE`: `'NMOS'` or `'PMOS'`
   - `CORNER`: process corner (default `'tt'` for typical-typical)
   - `TEMP`: measurement temperature in °C (default `-40`)
   - `TARGET_PDK`: which PDK to use for training (default `'N180A'` -- has the closest gate length to the 250nm used in the paper)
3. **Run Section 2** (Symmetric ANN Architecture + Training) to preprocess and augment the data, define the model, and train both the single-Lg and multi-Lg models. Training hyperparameters (epochs, learning rate, batch size) can be adjusted in the `train_model_weighted()` call at the bottom of this section.
4. **Run Section 3** (Symmetry Verification) to confirm that the model satisfies `Id(V) + Id(V-hat) = 0` to machine precision across the full voltage space.
5. **Run Section 4** (Id-Vds Curves) to reproduce the paper's Fig. 2 — the family of Id-Vds curves across all Vds (including negative), with and without the extrapolation routine.
6. **Run Section 5** (Multi-Gate-Length Curves) to view the improvement: Id-Vds curves predicted by the multi-Lg model across all gate lengths in the selected PDK.
7. **Run Section 6** (Quantitative Error Analysis) to compute MAE and R² on the held-out test set and view parity plots for both the single-Lg and multi-Lg models.
