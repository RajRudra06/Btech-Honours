# CALUDE_Summary.md - BTech Honours Research Progress

## Project Title
**Geometric Perception Under Photometric Stress: How Architectural Choices in Surface Normal Estimation Respond to Structured Lighting and Colour Degradations**

---

## 1. Research Framework
*   **Domain:** Computer Vision -> 3D Low-Level Vision -> Surface Normal Estimation (SNE).
*   **Core Question:** How do specific architectural components mediate a model's ability to separate **geometry** (fixed orientation) from **appearance** (changing lighting)?
*   **Study Type:** Field-driven, understanding-first (Path B).
*   **Dataset:** NYUv2 (Silberman et al. 2012).
*   **Ground Truth:** Ladicky et al. 2014 (Precomputed normals).
*   **Evaluation Protocol:** Wang et al. 2015 (654-image test split, MAE, Delta thresholds).

---

## 2. Experimental Variables (The Axes)
The study investigates three primary architectural axes in CNN-based encoder-decoder models:

1.  **Encoder Depth:** Testing if deeper abstraction provides better photometric invariance (ResNet-50 vs 101 vs 152).
2.  **Skip Connections:** Testing if low-level spatial shortcuts reintroduce photometric sensitivity (Full vs Partial vs None).
3.  **Normalization Layers:** Testing the hypothesis that Instance/Group Norm provides better invariance than Batch Norm.
4.  **Receptive Field (Secondary):** Impact of global context (ASPP) on planar vs. boundary robustness.

---

## 3. The Photometric Stress Taxonomy
Models are trained on clean images and evaluated (inference-only) on four degraded conditions:
1.  **Under-exposure** (0.3x multiplier).
2.  **Over-exposure** (2.5x multiplier + clipping).
3.  **Shadows** (Localized gradient darkening).
4.  **Day-Night/Color Shift** (Channel-wise temperature shifts).

*Note: Blur, noise, and resolution loss are excluded to keep the focus strictly on appearance vs. geometry.*

---

## 4. Current Progress & Workspace
*   **Base Directory:** `/content/drive/MyDrive/Btech Honours/First_try/surface_normal_project` (BASE).
*   **Data Status:**
    *   `nyu_depth_v2_labeled.mat` (2.9GB) downloaded and verified.
    *   Ladicky GT normals (PNGs) downloaded via AWS S3 and extracted.
*   **Visual Validation:** Step 1 complete. Visualized RGB, Depth, and established the quality gap between Simple Gradient (Cell 8) and GT (Cell 10).
*   **Literature Check:** Verified that Wang et al. (2015) metrics are still used in 2024 SOTA papers (e.g., Metric3D v2).

---

## 5. Implementation Roadmap
1.  **Step 1 (DONE):** Dataset loading and GT visualization.
2.  **Step 2 (PENDING):** Run a pretrained CNN-based SNE model on a clean sample and compute baseline MAE.
3.  **Step 3 (PENDING):** Apply photometric stress to the sample and measure prediction drift.
4.  **Phase 1 Execution:** Build the PyTorch Dataset/DataLoader and train a baseline ResNet-50 + UNet model.
5.  **Phase 2 Execution:** Perform surgical ablations (swapping norm layers, changing skip connections) and retrain/evaluate.

---

## 6. Key Citations to Maintain
*   **RGB+Depth:** Silberman, Nathan, et al. "Indoor segmentation and support inference from rgbd images." *ECCV 2012*.
*   **Normals:** Ladicky, Lubor, et al. "Discriminative multi-class segmentation of volumetric data." *CVPR 2014*.
*   **Protocol:** Wang, Xiaolong, et al. "Designing deep networks for surface normal estimation." *CVPR 2015*.
*   **Modern Validation:** Hu, Mu, et al. "Metric3D v2: A Versatile Geometric Foundation Model for Zero-shot Metric Depth and Surface Normal Estimation." *arXiv 2024*.
