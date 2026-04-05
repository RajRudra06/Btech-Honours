# Demo Strategy: Geometric Perception Under Photometric Stress
**Date:** Tuesday, 31 March 2026
**Researcher:** Rudra Rajpurohit
**Status:** Phase 1 - Preliminary Failure Analysis for Professor Demo

---

## 1. Core Objective
To demonstrate that **State-of-the-Art (SOTA)** Surface Normal Estimation models are architecturally sensitive to 2D appearance changes. Specifically, we aim to prove that **Skip Connections** (a fundamental design choice in U-Net architectures) erroneously propagate photometric noise (lighting) into geometric predictions (3D orientation).

---

## 2. The Experimental Subject (The Model)
*   **Model:** **Bae et al. (CVPR 2021)** - *Estimating Spatially-Varying Uncertainty for Shape-from-X.*
*   **Architecture DNA:** 
    *   **Encoder:** ResNet-101 (Pretrained on ImageNet).
    *   **Decoder:** U-Net style with **heavy Skip Connections**.
    *   **Task:** Dense per-pixel Surface Normal Estimation.
*   **Why this model?** It represents the industry standard for CNN-based geometric prediction. Its reliance on skip connections to preserve spatial detail makes it the perfect "victim" for testing photometric "leakage."

---

## 3. The Test Axis: Skip Connections
*   **Hypothesis:** Skip connections copy low-level features (RGB gradients) from the encoder directly to the decoder. While this helps reconstruct sharp corners in clean images, it causes the model to "hallucinate" 3D geometry when it encounters high-contrast 2D lighting features (like shadows).
*   **Isolation Method:** We will use the pretrained model as the "Sensitive Baseline" and contrast its failures against the Ground Truth (GT) and (eventually) an ablated "No-Skip" version.

---

## RESNET?

## Comprehensive ResNet Theory (Study Note / Master Prompt)

---

### 1) Why ResNet was invented (historical problem)

Before **ResNet (2015)**:
*   **Problem 1: Vanishing / Exploding Gradients:** Backpropagation multiplies gradients across layers. In deep networks, gradients → very small (vanish) or very large (explode), causing early layers to stop learning.
*   **Problem 2: Degradation Problem (CRITICAL):** Increasing depth **increased training error**. For example, a 20-layer CNN would perform better than a 56-layer CNN. This is NOT overfitting; it means optimization becomes harder.

**Key insight:** A deeper network should at least learn **identity mapping**, but standard deep CNNs couldn’t.

---

### 2) Core idea of ResNet (Residual Learning)

Instead of learning \( H(x) \), ResNet learns the residual \( F(x) = H(x) - x \). So, \( H(x) = F(x) + x \).

*   **Interpretation:** Learn the **residual (difference)** instead of the full transformation, which makes optimization easier.
*   **Residual block equation:** \( y = F(x) + x \), where \( x \) is the input and \( F(x) \) is the stacked conv layers.

---

### 3) Why residual connections work

*   **(A) Gradient flow improvement:** Even if \( \frac{dF}{dx} \) is small, the gradient still flows via the **identity path**.
*   **(B) Solves degradation problem:** If a deeper layer is useless, \( F(x) = 0 \Rightarrow y = x \). Identity mapping is easy, so the network won’t get worse with depth.

---

### 4) Types of residual connections

1.  **Identity shortcut:** \( y = F(x) + x \). Same dimensions, no extra parameters.
2.  **Projection shortcut:** \( y = F(x) + W_s x \). Uses 1×1 convolution to match dimensions when channels change or downsampling occurs.

---

### 5) Structure of ResNet

ResNet is organized into stages (Conv1, Conv2_x, Conv3_x, Conv4_x, Conv5_x). Each stage consists of multiple residual blocks, gradually reducing spatial size and increasing channels.

---

### 6) Basic Residual Block (Shallow models: ResNet-18, 34)
Structure: `Conv → BN → ReLU` followed by `Conv → BN` -> `Add input` -> `ReLU`.

---

### 7) Bottleneck Block (Deep models: ResNet-50+)
Structure: `1×1 → reduce channels`, `3×3 → process`, `1×1 → expand channels`.
*   **Why:** Reduce channels before expensive 3×3 conv (e.g., 256 → 64 → 64 → 256). Lower computation enables deeper networks.

---

### 8) Types of ResNet models
| Model | Depth     | Block Type |
| :--- | :--- | :--- |
| **18** | shallow   | basic      |
| **34** | shallow   | basic      |
| **50** | deep      | bottleneck |
| **101** | deeper    | bottleneck |
| **152** | very deep | bottleneck |

---

### 9) What ResNet actually learns
*   **Early layers:** Edges, Gradients, Textures.
*   **Mid layers:** Patterns, Parts of objects.
*   **Deep layers:** Object-level features, Semantic understanding.

---

### 10) Important distinction (for your research)
*   **ResNet skips (SHORT skips):** Inside blocks, local, feature refinement. Do NOT directly pass raw image features.
*   **U-Net skips (LONG skips):** Encoder → Decoder, pass high-resolution features. Carry edges, gradients, and photometric signals.
👉 **Your research focuses on U-Net skips, not ResNet skips.**

---

### 11) Why ResNet is used as encoder
*   Strong feature extraction and stable training.
*   Pretrained on ImageNet.
*   Provides multi-scale features.

---

### 12) Key intuition (final)
ResNet allows networks to: **“Learn corrections instead of full transformations.”**

---

### 13) Must-read papers
*   **Deep Residual Learning for Image Recognition** (Core paper)
*   **Identity Mappings in Deep Residual Networks** (Explains why identity skip works)
*   **DenseNet paper** (Extends the idea of skip connections)

---

## The Evolution of Deep Vision: From "What" to "Where Exactly"

To understand why we are testing **Skip Connections**, we must look at the linear evolution of how AI "sees" and "paints" the world.

---

### Phase 1: The Observer (Image Classification)
*   **The Task:** "Is this a room or a forest?"
*   **The Model:** ResNet, VGG (Encoder-Only).
*   **The Story:** The AI looks at a photo and "squints." It starts with high-resolution pixels and compresses them into a tiny 7x7 grid of "Meaning" (The Bottleneck). 
*   **The Result:** It identifies the object ("Room"), but it **destroys all spatial info**. It forgets *where* the wall was; it only knows a wall exists.

### Phase 2: The Painter (Basic Encoder-Decoder)
*   **The Task:** "Color every pixel that belongs to the wall."
*   **The Model:** FCN (Encoder + Decoder, No Skips).
*   **The Story:** To paint a map, the AI must "un-squint." It takes the 7x7 "Meaning" and stretches it back to 224x224. 
*   **The Result:** Because it "forgot" the exact edges during the squinting phase, the output is **blurry**. It guesses where the edges are, leading to "melted" geometry.

### Phase 3: The Architect (U-Net / Skip Connections)
*   **The Task:** "Draw a razor-sharp 3D map of the room."
*   **The Model:** U-Net, Bae et al. (Encoder + Decoder + **Skips**).
*   **The Story:** We realized the Encoder *did* see the sharp edges at the start. So, we built a **Bridge (Skip Connection)**. While the Decoder is painting, it reaches back to the Encoder's early notes to "trace" the sharp RGB lines.
*   **The Result:** **Razor-Sharp** boundaries. The AI can now align 3D normals perfectly with 2D edges.

---

### Phase 4: Our Research (The "Cheating" Problem)
*   **The Task:** Surface Normal Estimation under **Photometric Stress**.
*   **The Conflict:** We are the first to ask: **"Is this Bridge (Skip Connection) a vulnerability?"**
*   **The Hypothesis:** If a **shadow** creates a sharp line on a flat wall, the Encoder "traces" it via the Skip Connection. The Decoder, trusting the "trace" too much, paints a **fake 3D corner** where there is only a shadow.
*   **The Goal:** To prove that while Skips provide **Accuracy (Sharpness)**, they destroy **Robustness (Invariance)** by forcing the model to "cheat" using lighting-sensitive RGB edges.

---

## Skip and its meaning and AIM of experiment is ?

### The Story of Our Research: A Tale of Two Drawings

**Act 1: The Basic Task (The 3D Map)**
Imagine you are looking at a 2D photograph of a room. Your job is to draw a 3D map of that room, showing which way every surface is facing (the **Surface Normals**).

*   **The AI's Brain (The Bottleneck):** To do this, the AI first "squints" at the photo, compressing it down into its memory. It understands the "big picture"—that there is a wall on the left and a floor on the bottom. This is the **Bottleneck**.

---

**Act 2: The Innovation (The "Tracing Paper" Analogy)**
In a standard AI, once it squints and remembers the scene, it has to draw the 3D map from memory. 
*   **The "No-Skip" Result:** Because the AI is drawing from memory, it forgets exactly where the edges are. The 3D map is **blurry** and "melted." It’s a good guess, but it's not precise.

To fix this, we invented **Skip Connections**. 
*   **The "Skip" Result:** Think of skip connections as a piece of **Tracing Paper** placed over the original photo. While the AI is drawing the 3D map, it can "look through" the tracing paper to see the sharp lines of the original photo. It can now "snap" its 3D edges perfectly to the lines in the photo.
*   **The Outcome:** The 3D map becomes **pixel-perfect and sharp**. This is why SOTA models like **Bae et al. (2021)** use heavy skip connections.

---

**Act 3: The "Cheating" Problem (Early vs. Deep Skips)**
Here is the catch: Not all "Tracing Paper" is the same. Skip connections carry different types of information depending on where they come from:

1.  **Early Skip Connections (The "Pixel" Signal):** These are like looking through a camera sensor. They see raw pixels, sharp brightness changes, and high contrast. They carry **Edges and Gradients**.
2.  **Deep Skip Connections (The "Shape" Signal):** These are like looking at a rough sketch. They see object shapes and room layouts. They carry **Abstract Structure**.

**The Hypothesis:** In a clean world, the AI uses **Early Skip Connections** to "cheat." It sees a sharp line in the RGB photo and assumes, *"That line must be a 3D corner!"* It traces it, and usually, it's right. This is **High Accuracy**.

---

**Act 4: The Crisis (Photometric Stress)**
Now, we introduce **Photometric Stress**—like a sharp **Shadow** on a flat wall. 

*   **The Failure:** The AI looks through its **Early Skip Connection** tracing paper and sees a sharp line (the shadow). It doesn't know it's a shadow; it just sees a "line." 
*   **Geometric Hallucination:** Because the AI has been trained to "trust the tracing paper" for sharpness, it traces the shadow and predicts a **fake 3D corner** in the middle of a flat wall. 
*   **The No-Free-Lunch Theorem:** We have traded **Robustness** (the ability to ignore shadows) for **Accuracy** (pixel-sharpness). The AI is now "accurate" but "fragile."

---

**Act 5: The Diagnostic Experiment (What we are doing)**
We are going to be the **Inspectors** who prove exactly which "Tracing Paper" is causing the failure. We will compare four versions of the AI:

| Model | The "Axis" | The Strategy | The Prediction |
| :--- | :--- | :--- | :--- |
| **1. Bae et al. (SOTA)** | **Full Skips** | Use all tracing paper. | **Sharp but Fragile:** Will hallucinate geometry at every shadow edge. |
| **2. Vanilla U-Net** | **Standard Skips** | Symmetric baseline. | Moderate sensitivity. |
| **3. No-Skip Model** | **Memory Only** | No tracing paper. | **Blurry but Robust:** Will correctly see the wall as flat, ignoring the shadow. |
| **4. Deep-Skip Only** | **Selective Skips** | Only use "Shape" tracing paper. | **The Best of Both?** Might stay sharp but ignore the raw pixel-shadows. |

---

### The Final Takeaway (The Contribution)
We are trying to prove that **Early Skip Connections are the "Leak"** that lets lighting artifacts contaminate 3D geometry. By identifying this, we aren't just saying "skips are bad"—we are saying: 
> *"To build the next generation of 3D AI, we must learn to **filter** what we trace. We need the sharpness of early layers, but we must protect them from being fooled by the lighting."*

---

## Why/What Skip Connections

## Summary of the Discussion (Skip Connections & Surface Normal Estimation)

### 1) Core problem: why standard CNNs fail at precise geometry

A typical encoder–decoder CNN compresses an image into a **low-resolution bottleneck** and then reconstructs it.

* During **downsampling**, fine details like edges and pixel-level boundaries are lost
* The bottleneck mainly retains **semantic information** (“what is in the image”)
* When upsampling, the model must **guess spatial details**

**Result:**

* Outputs are **smooth and blurry**
* Edges are not aligned to exact pixel locations
* For surface normals, this leads to **incorrect orientation at boundaries**

---

### 2) What skip connections solve

Skip connections directly pass **high-resolution features from early layers** to later layers.

* Early layers contain:

  * Edges
  * Gradients
  * Fine textures
* These are **not recoverable once lost**

With skip connections:

* The decoder receives both:

  * **Global understanding (from bottleneck)**
  * **Precise spatial detail (from early layers)**

**Key idea:**
They combine **“what” + “where exactly”**

---

### 3) Why this is critical for surface normals

Surface normal estimation is a **dense prediction task** where each pixel represents a 3D direction.

* At object boundaries:

  * Normals change **abruptly**
  * Precision must be **pixel-level**

#### Without skip connections

* Normals transition smoothly across edges
* Geometry looks **rounded or melted**

#### With skip connections

* Normals change sharply at boundaries
* Geometry aligns correctly with RGB edges

**Conclusion:**
Skip connections are **essential**, not just helpful, for high-quality normal maps.

---

### 4) Technical formulation (important to remember)

Without skip connections:
[
\hat{y} = D(E(x))
]

With skip connections:
[
\hat{y} = D(z, E_1, E_2, ..., E_n)
]

At each decoder stage:
[
F_{dec}^{(i)} = \text{Up}(F_{dec}^{(i+1)}) \oplus E_i
]

Where:

* ( E_i ): encoder features (high resolution)
* ( \oplus ): concatenation or addition

**Meaning:**
The output depends on both:

* Compressed global features
* Multi-scale spatial details

---

### 5) Types of skip connections

#### Long skip connections

* Encoder → Decoder
* Preserve spatial detail
* Used in dense prediction tasks

#### Short skip connections

* Within layers
* Help gradient flow and training stability
* Do **not directly restore spatial detail**

---

### 6) Important architectures to remember

#### Most suitable (for surface normals)

* **U-Net variants**

  * Strong spatial preservation
  * Symmetric encoder–decoder with skips

* **U-Net + ResNet encoder**

  * Combines strong feature extraction + spatial recovery
  * Very commonly used in practice

* **HRNet**

  * Maintains high resolution throughout
  * Excellent for precise geometry tasks

---

#### Other relevant architectures

* **ResNet**

  * Uses short skip connections (residuals)

* **DenseNet**

  * Extensive feature reuse via dense connections

* **FPN**

  * Multi-scale feature fusion

* **SegFormer**

  * Modern transformer-based architecture

---

### 7) Final mental model (to remember easily)

Think of it like this:

* **Bottleneck only:**
  “I know there is an edge somewhere”

* **With skip connections:**
  “The edge is exactly at this pixel”

---

### Final takeaway

* Surface normal estimation requires **precise spatial alignment**
* Standard CNNs lose this due to compression
* Skip connections restore it by reusing early features

**Therefore:**
Using architectures like **U-Net variants, U-Net + ResNet encoder, and HRNet is the correct design choice** for accurate normal prediction.

---

## SKIP/NO-SKIP Logic

### Short answer
You’re mostly correct, but one key correction:
*   **No-skip model:** Usually **less accurate on clean data** (blurry, less sharp), but **more stable under photometric stress**.
*   **Skip model:** **more accurate on clean data**, but **larger error increase under stress**.

---

### 1) Correct mental model (fixing your statement)

#### No-skip model
*   Relies only on **deep features (semantics + geometry)**.
*   Does NOT use raw edge signals.
*   **On clean data:** Normals are smooth and slightly inaccurate at boundaries. Accuracy is **lower than skip model**.
*   **Under photometric stress:** Since it ignores raw edges, it is not strongly affected by shadows/exposure. Error increase is **small (stable)**.

#### Skip model (U-Net style)
*   Uses both deep features (geometry) and early features (edges).
*   **On clean data:** Edges align perfectly and normals are sharp. **High accuracy**.
*   **Under photometric stress:** Shadow edges are treated like real edges; the model “snaps” geometry to them. This produces fake corners and incorrect plane boundaries. Error increase is **large**.

---

### 2) What you said vs. Corrected version
*   **Your initial idea:** "No-skip might be very accurate but weird normals."
*   **Correct version:** "No-skip is actually **less accurate overall**, but **more robust**."

---

### 3) What actually happens visually

#### No-skip model
*   Wall remains wall under shadow.
*   **But:** Edges are blurry and normals are smooth.
*   👉 **“Safe but imprecise”**

#### Skip model
*   Very sharp normals on clean data.
*   **Under shadow:** Wall may look like it has a corner; geometry bends at shadow edges.
*   👉 **“Sharp but can hallucinate”**

---

### 4) What your experiment will likely show

| Model   | Clean Error | Stress Error | Δ (increase) |
| :--- | :--- | :--- | :--- |
| **No-skip** | Medium/High | Medium/High  | **Small** |
| **Skip**    | **Low**     | High         | **Large** |

---

### 5) The key insight you are reaching
This is the exact conclusion:
> Skip connections improve **accuracy**, but reduce **robustness to photometric changes**.

---

### 6) Why this happens (final clarity)

#### No-skip
*   Uses context and geometry; ignores raw intensity changes.
*   👉 Shadows don’t strongly affect it.

#### Skip model
*   Uses raw gradients (early skips).
*   👉 Shadows = strong gradients → misinterpreted as geometry.

---

### 7) Final one-line understanding
*   **No-skip:** “I’ll guess geometry from context” → stable but blurry.
*   **Skip:** “I’ll trust edges” → accurate but easily fooled.

---

### Final takeaway
No-skip model is **less accurate but robust**; Skip model is **more accurate but sensitive to photometric stress**. Your experiment is designed to **prove exactly this trade-off and identify its source (early skips)**.

---

## Experimental Protocol: The Accuracy-Robustness Trade-off

To rigorously test our hypothesis, we will conduct a comparative study across a spectrum of architectural "DNA." We are not simply looking for the most accurate model; we are investigating the **Relative Degradation** of each architecture under photometric stress.

### 1. The Three-Model Spectrum

We will evaluate three distinct architectural configurations to isolate the impact of skip connections:

| Model | Classification | Research Purpose | Expected Behavior |
| :--- | :--- | :--- | :--- |
| **1. Vanilla U-Net** | **The Baseline** | Establishes the standard behavior of a symmetric encoder-decoder with skip connections. | Balanced performance with moderate sensitivity to lighting edges. |
| **2. Bae et al. (SOTA)** | **The Performance Peak** | Represents the industry standard (ResNet-101 + Dense Skips). Shows how high capacity affects robustness. | **Highest Accuracy** on clean data; **Highest Sensitivity** (fragility) under stress. |
| **3. Non-Skip Model** | **The Robustness Control** | A "Scientific Placebo" (Pure Encoder-Decoder). Removes the spatial "copy-paste" mechanism. | **Lower Accuracy** on clean data (blurry); **Highest Robustness** (immune to lighting edges). |

---

### 2. The Evaluation Matrix (The "Stress Test")

All three models will be trained/evaluated on **Clean NYUv2 data** and then subjected to the **Top 5 Photometric Stresses** during inference. We will measure the **Mean Angular Error (MAE)** and the **% Performance Drop**.

| Model | Clean (MAE) | Stressed (MAE) | **% Performance Drop (Fragility)** |
| :--- | :--- | :--- | :--- |
| **Vanilla U-Net** | Baseline | + Error | Moderate % |
| **Bae et al. (SOTA)** | **Best (Lowest)** | +++ Error | **Highest % (Most Fragile)** |
| **Non-Skip Model** | Worst (Highest) | **Stable** | **Lowest % (Most Robust)** |

**The Research Goal:** To prove that while skip connections are necessary for pixel-perfect accuracy, they create a **direct vulnerability** to photometric noise.

---

### 3. Implementation Workflow (The "Surgical" Approach)

For the initial demo and proof-of-concept, we will use a **"Zero-Shot Ablation"** strategy:
1.  **Baseline Inference:** Load the pretrained **Bae et al. (2021)** model.
2.  **Ablated Inference:** Manually disconnect or `zero_out` the skip connection tensors in the model's `forward()` pass.
3.  **Comparison:** Run both versions on a **Shadow-Stressed** image to visualize the "Geometric Drift" vs. "Geometric Stability."

---

## Flow/Expectation of experiment

## Summary
Your pipeline should clearly separate **(1) training on clean data** and **(2) evaluation under photometric stress**, and report both **absolute accuracy** and **error increase (Δ)**. The goal is to show:
*   Skip models → better on clean
*   But degrade more under stress

---

### 1) Models you will train
Use exactly these 3 (clean, controlled setup):
1.  **No-skip encoder–decoder** (control)
2.  **U-Net (full skip connections)**
3.  **ResNet-101 + U-Net decoder (your paper model)**

---

### 2) Training phase (same for all models)

#### Input
*   NYUv2 RGB images

#### Target
*   Ground truth surface normals

#### Conditions
*   Train on **clean images only**
*   Same loss (e.g., cosine/angular loss), epochs, and batch size to ensure fairness.

---

### 3) Evaluation phase (2 stages)

#### Stage A — Clean evaluation
Evaluate all models on **clean validation set**.
*   **Metrics:** Mean Angular Error (MAE), Median error, % within thresholds.
*   **Goal:** Determine which model is best under ideal conditions.

#### Stage B — Photometric stress evaluation
Apply transformations **only at inference**.
*   **Stress types:** Shadows, Exposure (bright/dark), Gamma, Contrast, Noise.
*   **For each stress:** Run all 3 models and compute the same metrics.

---

### 4) Key metric (your main contribution)

#### Error drop / increase
\[ \Delta = \text{Error}_{stress} - \text{Error}_{clean} \]
This is the MOST important number.

---

### 5) How your results table will look

#### Table 1 — Clean performance
| Model        | MAE ↓  | Median ↓ |
| :----------- | :----- | :------- |
| No-skip      | Higher | Higher   |
| U-Net        | Lower  | Lower    |
| ResNet+U-Net | Lowest | Lowest   |

#### Table 2 — Under stress (example: shadows)
| Model        | MAE (stress) | Δ Error    |
| :----------- | :----------- | :--------- |
| No-skip      | Medium       | **Small**  |
| U-Net        | High         | **Large**  |
| ResNet+U-Net | Very High    | **Very Large** |

#### Table 3 — Across all stresses
| Model        | Shadow Δ | Exposure Δ | Noise Δ | Avg Δ   |
| :----------- | :------- | :--------- | :------ | :------ |
| No-skip      | Low      | Low        | Low     | **Low** |
| U-Net        | High     | Medium     | Medium  | **High** |
| ResNet+U-Net | Highest  | High       | High    | **Highest** |

---

### 6) Optional but very strong (recommended)

#### Edge vs Non-edge evaluation
Split pixels into Edge regions (high gradient) and Non-edge regions.
*   **Expected result:** Skip models fail **mostly on edges (especially shadow edges)**. This directly supports your hypothesis.

---

### 7) Visual results (very important for report)
For each model show the input image (clean vs stressed) and predicted normals.
*   **Highlight:** Fake edges in skip models; smooth but stable normals in no-skip.

---

### 8) Final interpretation (what you will conclude)
From your tables, you will say:
1.  **Skip connections:** Improve accuracy on clean data but increase sensitivity to photometric changes.
2.  **No-skip model:** Less accurate but more robust.
3.  **Conclusion:** Skip connections introduce a **robustness–accuracy trade-off** due to propagation of high-frequency photometric signals.

---

### 9) Final pipeline (clean version)
1.  Train all 3 models on clean data.
2.  Evaluate on clean → record baseline.
3.  Apply photometric stresses (inference only).
4.  Evaluate again.
5.  Compute Δ error.
6.  Compare across models.
7.  Analyze edge regions.
8.  Show qualitative results.

---

### Final takeaway
Your pipeline is simple but powerful: **Train once (clean) → Test under stress → Measure how much performance drops.** That drop (Δ) is what proves your hypothesis about skip connections.

---

## edge/non edge

## Summary
“Edge vs Non-edge evaluation” isolates **where** the error occurs. It lets you show that degradation under photometric stress is **concentrated at edges**—and primarily driven by **early skip connections** that carry edge/gradient signals.

---

### 1) What “Edge vs Non-edge” means
You split the image pixels into two sets:
*   **Edge pixels:** Locations with **high image gradient** (strong intensity change). Includes true object boundaries (wall–floor) and **shadow boundaries** (critical for your hypothesis).
*   **Non-edge pixels:** Smooth regions (walls, floors, flat surfaces) with low gradient.

---

### 2) How to compute the split (practical)
Use the **input RGB image** (not the prediction):
1.  Convert to grayscale.
2.  Compute gradient magnitude (e.g., Sobel).
3.  Threshold: Top ~10–20% → **edge mask**; Rest → **non-edge mask**.

This gives two boolean masks: \( M_{\text{edge}} \) and \( M_{\text{non-edge}} \).

---

### 3) What you measure
Compute your normal error **separately**: \( \text{MAE}_{\text{edge}}, \text{MAE}_{\text{non-edge}} \).
And under stress: \( \Delta_{\text{edge}}, \Delta_{\text{non-edge}} \).

---

### 4) How it fits into your pipeline
1.  **Step 1 — Train:** Same as before (clean data).
2.  **Step 2 — Clean evaluation:** Compute Global MAE and **Edge MAE / Non-edge MAE**.
3.  **Step 3 — Apply photometric stress:** Shadows, exposure, etc.
4.  **Step 4 — Stress evaluation:** Compute Global MAE and **Edge MAE / Non-edge MAE**.
5.  **Step 5 — Compute Δ (core analysis):** 
    *   \( \Delta_{\text{edge}} = \text{Error}_{\text{edge}}^{\text{stress}} - \text{Error}_{\text{edge}}^{\text{clean}} \)
    *   \( \Delta_{\text{non-edge}} = \text{Error}_{\text{non-edge}}^{\text{stress}} - \text{Error}_{\text{non-edge}}^{\text{clean}} \)

---

### 5) What results you expect (this is the key)

#### For Skip models (U-Net, ResNet+U-Net)
*   **Edge:** **Large error increase**, especially at shadow edges.
*   **Non-edge:** Small / moderate change.

#### For No-skip model
*   **Edge:** Already blurry → small increase.
*   **Non-edge:** Very stable.

---

### 6) What this proves (important)
Without this analysis, you only show “error increased.” With this, you show **where and why it increased**. Your conclusion becomes much stronger:
> The performance degradation under photometric stress is **not uniform**, but is concentrated in **high-gradient regions**, indicating that **skip connections propagate photometric edge artifacts into geometric predictions**.

---

### 7) Why this directly links to your hypothesis
Early skips carry gradients and edges. Shadows = gradients. So if the issue is due to skips, you should see errors spike specifically at **edges**.

---

### 8) Optional refinement (very strong for report)
Separate edges further into **True geometry edges** (depth discontinuity) and **Shadow edges** (no depth change). Then show that skip models fail **more on shadow edges than real edges**. This is a very strong validation of your idea.

---

### 9) Final mental model
*   Global error → “How much did model fail?”
*   Edge vs non-edge → “Where did it fail?”
*   Your hypothesis → “Why did it fail?”

---

### Final takeaway
This step is critical because it moves your work from **observation → explanation** and directly connects skip connections, edge information, and photometric artifacts.

---

## Possible Critique: Brutally Honest Audit

### The Verdict: "A High-Tier Diagnosis of a Known Problem"
*   **Is it decent?** Yes. It is significantly better than most undergraduate projects because it performs **Interpretability Research** (diagnosing *why*) rather than just **Application Research** (building a tool).
*   **Is it impressive?** Not yet. Right now, it is **Predictable**. Experienced researchers know that skip connections cause photometric leakage. If the conclusion is just "Shadows make U-Nets fail," it risks being a "water is wet" study.

---

### 1. The "Captain Obvious" Risk
The "No Free Lunch" theorem (Accuracy vs. Robustness) is well-known. A simple table showing an accuracy drop is a **predictable paper**.
*   **The Solution:** Go deeper into the **"Early vs. Deep"** skip connection split. If you can prove exactly *which* layer of the ResNet-101 is the "traitor" that lets the shadow in, that is the "Gold Nugget" that makes the project impressive.

### 2. The "Synthetic Trap"
Synthetic shadows are mathematically perfect.
*   **The Brutal Truth:** Real-world shadows have penumbra, color-bleeding, and sensor noise. Testing only on "Photoshop" shadows limits the research's real-world applicability (e.g., for robotics or AR).
*   **The Solution:** Include a small "Real-World Stress Test" using datasets like **DIODE** or **iBims-1** which contain natural variation.

### 3. The "Legacy" Problem
Using **Bae et al. (2021)** as a SOTA is safe but "old" in AI years.
*   **The Brutal Truth:** Modern SOTA has moved toward Transformers.
*   **The Solution:** To move from a "student project" to a "SOTA survey," compare **CNN Skips** vs. **Transformer Global Attention** (e.g., Metric3D v2). Does a Transformer "hallucinate" shadows like a U-Net?

### 4. The "Strong" Points (Why this works)
*   **The Tracing Paper Logic:** Demonstrates an understanding of **Inductive Bias**—a senior-level engineering perspective.
*   **Edge-Region Analysis:** Splitting pixels into "Edge" vs. "Non-Edge" is the smartest diagnostic in the protocol. It provides the mathematical "smoking gun."
*   **The 3-Model Spectrum:** A clean, easy-to-defend ablation study that is difficult to find flaws in.

---

### Final Grade for Strategy
*   **Clarity:** 10/10
*   **Rigor:** 8/10
*   **Innovation:** 5/10 (Known problem, needs the "Selective Skip" layer for the "Wow" factor)
*   **Feasibility:** 10/10 (Guaranteed to produce results and a clear thesis)

---

## Possible Contributions: Defining the Research Value

### The Core Idea
The idea that “skip connections can propagate low-level signals” is known. What is **not well established** is a **clear, quantified, and experimentally isolated link between skip connections and photometric robustness in surface normal estimation**. That gap is exactly where this research sits.

---

### 1. What is NOT clearly established (The Contribution)
This work is not repeating theory—it is **testing a specific, structured hypothesis**:
> *Do skip connections cause systematic geometric errors under photometric stress, and can we isolate which skip pathways are responsible?*

#### The novelty comes from 3 things:
1.  **Task-specific focus (Surface Normal Estimation):** Most prior work focuses on segmentation or classification. SNE is much more sensitive; small edge errors lead to large angular errors.
2.  **Photometric stress protocol:** Training on clean and testing on controlled photometric perturbations isolates **robustness behavior**, not just accuracy. This is not standard in many works.
3.  **Architectural isolation (Most important):** We are not just comparing models—we are isolating **No-skip, Full-skip, and Early-vs-Deep skip** configurations to answer: *Which part of the architecture causes the failure?*

---

### 2. What this paper can actually claim
1.  **Contribution 1:** Skip connections improve accuracy but introduce a measurable robustness trade-off under photometric stress.
2.  **Contribution 2:** Performance degradation is concentrated in **high-gradient regions**, not uniformly distributed.
3.  **Contribution 3 (Strongest):** Early (high-resolution) skip connections are the primary pathway through which photometric artifacts propagate into geometric predictions.
4.  **Contribution 4 (Ablation focus):** Selectively modifying skip pathways can improve robustness without fully sacrificing accuracy.

---

### 3. Why this is NOT trivial
We are turning a vague intuition into a **quantified, experimentally validated mechanism**.
*   **Known idea:** “Edges can be affected by lighting.”
*   **This work:** “In SNE, early skip connections cause a **ΔX° increase in angular error under shadows**, specifically at edge regions, due to propagation of high-frequency gradients.”
👉 That is publishable-level specificity.

---

### 4. Why this matters (Practical Impact)
*   **Model Design:** Should we filter skip connections? Gate them? Reduce early skip strength?
*   **Robust Perception Systems:** Robotics, Autonomous Driving, and AR/VR all depend on geometry under varying lighting.
*   **Future Research Direction:** “Photometrically aware architectures” and “Robust skip mechanisms.”

---

### 5. Final Takeaway
The value of this research is in turning a **general intuition** into a **precise, experimentally validated insight**, identifying a **specific architectural failure mechanism**, and providing a foundation for **better, more robust models**.

---

## Problem/Possible Solutions: Upgrading the Research Insight

### Summary
Below is a **concrete, step-by-step plan** to address each critique and upgrade the project from “predictable” → “insightful.” We focus on three upgrades:
1.  **Pinpoint which skip layer fails (core contribution)**
2.  **Validate on real-world data (robustness)**
3.  **Optional: compare with a transformer (modern relevance)**

---

### 1) Solve the “Captain Obvious” Risk: Identifying which skip connection causes the failure

#### Step 1 — Map skip connections in the model
For **ResNet-101 encoder + U-Net decoder**, the skips come from:
| Layer   | Resolution | Type       |
| :--- | :--- | :--- |
| **Conv2_x** | High       | Early skip |
| **Conv3_x** | Medium     | Mid skip   |
| **Conv4_x** | Low        | Deep skip  |

#### Step 2 — Create ablation models
Train these variants:
*   **A. Full model (baseline):** All skips enabled.
*   **B. No-skip model:** Remove all skips.
*   **C. Early-skip removed (IMPORTANT):** Remove **Conv2_x skip only**. (Tests: Are high-frequency edges the problem?)
*   **D. Deep-skip removed:** Remove **Conv4_x skip only**. (Tests: Are semantic skips involved?)
*   **E. Only early skip:** Keep Conv2_x, remove others. (Tests: “edge-only model”)

#### Step 3 — Run full evaluation & Analysis
Look for: Does removing early skip reduce Δ error significantly? Does removing deep skip change little?
**Final output (The "Gold Nugget"):** *“Conv2_x skip contributes X% of the photometric degradation, confirming early high-resolution features are the primary leakage pathway.”*

---

### 2) Solve the “Synthetic Trap”: Real-world Robustness

#### Step 1 — Add real-world datasets
Use **DIODE** or **iBims-1**.
#### Step 2 — Evaluation plan
Do NOT retrain; just run inference. Measure the same metrics (MAE, etc.) and check if the skip model still degrades more and if errors still concentrate at edges.
**Final output:** *“The observed failure mode persists in real-world data, confirming it is not an artifact of synthetic augmentation.”*

---

### 3) Solve the “Legacy Problem”: CNN vs. Transformer

#### Step 1 — Add a transformer model (Optional but strong)
Example: **SegFormer** or any normal-estimation transformer.
#### Step 2 — Compare behavior
| Model Type  | Behavior                    |
| :--- | :--- |
| **CNN (U-Net)** | Strong degradation at edges |
| **Transformer** | ?                           |
**Final output:** *“Transformer-based models exhibit reduced sensitivity to photometric edges, suggesting the failure mode is tied to CNN skip pathways.”*

---

### 4) Strengthen Edge Analysis
*   **Step 1:** Compute edge masks based on gradient thresholds.
*   **Step 2:** Evaluate Edge MAE and Non-edge MAE per model.
*   **Step 3:** Combine with ablations to show that No-early-skip models significantly reduce edge error.

---

### 5) Final Upgraded Pipeline
1.  **Phase 1 — Training:** Full skip, No skip, Early-skip removed, Deep-skip removed.
2.  **Phase 2 — Evaluation:** Clean (Baseline), Synthetic stress (Δ error), Real-world data (Validation).
3.  **Phase 3 — Analysis:** Global error, Δ error, Edge vs. Non-edge, Compare ablations.

---

### Final Takeaway
To upgrade the project: Don’t stop at **“skip vs. no-skip”**. Push into **Which skip? (core)**, **Does it hold in real data?**, and **Is it architecture-specific?**

---

## Summary of Deep-Dive Discussion (Research Evolution)

Our recent technical investigation has crystallized the research from a broad survey into a **surgical, architectural diagnostic**. Key insights include:

1.  **The Skip-Connection Dichotomy:** We distinguished between **Short Skips** (ResNet-style for training health) and **Long Skips** (U-Net-style for spatial sharpness). Our research isolates the **Long Skips** as the primary pathway for photometric noise leakage.
2.  **The Artifact-Free Upsampling:** We adopted the **Google Brain (2016)** discovery: separating upsampling from refinement. We will use **Bilinear Interpolation** (smooth/blurry) followed by **Convolutional Refinement** to avoid checkerboard artifacts that ruin normal estimation.
3.  **The "Cheating" Hypothesis:** We established the **Tracing Paper Analogy**. Skip connections act as tracing paper that allows the model to "cheat" by copying RGB edges. We hypothesize that this "cheating" makes the model interpret 2D shadows as 3D geometry.
4.  **The Diagnostic "Smoking Gun":** We identified **Edge vs. Non-Edge Analysis** as the mathematical proof needed to show that errors are geographically tied to high-gradient lighting artifacts.

---

## Experiment Details: Finalized Research Pipeline

This section outlines the **Finalized Training and Evaluation Blueprint** for the B.Tech Honours thesis.

### 1. Overall Training Philosophy
We define the model as:
\[ \hat{N}(x) = D_\phi(E_\theta(x)) \]
*   \(E_\theta\): **ResNet-101 Encoder** (ImageNet Pretrained).
*   \(D_\phi\): **Symmetric Decoder** (5-Stage Bilinear-Refinement).
*   **Experimental Variable:** Skip Connections (Present vs. Absent).

### 2. Decoder Design (Research-Correct)
To ensure scientific validity, the decoder architecture is kept **identical** across all models; only the skip pathways are toggled.
*   **Structure:** 5 stages of `Upsample (Bilinear) -> Concat(Optional Skip) -> Conv(3x3) -> Conv(3x3)`.
*   **Resolution Flow:** \(7 \times 7 \times 2048\) (Bottleneck) \(\rightarrow 14 \times 14 \rightarrow 28 \times 28 \rightarrow 56 \times 56 \rightarrow 112 \times 112 \rightarrow 224 \times 224 \times 3\) (Output).

### 3. Training Protocol (The 2-Phase Strategy)
We use a **Freeze-then-Unfreeze** approach to ensure the model leverages ImageNet knowledge without destabilizing.
*   **Phase 1 (Stabilization):** Encoder frozen. Train only the Decoder for 10-20 epochs. Maps fixed features to normals.
*   **Phase 2 (Fine-Tuning):** Unfreeze Encoder with a **10x lower learning rate**. Train for 30-50 total epochs to adapt to NYUv2 indoor geometry.

### 4. Loss Function & Output
*   **Output Tensor:** \(H \times W \times 3\) (RGB representation of unit vectors).
*   **Constraint:** Every pixel vector is L2-normalized: \( \hat{n} = \frac{n}{\|n\|_2 + \epsilon} \).
*   **Loss:** **Cosine Similarity Loss**: \( L = 1 - (\hat{n} \cdot n) \). This measures the angle between the prediction and ground truth, independent of magnitude.

#### Optimization vs. Evaluation: Loss vs. Metrics
While both measure "badness," they have different jobs in the research pipeline:

**The Loss (Cosine Similarity): The "Coach"**
*   **Purpose:** Used during training to update the model weights via Backpropagation.
*   **Requirement:** It MUST be differentiable (the computer must be able to calculate its slope/gradient).
*   **Math:** \( L = 1 - (\hat{n} \cdot n) \).
*   **Why we use it:** It is mathematically "smooth." If the model is slightly off, the loss tells the optimizer exactly which way to "turn" the 3D arrow to get closer to the truth.

**The Metrics (Mean/Median Error): The "Judge"**
*   **Purpose:** Used during evaluation to compare your work with other papers (like Wang et al. or Bae et al.).
*   **Requirement:** It must be Human-Readable and Standardized. It does not need to be differentiable.
*   **Math:** \( \theta = \arccos(\hat{n} \cdot n) \).
*   **Why we use it:** We think in degrees (\( 15^\circ \) error), not in cosine values (0.034 loss). The metrics tell a human story: "Is this model good enough for a robot to see a wall?"

---

**Pixel-wise Calculation & Aggregation**
Surface Normal Estimation is a **Dense Prediction** task.
*   **Calculation:** Loss and metrics are calculated for every single one of the \( 224 \times 224 \) pixels.
*   **Aggregation:** We calculate the error for each pixel, then take the **Average** (Mean Error) or find the **Middle Value** (Median Error) for the entire image.
*   **Invalid Pixels:** In NYUv2, some pixels are "empty" (no depth data). We use a **Mask** to ignore these pixels so they don't mess up our average.

---

**Summary Table: Training vs. Evaluation**

| Feature | Loss (Cosine Similarity) | Metric (Mean Angular Error) |
| :--- | :--- | :--- |
| **When?** | During Training (Every Batch) | During Testing (Final Score) |
| **Used by?** | The Optimizer (Adam/SGD) | The Researcher (You/Professor) |
| **Math Unit** | Unitless (0.0 to 2.0) | **Degrees** (\( 0^\circ \) to \( 180^\circ \)) |
| **Differentiable?** | Yes ✅ | No (Arccos is tricky at 0) |
| **Scope?** | All Valid Pixels | All Valid Pixels |

---

**Final Research Insight for the Demo:**
When you show your results, you won't just say "The loss was 0.05." You will say:
> *"Sir, following the **Wang et al.** protocol, the **Mean Angular Error** was \( 12.4^\circ \). However, our **Edge-Specific Analysis** shows that under shadow stress, the error at boundaries increases by **300%** due to skip-connection leakage."*

---

### 5. Evaluation Protocol (Clean vs. Stressed)
*   **Stage A (Clean):** Establish baseline MAE and Median Error on NYUv2 validation set.
*   **Stage B (Stress):** Apply **Photometric Stressors** (Inference-only). Measure the **Error Increase (\(\Delta\))** and perform **Edge-Region Analysis** to pinpoint the failure mechanism.

---

## Evaluation Protocol & Scientific Citations

To ensure our research is benchmarked against the highest academic standards, we strictly follow the evaluation protocol established in the literature. Our study sits at the intersection of three foundational works:

### 1. The Lineage of our Data & Evaluation
*   **Data Source (Ladicky et al. 2014):** We use the NYUv2 dataset where the Ground Truth (GT) surface normals were precomputed by Ladicky et al. using mathematical optimization over 3D point clouds. This provides the "labels" our model tries to predict.
*   **The Standard Exam (Wang et al. 2015):** We follow the rigid evaluation protocol defined in *"Designing Deep Networks for Surface Normal Estimation."* This includes:
    *   **The Split:** A fixed **654-image test set** from NYUv2. Benchmarking on any other split would invalidate our comparison with SOTA.
    *   **The Metrics:** We report the five standard angular error metrics (Mean, Median, and % within thresholds).
*   **The Subject (Bae et al. 2021):** We use the modern ResNet-101 + U-Net architecture as our primary experimental subject to study skip-connection vulnerability.
*   **The Stress Test (This Research, 2026):** We are the first to subject this established lineage to **structured photometric stress** to quantify the "Accuracy-Robustness Trade-off" in 3D perception.

---

### 2. The Five Metrics of Success (The Wang et al. Scorecard)
We evaluate the angular error (\( \theta \)) between the predicted normal vector (\( \hat{n} \)) and the ground truth (\( n \)) at every pixel.

| Metric | Calculation | Goal |
| :--- | :--- | :--- |
| **Mean Error ↓** | Average of all pixel angular errors. | Lower is better. |
| **Median Error ↓** | Middle value of error (robust to outliers). | Lower is better. |
| **\( \delta < 11.25^\circ \) ↑** | % of pixels with error less than \( 11.25^\circ \). | Higher is better. |
| **\( \delta < 22.5^\circ \) ↑** | % of pixels with error less than \( 22.5^\circ \). | Higher is better. |
| **\( \delta < 30.0^\circ \) ↑** | % of pixels with error less than \( 30.0^\circ \). | Higher is better. |

---

### 3. Formal Dataset Citation (Ready for Report)
> *"We evaluate on the NYUv2 dataset (Silberman et al. 2012) following the standard 654-image test split defined by Wang et al. (2015). Ground truth surface normals were obtained from the precomputed maps provided by Ladicky et al. (2014). This protocol ensures our results are directly comparable to modern SOTA models like Metric3D v2 (2024)."*

---

## Technical Deep-Dive: Implementation Progress

This section provides a rigorous record of the engineering decisions and sub-steps completed during the initialization of the research pipeline.

---

### Step 1: Data Loading & Environment Stabilization ✅
**Objective:** To establish a "Source of Truth" that is protected from accidental corruption while providing a high-performance environment for the GPU.

#### Sub-steps & Justification:
1.  **Container Mirroring:** We used a shell-based recursive clone (`cp -rp`) to move the `nyu_depth_v2_labeled.mat` (2.8GB) and the Ladicky Normal PNGs into a dedicated research directory.
    *   *Why:* This protects the original downloads. If a script fails, we delete the mirror, not the primary evidence.
2.  **Path Standardization:** We consolidated different naming conventions (e.g., `normals_gt/nyu-normal/`) into a unified folder structure.
    *   *Why:* Ensures that the indexing (1-based for NYUv2) matches exactly between RGB and Normal modalities.
3.  **HDF5 Integration:** We established `h5py` as the interface for the `.mat` file.
    *   *Why:* Standard PIL readers cannot open HDF5 containers. `h5py` allows us to "slice" specific images directly into RAM without loading the entire 3GB file.

---

### Step 2: High-Precision Preprocessing & Dataset Compilation ✅
**Objective:** To move the data from the "Storage Domain" (quantized integers) to the "Scientific Domain" (continuous floats) and lock it into a high-performance format.

#### Sub-steps & Justification:
1.  **Domain Mapping ([0, 255] → [-1, 1]):** We immediately converted the 8-bit PNG normals into 32-bit floating-point tensors.
    *   *Why:* PNGs cannot store negative numbers. 3D geometry requires the full range across the X, Y, and Z axes. Performing this once prevents repeated math during training.
2.  **Statistical RGB Normalization (ImageNet):** We applied the Mean `[0.485, 0.456, 0.406]` and Std `[0.229, 0.224, 0.225]`.
    *   *Why:* Our ResNet-101 encoder expects inputs to have a specific distribution. Matching this "ImageNet Signal" ensures the pretrained weights function as intended.
3.  **Geometric Integrity (L2-Normalization):** We enforced \( \|n\|_2 = 1.0 \) at two critical points: after domain mapping and after spatial resizing.
    *   *Why:* **Bilinear Interpolation** averages vectors, which mathematically "shrinks" them (interpolation shrinkage). Re-normalizing ensures our Ground Truth represents perfect physical orientations.
4.  **Spatial Alignment (Bicubic vs. Nearest-Neighbor):**
    *   **RGB:** Used **Bicubic** resizing to 224x224 to maximize edge contrast for the encoder.
    *   **Normal:** Used **Bilinear** for smooth geometric transitions.
    *   **Mask:** Used **Nearest-Neighbor** to maintain the binary (Valid/Invalid) nature of the data without blurring.
5.  **Coordinate Stability (`align_corners=False`):** We strictly used the modern boundary-alignment standard.
    *   *Why:* Prevents "geometric drift" where objects appear to stretch toward the corners of the photo, ensuring the RGB edge and Normal edge stay pixel-aligned.
6.  **Tensor Preservation (`.pt` compilation):** We saved the final results as PyTorch `.pt` files in a `transformed/` folder.
    *   *Why:* **Bit-Exactness.** PNGs round numbers; `.pt` files store the exact 32-bit decimal. This also removes the "CPU Bottleneck" during training, making the pipeline 5x-10x faster.

---

### Step 3: High-Precision Architecture Implementation ✅
**Objective:** To construct a "Surgical Testbed" that allows for the perfect isolation of the Skip Connection Axis while maintaining state-of-the-art geometric performance.

#### Sub-steps & Justification:
1.  **SNE_DecoderBlock (The Refinement Engine):**
    *   **Bilinear Upsampling vs. Transposed Convolutions:** We strictly use `F.interpolate(mode='bilinear')`. Unlike `ConvTranspose2d`, which often creates overlapping "hotspots" (checkerboard artifacts), Bilinear is a deterministic, smooth operation. This provides a clean geometric baseline for the subsequent refinement layers.
    *   **Surgical Axis Isolation (`combined_in`):** This is the heart of the experiment. The input channel count is dynamically calculated based on the `use_skip` flag. If `False`, the decoder is physically "blind" to the encoder's early gradients.
    *   **Residual Shortcut ($y = F(x) + W_s(x)$):** We implemented a **Projection Shortcut** using a $1 \times 1$ convolution. This solves the **Vanishing Gradient Problem** in the decoder, ensuring the abstract geometric signal is preserved even before the refinement layers are fully trained.
2.  **UniversalSNE (The Global Model):**
    *   **Encoder Hierarchy (ResNet-101):** We leverage a ResNet-101 backbone for its stable multi-scale feature extraction. It captures features at five distinct spatial resolutions ($224 \rightarrow 112 \rightarrow 56 \rightarrow 28 \rightarrow 14 \rightarrow 7$).
    *   **Symmetric Decoder Mapping:** The decoder blocks mirror the encoder's spatial decay exactly. This ensures that when we toggle the skips, the high-resolution features from the encoder precisely align with the upsampled features in the decoder, preventing **Spatial Drift**.
    *   **Refined Output Head:** We use a 3-stage refinement sub-network at the final $224 \times 224$ resolution. This allows the model to perform a final "Geometric Smoothing" before the data is projected into the final 3-channel space.
    *   **L2 Projection (The Unit Sphere Mandate):** The final operation is `F.normalize(p=2)`. This projects the 3D output tensor onto a **Unit Sphere Manifold**, forcing the model to respect the physical constraint that surface normals are unit directions.

---

### Step 4: The 4-Point Scientific Audit ✅
**Objective:** To mathematically verify the model's structural and geometric integrity before committing to the 50-epoch training cycle.

#### Sub-steps & Justification:
1.  **Geometric Integrity (Unit Vector Check):**
    *   **The Test:** We calculate the Euclidean Norm ($L2$) of every pixel vector: $\sqrt{x^2 + y^2 + z^2}$.
    *   **The Result:** A Pass (Average Magnitude = 1.000000) confirms that our output is a valid geometric direction. This is required for the **Cosine Similarity Loss** (Step 7) to be mathematically stable and valid.
2.  **Encoder Freezing Logic Audit:**
    *   **The Test:** We verify the `requires_grad` attribute of the encoder tensors after calling `set_encoder_frozen(True)`.
    *   **The Result:** This confirms our **2-Phase Training Protocol**. It ensures that the **ImageNet-pretrained features** are protected from being corrupted by the randomly initialized decoder during the initial stabilization phase.
3.  **Gradient Flow (Backpropagation Success):**
    *   **The Test:** We execute a "Mini-Backprop" using a dummy loss.
    *   **The Result:** This is a **Connectivity Test**. It proves that gradients can flow across the L2-normalization, residual shortcuts, and bilinear interpolations. This guarantees that the weights *can* update once training begins.
4.  **Axis Isolation (The Parameter Delta):**
    *   **The Test:** We compare the total parameter count of the model with skips ON (~64.1M) vs. skips OFF (~57.8M).
    *   **The Result:** This is the **Empirical Proof of Ablation**. The difference of **~6.23 Million parameters** represents the additional filters required to process the skip signals. This validates that our "Surgical Switch" is physically active.

---

## 11-Step Pipeline Summary: The Implementation Sequence

This table outlines the end-to-end execution of our research, from data ingestion to scientific diagnosis.

| Step | Action | Justification (Scientific Rigor) |
| :--- | :--- | :--- |
| **1** | **Data Loading** | Load NYUv2 `.mat` (RGB) + Ladicky PNGs (Normals). | **Standardization:** Uses the benchmark pair defined by Silberman (2012) & Ladicky (2014). |
| **2** | **Preprocessing** | Resize to $224 \times 224$; Map normal values to $[-1, 1]$. | **Compatibility:** Matches ResNet-101 input standards and 3D vector geometry. |
| **3** | **Architecture** | ResNet-101 Encoder + Symmetric Universal Decoder. | **Fairness:** Matches Bae et al. (2021) backbone to eliminate confounding variables. |
| **4** | **Upsampling** | **Bilinear Interpolation + 3x3 Conv Refinement.** | **Artifact Prevention:** Follows Google Brain (2016) to ensure smooth geometric planes. |
| **5** | **Train Phase 1** | **Freeze Encoder;** Train only Decoder (10-20 epochs). | **Stability:** Protects ImageNet features from being "corrupted" by early training noise. |
| **6** | **Train Phase 2** | **Unfreeze Encoder;** Low Learning Rate (30-50 total epochs). | **Finetuning:** Adapts the general "visual brain" to specific indoor geometry rules. |
| **7** | **The Loss** | **Cosine Similarity Loss** (per-pixel, masked). | **Physical Accuracy:** Minimizes the angular error between 3D vectors. |
| **8** | **The Scorecard** | Calculate **Wang et al. (2015)** Metrics (Mean, Median, $\delta$s). | **Reproducibility:** Standardizes the "Grade" against all SOTA SNE papers. |
| **9** | **The Stress** | Apply **5 Photometric Stresses** (Inference-only). | **Robustness Test:** Measures if the model generalizes beyond its "clean" training. |
| **10** | **The Diagnosis** | **Edge vs. Non-Edge Evaluation** (Sobel masks). | **Internal Audit:** Proves that errors concentrate specifically at lighting boundaries. |
| **11** | **The Axis** | Toggle `use_skip=True/False` and repeat Steps 8-10. | **Ablation Study:** Surgically isolates skip connections as the source of vulnerability. |

---

## 4. Top 5 Photometric Stresses (The "Stressor" Suite)
We will subject the model to these five structured degradations to identify where the "Geometric-Appearance Disentanglement" breaks.

| # | Stress Type | Implementation Method | Predicted Failure Mode |
| :--- | :--- | :--- | :--- |
| **1** | **Hard-Edge Shadows** | Multiplicative darkening of a localized geometric region (e.g., a diagonal slab across a wall). | **The "Fake Corner" effect:** The model predicts a sharp 3D edge/kink where the shadow starts. |
| **2** | **Under-exposure** | Global scaling (0.3x) + Gaussian sensor noise. | **Planar Flattening:** Large surfaces (walls) lose their orientation and "melt" into the background. |
| **3** | **Over-exposure** | Global scaling (2.5x) + Pixel clipping at 255. | **Texture Washout:** The model fails to distinguish between the wall and the floor because the "corner" contrast is lost. |
| **4** | **Warm/Cool Shift** | Channel-wise scaling (R+20%, B-20%) to simulate sunset/artificial lighting. | **Normal Bias:** The entire normal map "tilts" in 3D space because the model interprets color shifts as shading cues. |
| **5** | **Specular Highlights** | Addition of high-intensity "hotspots" on reflective surfaces. | **Surface Pitting:** The model predicts "holes" or "bumps" on flat surfaces where the light reflects most strongly. |

---

## 5. Visual Proof-of-Concept (The "Aha!" Moment)
For the professor, we will present a **4-Panel Comparison**:
1.  **Panel A (RGB):** The original NYUv2 room + Synthetic Shadow.
2.  **Panel B (GT):** The true Surface Normals (showing a perfectly flat wall).
3.  **Panel C (Pred):** The Bae et al. prediction (showing the "broken" geometry).
4.  **Panel D (Error):** An **Angular Error Heatmap** highlighting the exact pixels where the shadow "fooled" the architecture.

---

## 6. Immediate Implementation Steps
1.  **Scripting:** Write the `photometric_stressors.py` module in the Colab.
2.  **Model Loading:** Pull the `models/NNET.py` and `checkpoints/` from the [Bae et al. Repo](https://github.com/baegwangbin/surface_normal_uncertainty).
3.  **Inference:** Run the "Shadow Test" on a known flat-wall image from the NYUv2 test set.
4.  **Documentation:** Capture the visual failures for the demo PDF.
