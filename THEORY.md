# Theory & Concepts

## 2026-03-19: The "General" Nature of CNNs & Adaptation to Surface Normals

This section establishes the foundational understanding of how Convolutional Neural Networks (CNNs) serve as general-purpose feature extractors and how this universality applies to our specific research problem: **Geometric Perception Under Photometric Stress**.

### 1. The Universal Feature Extractor (The "Encoder")

At their core, CNNs are hierarchical feature extractors. Regardless of the final task—be it classifying a cat, segmenting a road, or estimating the 3D orientation of a wall—the initial process is identical.

**The Process:** `Image → Conv → Pool → Conv → Pool → Deep Features`

*   **Low-level layers:** Detect edges, corners, and simple textures. In our context, these layers might detect the sharp line where a wall meets the floor.
*   **Mid-level layers:** Detect shapes and patterns. Here, the network might recognize a rectangular planar surface.
*   **High-level layers:** Encode semantic concepts and global context. Ideally, this is where the network understands "this is a vertical wall inside a room," regardless of whether it is well-lit or in shadow.

**Relevance to Our Research:**
Our hypothesis relies on this hierarchy. We posit that **deeper encoders** (with more abstraction) will be better at separating the "geometry" (the wall) from the "appearance" (the shadow on the wall) because they have moved beyond the low-level pixel intensity values that are corrupted by photometric stress.

### 2. The "Head" Defines the Task

The distinction between tasks lies solely in how these extracted features are processed at the end.

| Task | Head Function | Output Structure | Our Research Connection |
| :--- | :--- | :--- | :--- |
| **Classification** | **Compress:** Spatial dimensions are collapsed (Global Average Pooling) to a single vector. | `Class Label` (e.g., "Bedroom") | **Not used.** We need dense, per-pixel predictions. |
| **Segmentation** | **Expand:** Features are upsampled to the original image resolution. | `Class ID per pixel` (e.g., Pixel[0,0] = Wall) | **The Architecture Basis.** We borrow the *structure* (Encoder-Decoder) but change the *output*. |
| **Surface Normals** | **Expand:** Features are upsampled to the original image resolution. | `Unit Vector (x,y,z) per pixel` | **Our Core Task.** Instead of a discrete class, we regress continuous values representing 3D direction. |

### 3. Adaptation for Surface Normal Estimation

Adapting a standard CNN (like ResNet) for our task involves two surgical changes:

1.  **Removal of the Classification Head:** The fully connected layers and softmax are discarded.
2.  **Addition of a Geometric Decoder:**
    *   **Upsampling:** We must restore spatial resolution (using transpose convolutions or bilinear interpolation) to match the input image size.
    *   **Skip Connections:** We re-introduce low-level features from the encoder to the decoder to recover fine details (edges). *Crucial Hypothesis:** These skip connections might accidentally re-introduce photometric noise (shadows) that the deep encoder had successfully filtered out.
    *   **Normalization Layer:** We must decide how to normalize features (Batch vs. Instance vs. Layer Norm) to handle lighting variations.
    *   **Output Layer:** A final convolution with **3 channels** (representing x, y, z) followed by an **L2 Normalization** step to ensure every pixel is a unit vector ($x^2 + y^2 + z^2 = 1$).

### 4. The "Geometry vs. Appearance" Conflict

This general architecture highlights the central tension of our thesis:

*   **The Encoder's Job:** To extract "invariant" features (it *should* see the wall as a wall, even in the dark).
*   **The Skip Connection's Job:** To preserve "spatial" details (it passes the raw edge information).
*   **The Conflict:** Photometric stress (e.g., a hard shadow) looks like a spatial edge. A strong skip connection might pass this "shadow edge" to the output, fooling the model into predicting a geometric corner where there is only a light change.

**Conclusion:** The "generality" of the CNN is both its strength (powerful feature extraction) and its potential weakness (blindly passing low-level appearance cues). Our research investigates which architectural knobs (Depth, Skips, Norms) tune this trade-off to favor **geometric truth** over **photometric illusion**.

## 2026-03-19: Surface Normals - The "Invisible" Geometry in 3 Numbers

This section deconstructs the target output of our models: the surface normal vector $(n_x, n_y, n_z)$. Understanding this data structure is critical because our entire failure analysis (MAE, boundary errors) relies on correctly interpreting these 3 numbers.

### 1. The Physics: A Unit Direction Vector

At every single pixel $(u, v)$ in our input image, the model predicts a vector $\mathbf{n} = (n_x, n_y, n_z)$.
This vector represents the **orientation** of the surface at that exact point in 3D space.

*   **The Constraint:** It is a *unit vector*. Its length is always 1.
    $$n_x^2 + n_y^2 + n_z^2 = 1$$
*   **The Range:** Because it is a unit vector, each component must strictly lie between -1 and 1.
    $$-1 \le n_x, n_y, n_z \le 1$$

**Why this matters for Photometric Stress:**
Unlike RGB pixel values which can be arbitrarily bright or dark (0 to 255), surface normals are geometrically constrained. A wall facing "Left" is always $(-1, 0, 0)$. If a shadow falls on it, the RGB values drop (e.g., 200 $\to$ 50), but the **True Normal** remains $(-1, 0, 0)$.
*   **Our Hypothesis:** A non-robust model will let the RGB drop "leak" into the normal prediction, causing the vector to deviate from $(-1, 0, 0)$.

### 2. The Coordinate System (The "Perpendicular Axes")

The 3 numbers correspond to projections along three mutually perpendicular axes relative to the camera:

| Component | Meaning (Intuition) | Example Vector | Direction |
| :--- | :--- | :--- | :--- |
| **$n_z$ (Blue)** | Facing the Camera? | $(0, 0, 1)$ | Surface is flat, facing directly at you (e.g., a poster on the wall). |
| **$n_x$ (Red)** | Facing Left/Right? | $(1, 0, 0)$ | Surface is facing strictly Right. |
| **$n_y$ (Green)** | Facing Up/Down? | $(0, 1, 0)$ | Surface is facing strictly Up (e.g., the floor). |

### 3. Visualization: The "RGB Map" is a Lie (Sort of)

We often see colorful "Normal Maps" in papers. These are just visualizations. The model "thinks" in math vectors $[-1, 1]$, but we convert them to colors $[0, 255]$ to see them.

**The Conversion Pipeline:**
1.  **Raw Prediction:** $\mathbf{n} \in [-1, 1]$ (e.g., $-1$ for facing Left).
2.  **Shift & Scale:** Map $[-1, 1] \to [0, 1]$.
    $$n' = \frac{n + 1}{2}$$
3.  **Colorize:** Map $[0, 1] \to [0, 255]$.
    $$Pixel_{Red} = n'_x \times 255$$

**Implication for Research:**
When we visualize our "Failure Cases" in the paper:
*   **Greenish regions** usually correspond to floors ($n_y \approx 1$).
*   **Blueish regions** usually correspond to walls facing us ($n_z \approx 1$).
*   **Reddish regions** usually correspond to side walls ($n_x \approx 1$).
If a "Blue" wall suddenly turns "Purple" under shadow, we know the model has failed to separate geometry from lighting.

### 4. Mental Model for the Thesis

At each pixel, imagine a **tiny arrow** sticking out of the image.
*   **Input:** A degraded RGB pixel (dark, noisy, off-color).
*   **Task:** Predict the *direction* of the tiny arrow.
*   **Success:** The arrow points the same way as it does in the clean image.
*   **Failure:** The arrow twists because the model confused "darkness" for "curvature."

## 2026-03-19: Evolution of Dense Prediction Architectures

This section traces the lineage of the models we are comparing. Understanding *why* SegNet differs from U-Net is key to explaining why one might be more robust to shadows than the other.

### The Core Problem: "Shrink and Rebuild"

All these models answer the same question: **"After shrinking the image to understand it (Encoder), how do we rebuild it back to full size (Decoder)?"**

### 1. FCN (2014) - The "Stretch" Approach
*   **Logic:** Just use bilinear interpolation to stretch the tiny feature map back to full size.
*   **Result:** Blurry, blobby outputs.
*   **Robustness Theory:** Highly robust (because it ignores all fine detail), but useless for geometry because it can't see edges.

### 2. SegNet (2015) - The "Memory" Approach
*   **Logic:** When max-pooling (shrinking), remember the *indices* of the max values. Use these indices to un-pool later.
*   **Result:** Better structure, but still jagged.
*   **Robustness Theory:** Moderate. It doesn't pass raw pixel values, only locations. This might make it surprisingly robust to photometric shifts since it reconstructs from "memory" rather than "copying."

### 3. U-Net (2015) - The "Copy-Paste" Approach (Critical Baseline)
*   **Logic:** Don't just guess. **Copy** the high-resolution features from the encoder and **concatenate** them with the decoder.
*   **Result:** Extremely sharp edges.
*   **Robustness Theory (The Danger Zone):** By directly copying low-level features (which contain raw pixel intensity gradients), U-Net risks copying shadows and lighting artifacts directly into the prediction. **This is our primary suspect for low photometric robustness.**

### 4. ResNet (2015) - The "Deep Understanding" Approach
*   **Logic:** Use residual connections to allow training of extremely deep networks (100+ layers).
*   **Result:** High-level semantic understanding.
*   **Robustness Theory:** A ResNet encoder is our best bet for "invariance." The deeper we go, the more the model understands "concept" over "pixel."

### 5. FPN (2017) - The "Multi-Scale" Approach
*   **Logic:** Combine features from all scales (pyramid).
*   **Result:** Sees both the forest (global context) and the trees (local edges).
*   **Robustness Theory:** Balanced. The global context might overrule the local lighting errors.

### 6. Hourglass (2016) - The "Refinement" Approach
*   **Logic:** Encoder-Decoder-Encoder-Decoder... (Stacking).
*   **Result:** Iterative refinement of the prediction.
*   **Robustness Theory:** Potentially high. If the first pass is fooled by a shadow, the second pass (which sees the whole context again) might correct it.

### Summary for Our Thesis

We are essentially comparing:
1.  **U-Net Style (Concatenation):** "I trust the local details." (Risk: Trusts shadows too much).
2.  **FPN/ResNet Style (Summation/Depth):** "I trust the global context." (Hope: Ignores shadows).
3.  **Refinement Style (Iterative):** "I'll double-check my work." (Hope: Corrects initial errors).

## 2026-03-19: Theoretical Framework - Invariance, Equivariance, and the "Unsolved" Problem

This section formalizes the core theoretical argument of the research paper. It explains *why* the problem of photometric stress persists even in modern architectures and provides the specific mechanisms (Invariance vs. Equivariance) that we are testing.

### 1. The Core Misunderstanding: Abstraction $\ne$ Removal

It is a common misconception that deep encoders (like the "bottleneck" in ResNet) completely "remove" lighting information.
*   **Reality:** They abstract it, but they don't remove it.
*   **The Problem:** The model learns its abstraction on **clean, well-lit images**. Its internal representation of "geometry" is entangled with "normal lighting."
*   **The Failure Mode:** When you feed it an underexposed image, the deep features shift into a distribution the model has never seen. The "architect" (model) is experienced, but if you turn off the lights, even they can't see the walls.

### 2. The Visual Cues for Geometry (And how stress breaks them)

The model infers geometry from four cues. Photometric stress breaks the first two, forcing reliance on the last two:

| Cue | What it is | Effect of Photometric Stress |
| :--- | :--- | :--- |
| **1. Shading** | Brightness variation tells orientation (Render Equation). | **Catastrophic Failure.** Shadows/Exposure change brightness without changing orientation. |
| **2. Texture Gradients** | Texture compression reveals curvature. | **Moderate Failure.** Low contrast (underexposure) wipes out texture. |
| **3. Edges/Boundaries** | Discontinuities where surfaces meet. | **Resilient (mostly).** Edges usually survive, but can fade in dark shadows. |
| **4. Context** | "Floors are down, walls are up." | **Highly Robust.** Deep encoders rely on this. |

**Thesis Goal:** We want to find architectures that shift reliance from **Cue 1 (Shading)** to **Cue 4 (Context)**.

### 3. Formal Definition: Invariance vs. Equivariance

This is the mathematical language we will use in the paper:

*   **Invariance (Goal):** Output $Y$ stays the same when Input $X$ changes photometrically.
    *   $f(Lighting(X)) = f(X)$
    *   *We want the surface normal prediction to be invariant to lighting.*
*   **Equivariance (Reality):** Output $Y$ changes corresponding to the Input $X$ change.
    *   *Currently, models are equivariant to lighting—they change their prediction when the light moves.*

**Research Question:** Which architectural choices push the model closer to **Invariance**?

### 4. The Three Mechanisms of Failure (Our Hypotheses)

We identify three specific pathways through which photometric stress corrupts the geometric prediction. These map directly to our 10 Axes.

#### Mechanism 1: The "Mixed Cue" Conflation (Encoder Depth)
*   **Theory:** Shading is both geometric and photometric. The model learns to conflate them ("Darker = Curved away").
*   **Hypothesis:** Deeper encoders have more capacity to disentangle this, but never fully because the physics is ambiguous without known light sources.

#### Mechanism 2: The "Leakage" (Skip Connections)
*   **Theory:** Even if the deep encoder is perfect, U-Net style skip connections pipe raw, low-level features (which are 100% photometric) directly into the decoder.
*   **Hypothesis:** **Dense skip connections are the enemy of robustness.** They prioritize spatial sharpness (edges) at the cost of reintroducing shadow sensitivity.

#### Mechanism 3: The "Wrong Statistics" (Normalization)
*   **Theory:** BatchNorm learns "global average brightness" statistics from the training set.
*   **Hypothesis:** When tested on a dark image, the statistics don't match. **Instance Normalization** (which normalizes per-image) should theoretically be much more robust because it adapts to the specific lighting of the current image.

### Conclusion

This framework gives us a coherent story. Every experiment in the paper is designed to isolate one of these three mechanisms.

## 2026-03-19: Simplified Theoretical Summary & The "Why" of Failure

This section distills the complex physics into a simplified narrative suitable for the "Introduction" or "Motivation" section of the paper, explaining the root cause of the problem.

### 1. The Core Challenge: Inferring Fixed Geometry from Changing Appearance

*   **Input:** An RGB image (which mixes Lighting, Geometry, and Semantics).
*   **Output:** Surface Normals (Fixed Geometry).
*   **The Conflict:** A wall's geometry is constant, but its appearance changes wildly with lighting. The model must learn to ignore the appearance change to see the constant geometry.

### 2. The "Shading" Trap

The primary cue for geometry in an image is **Shading** (brightness variation).
$$Brightness = Surface Orientation \times Light Direction$$
*   **The Trap:** Shading is a product of *both* Geometry and Lighting.
*   **The Shortcut:** During training (on clean images), the model learns "Darker pixels = Curved away."
*   **The Failure:** Under photometric stress (e.g., underexposure), "Darker pixels" no longer means "Curved away"—it just means "Dark room." The model's shortcut fails because it conflated Geometry with Lighting.

### 3. Why the Three Mechanisms Matter (The Research Contribution)

This paper is not just "characterizing" failure; it is identifying *which part* of the brain breaks.

| Mechanism | Why it helps/hurts |
| :--- | :--- |
| **1. Encoder Depth** | **Abstracts away raw brightness.** A shallow encoder sees "dark pixels." A deep encoder sees "a wall." Deep encoders are theoretically better at ignoring lighting. |
| **2. Skip Connections** | **Re-introduces raw brightness.** Even if the encoder is smart, skip connections act like a "backdoor," letting raw, shadow-corrupted pixel values from the input flow directly into the final prediction. **Hypothesis: Skips hurt robustness.** |
| **3. Normalization** | **Calibrates expectations.** **BatchNorm** expects "normal lighting" based on training data. **InstanceNorm** adapts to the current image's lighting. **Hypothesis: InstanceNorm is more robust to exposure changes.** |

### 4. Conclusion: Characterization as Contribution

We are not "solving" the problem of photometric stress. We are **characterizing** it.
*   We confirm that the problem exists (it's not fully solved by modern models).
*   We map the problem to specific architectural choices (Depth, Skips, Norms).
*   This provides the "Why" that future solution papers will cite.

## 2026-03-19: Depth Map Visualization & Dataset Realities (NYUv2)

This section explains the practical data source for our research (NYUv2) and how we visualize the geometric ground truth (Depth) before it is converted to Surface Normals.

### 1. The Perceptual Colormap: Plasma

We use the **Plasma** colormap to visualize depth. It is designed to be "perceptually uniform," meaning the visual difference in color matches the mathematical difference in depth.

| Color | Depth Meaning | Intuition |
| :--- | :--- | :--- |
| **Dark Purple** | **Minimum Depth** | Closest to camera (e.g., chairs in the foreground). |
| **Blue / Violet** | **Close** | Foreground objects. |
| **Orange** | **Medium to Far** | Most of the room (walls, ceiling). |
| **Yellow** | **Maximum Depth** | Furthest surfaces (e.g., the center of the back wall). |

### 2. The NYUv2 "Pink/Magenta" Edge Problem

In the NYUv2 dataset, you will often see **Pink/Magenta** at the very edges of the depth map.
*   **The Reality:** This is often **missing depth data**. The Kinect sensor used for NYUv2 has "blind spots" at edges or on certain materials (glass, black surfaces).
*   **Research Implication:** We must be careful not to penalize models for failing on these "pink" regions, as the ground truth there is invalid. Our spatial failure analysis (planar vs. boundaries) should focus on valid depth regions.

### 3. Gradients vs. Discontinuities

*   **Smooth Gradients (Walls):** As a wall recedes into the distance, the color shifts smoothly from orange to yellow.
*   **Sharp Boundaries (Edges):** Where a chair (purple) meets a wall (orange), there is a sharp color jump.
*   **Connection to Normals:** Surface Normals are calculated from the **gradient** (rate of change) of this depth map.
    *   Smooth Gradient $\to$ Consistent Normal (Straight Blue).
    *   Sharp Jump $\to$ Flipped Normal (Edge detail).

### 4. Summary: Relative Scale

NYUv2 depth maps use **Relative Scaling**. The darkest purple is the closest point in *that specific image*, not an absolute 0 meters. This means a model must learn to understand the *relationship* between pixels rather than absolute values—making it theoretically more robust to global scaling but more sensitive to local distortions (like shadows).

## 2026-03-19: Data Representations & Ground Truth Generation for Surface Normals

This section addresses the technical details of normal vector representation and the methodologies for generating ground truth (GT) from raw depth data—crucial for maintaining a rigorous research pipeline.

### 1. Scaling: 0-1 (Normalised) vs. 0-255 (Quantized)

Both representations are mathematically equivalent; they are simply different storage conventions.
*   **Internal Computation (Code):** We use **normalised floats $[0.0, 1.0]$** (PyTorch/NumPy standard).
*   **External Storage (Image):** We use **8-bit integers $[0, 255]$** (PNG/JPEG standard).
*   **The Transformation:** To shift from the mathematical unit vector range $[-1, 1]$ to a displayable range:
    *   `Red = (nx + 1) / 2` (Gives $0.0$ to $1.0$)
    *   `R_int = Red * 255` (Gives $0$ to $255$ for storage)

### 2. Hierarchy of Depth-to-Normal Conversion Methods

The method used to generate ground truth determines the "quality ceiling" of your metrics.

#### Method 1: Simple Gradient (NumPy Gradient)
*   **Logic:** Compute $dz/dx$ and $dz/dy$ with `np.gradient`.
*   **Formula:** $Normal = [-dz/dx, -dz/dy, 1]$, then normalised.
*   **Limitation:** Fast but noisy. Good for visualization, not for academic ground truth.

#### Method 2: Least Squares Plane Fitting
*   **Logic:** For each pixel, fit a plane to its $5 \times 5$ or $7 \times 7$ neighbourhood.
*   **Advantage:** Robust to sensor noise by averaging over a local region. **Standard for many research datasets.**

#### Method 3: 3D Point Cloud Cross Product (Geometrically Exact)
*   **Logic:** Convert depth to 3D points using camera intrinsics ($f_x, f_y, c_x, c_y$).
*   **Formulas:**
    *   $X_{3d} = (pixel_x - c_x) \times depth / f_x$
    *   $Y_{3d} = (pixel_y - c_y) \times depth / f_y$
    *   $Z_{3d} = depth$
*   **The Normal:** Compute edge vectors between neighbours:
    *   $Edge_1 = Right_{3d} - Point_{3d}$
    *   $Edge_2 = Down_{3d} - Point_{3d}$
    *   $Normal = Normalise(Cross(Edge_1, Edge_2))$
*   **Advantage:** Geometrically exact; corrects for perspective distortion.

#### Method 4: Precomputed Ground Truth (Recommended)
*   **Logic:** Use high-quality normals pre-processed by established research groups (e.g., Wang et al., Eigen et al.).
*   **Why:** Ensures consistency and reproducibility across the research community.

### 3. NYUv2 Data Reality: Derived vs. Captured

*   **Raw NYUv2 mat file:** Contains only RGB and Depth. Normals are **derived**, not captured by the sensor.
*   **Research Implication:** Ground truth normals are fixed. Any measured drift in prediction under photometric stress is purely due to model failure to separate geometry from appearance.

## 2026-03-19: Intuition and Evolution of Depth-to-Normal Methods

This section traces the logical progression of how researchers developed methods to derive 3D orientation from 2D depth maps. Each step in this evolution was a direct response to a limitation in the previous method.

### 1. The Core Insight: Geometry is Shape

The fundamental insight connecting depth to normals is: **If you know the shape of a surface, you can find its orientation.** The shape of a surface is described by how depth changes across it.

### 2. Method 1: The Simple Gradient (Local Difference)
*   **Intuition:** Measure the height difference between two adjacent pixels.
*   **The Logic:** Calculate the slope in X and Y directions ($dz/dx$ and $dz/dy$).
*   **The Formula:**
    $$Normal = [-dz/dx, -dz/dy, 1]$$
*   **Limitation (Noise):** Like measuring a hill's slope from two blades of grass. Small sensor errors in two adjacent pixels create massive errors in the normal direction.

### 3. Method 2: Least Squares Plane Fitting (Neighborhood Averaging)
*   **Intuition:** Surfaces are locally flat. Average over a neighborhood to cancel out noise.
*   **The Logic:** Fit a plane through a patch (e.g., $5 \times 5$, 25 points) using least squares minimization.
*   **Advantage:** Robust; noise "averages out."
*   **Limitation (Boundaries):** At edges (chair vs. wall), fitting a plane to a neighborhood that spans both gives a meaningless average orientation.

### 4. Method 3: 3D Point Cloud Cross Product (Perspective Correction)
*   **Intuition:** Correct for the fact that image pixels are not equally spaced in real-world distances due to perspective.
*   **The Logic:** Work in real metric 3D space.
    1.  **Project to 3D:**
        $$X_{3d} = (pixel_x - c_x) \times depth / f_x$$
        $$Y_{3d} = (pixel_y - c_y) \times depth / f_y$$
        $$Z_{3d} = depth$$
    2.  **Compute Vectors:** $Edge_1 = Right_{3d} - Point_{3d}$, $Edge_2 = Down_{3d} - Point_{3d}$.
    3.  **Cross Product:** The normal is perpendicular to these two vectors lying in the surface plane.
        $$Normal = Normalise(Cross(Edge_1, Edge_2))$$
*   **Advantage:** **Geometrically Exact.** Perspective distortion is fully corrected.

### 5. Method 4: Precomputed Ground Truth (The Gold Standard)
*   **Intuition:** Fairness and reproducibility.
*   **The Logic:** One group computes a canonical, high-quality GT using Method 3 with careful preprocessing and smoothing (e.g., Wang et al.).
*   **Why we use it:** Benchmarking fairness. It acknowledges that ground truth is *derived*, not captured, and ensures every paper uses the same high-quality mathematical implementation.

### Conclusion: Research Relevance

We use **Method 4** to ensure that measured errors (MAE) are attributable to **architectural sensitivity to photometric stress** rather than inaccuracies in the geometric reference.

## 2026-03-19: Dataset Selection Rationale: Beyond NYUv2

This section evaluates various RGB-D datasets for their suitability in studying photometric robustness in surface normal estimation. It establishes why a multi-dataset approach provides a more rigorous characterization of architectural failure by isolating our primary variable of interest: photometric stress.

### 1. The Experimental Pipeline

Our research follows a strict "Train-on-Clean, Test-on-Degraded" protocol:

```
Clean RGB images → Apply photometric degradations → Feed to model → Compare predictions against ground truth normals
```

*   **No Retraining:** Models are never exposed to degraded data during training.
*   **Static Geometry:** Ground truth normals remain identical between clean and degraded tests because scene geometry is invariant to lighting.

### 2. "Better Science, Not Better Models" (The Core Logic)

A critical distinction in our methodology is that using high-quality datasets like iBims-1 is not intended to improve model performance, but to ensure **experimental cleanliness**.

*   **The Confounding Variable (Sensor Noise):** NYUv2 uses Kinect depth maps, which are inherently noisy. This makes it difficult to distinguish whether prediction errors are caused by **photometric degradation** or by **noisy ground truth**.
*   **Isolating the Variable of Interest:** By evaluating on a dataset with cleaner ground truth (like iBims-1), we can confidently attribute any degradation in metrics to the **architectural sensitivity** under study, rather than sensor artifacts. This significantly increases the scientific credibility of our findings.

### 3. Comparison of Datasets

| Dataset | Pros | Cons for Photometric Study | Role in Our Research |
| :--- | :--- | :--- | :--- |
| **NYUv2** | Huge benchmark, easy to compare with prior work. | Controlled indoor lighting; noisy Kinect sensor. | **Primary Training / Baseline.** |
| **iBims-1** | **High-quality laser-scanned GT.** Specifically designed for geometric evaluation. | Small scale. | **Secondary Evaluation.** |
| **ScanNet** | Large scale, more diverse scenes than NYUv2. | Still has some sensor noise. | Generalization testing. |
| **Matterport3D** | Very diverse environments, high quality depth. | Large scale complexity. | Generalization testing. |

### 4. Final Recommendation: The "Generalization" Strategy

We adopt a **Train-on-NYUv2, Evaluate-on-NYUv2 + iBims-1** strategy.
*   **Comparability:** NYUv2 keeps our results comparable to established SOTA literature.
*   **Rigor:** iBims-1's cleaner ground truth strengthens our failure analysis and proves that our findings generalize beyond a single dataset's specific sensor noise.

## 2026-03-19: DETAILED EXPERIMENT PIPELINE

**End to End Experiment Pipeline — Full Detail**

---

**PHASE 0 — Setup (done)**
```
Google Drive folder structure     ✓
NYUv2 raw dataset downloaded      ✓
RGB + Depth visualised            ✓
Surface normal visualised         ✓
Wang/Ladicky GT normals           ← Cell 9 pending
```

---

**PHASE 1 — Data Preparation**

```
Dataset 1 — NYUv2
├── Source: already downloaded (nyu_depth_v2_labeled.mat)
├── RGB images: 1449 total
├── Split: 795 train / 654 test (fixed standard split)
├── Ground truth normals: Wang/Ladicky precomputed (1449 PNGs)
└── Storage: BASE/dataset/NYUv2/

Dataset 2 — iBims-1 (secondary evaluation only, add if time allows)
├── Source: download separately when main experiments done
├── 100 high quality laser scanned images
├── Used only for evaluation, never training
└── Storage: BASE/dataset/iBims1/
```

**Data preprocessing steps:**
```
1. Load RGB images from .mat file
2. Load corresponding GT normal maps from Wang PNG files
3. Resize all images to 480x640 (standard NYUv2 resolution)
4. Normalise RGB: mean=[0.485, 0.456, 0.406] std=[0.229, 0.224, 0.225]
   (ImageNet normalisation stats — required since encoder is ImageNet pretrained)
5. Apply standard train augmentations (horizontal flip, random crop)
6. Save processed dataset as PyTorch Dataset class
7. Create DataLoader for train split (795 images)
8. Create DataLoader for test split (654 images)
```

---

**PHASE 2 — Photometric Degradation Pipeline**

```
Four degradation types applied synthetically at inference time only.
Never applied during training. Never touch ground truth normals.

Degradation 1 — Underexposure
Method: multiply RGB pixel values by factor 0.3
Simulates: dark room, night time, poorly lit scene
Implementation: img_degraded = img * 0.3

Degradation 2 — Overexposure  
Method: multiply RGB pixel values by factor 2.5, clip to 255
Simulates: harsh sunlight, blown out highlights
Implementation: img_degraded = np.clip(img * 2.5, 0, 255)

Degradation 3 — Shadows
Method: apply gradient mask darkening a region of the image
Simulates: cast shadows from windows or objects
Implementation: multiply region by 0.2 with soft edges

Degradation 4 — Day-Night / Colour Temperature Shift
Method: shift white balance toward warm (yellow) or cool (blue)
Simulates: artificial lighting vs daylight colour temperature
Implementation: multiply R,G,B channels by different factors

Storage structure:
BASE/results/degraded/underexposure/
BASE/results/degraded/overexposure/
BASE/results/degraded/shadows/
BASE/results/degraded/daynight/
```

---

**PHASE 3 — Model Variants**

```
BASE MODEL: ResNet encoder + UNet style decoder
Output layer: 3 channels (XYZ normal vector per pixel)
Loss function: cosine similarity loss (angular loss)
Optimiser: Adam, lr=1e-4
Epochs: 50 (with early stopping)
Batch size: 8 (fits Colab free tier)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AXIS 1 — Encoder Depth (3 models)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Model A1: ResNet-50  + UNet decoder + BatchNorm
Model A2: ResNet-101 + UNet decoder + BatchNorm  
Model A3: ResNet-152 + UNet decoder + BatchNorm

Everything identical except encoder depth.
All start from their respective ImageNet pretrained weights.
Decoder randomly initialised in all three.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AXIS 2 — Skip Connections (3 models)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Model B1: ResNet-50 + Full skip UNet decoder (all 4 skip connections)
Model B2: ResNet-50 + Partial skip decoder (only Stage 3+4 skip connections)
Model B3: ResNet-50 + No skip decoder (bottleneck only, no skip connections)

Everything identical except skip connection density.
All start from same ResNet-50 ImageNet pretrained weights.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AXIS 3 — Normalization Type (3 models)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Model C1: ResNet-50 + UNet decoder + BatchNorm   (default)
Model C2: ResNet-50 + UNet decoder + InstanceNorm
Model C3: ResNet-50 + UNet decoder + GroupNorm

Everything identical except normalisation layer type.
All start from ResNet-50 ImageNet pretrained weights.
Note: Model A1 and Model C1 are the same model — shared baseline.
Total unique training runs: 8 (not 9, since A1=C1)
```

---

**PHASE 4 — Training Protocol**

```
For each of the 8 model variants:

Step 1: Load ImageNet pretrained encoder weights
Step 2: Swap normalization layers if needed (Axis 3)
Step 3: Attach fresh randomly initialised decoder
Step 4: Fine-tune entire model on NYUv2 795 training images
Step 5: Validate on small held-out subset every 5 epochs
Step 6: Save best checkpoint to Drive

Checkpoint storage:
BASE/checkpoints/model_A1_resnet50_fullskip_bn/best.pth
BASE/checkpoints/model_A2_resnet101_fullskip_bn/best.pth
BASE/checkpoints/model_A3_resnet152_fullskip_bn/best.pth
BASE/checkpoints/model_B1_resnet50_fullskip_bn/best.pth  ← same as A1
BASE/checkpoints/model_B2_resnet50_partialskip_bn/best.pth
BASE/checkpoints/model_B3_resnet50_noskip_bn/best.pth
BASE/checkpoints/model_C2_resnet50_fullskip_in/best.pth
BASE/checkpoints/model_C3_resnet50_fullskip_gn/best.pth

Training log storage:
BASE/logs/model_A1_training_log.csv
(loss per epoch, validation metric per epoch)
```

---

**PHASE 5 — Evaluation Protocol**

```
For each trained model run evaluation in 5 conditions:

Condition 0: Clean input (baseline)
Condition 1: Underexposure
Condition 2: Overexposure
Condition 3: Shadows
Condition 4: Day-night colour shift

For each condition:
1. Load 654 test RGB images
2. Apply degradation (skip for clean condition)
3. Run model inference → predicted normal map per image
4. Load corresponding Wang GT normal map
5. Compute metrics

Metrics computed per image then averaged:
├── Mean Angular Error (MAE) — primary metric
│   angle between predicted and GT normal vectors
│   lower is better, measured in degrees
├── δ1 — % pixels with angular error < 11.25°
├── δ2 — % pixels with angular error < 22.5°
└── δ3 — % pixels with angular error < 30°

Additionally computed separately for:
├── Planar regions (flat surfaces — walls, floors, ceilings)
└── Boundary regions (edges where surfaces meet)
This decomposition is your key contribution over standard benchmarks.

Results storage:
BASE/results/clean/model_A1_clean_results.csv
BASE/results/degraded/underexposure/model_A1_underexposure_results.csv
... and so on for all models and all conditions
```

---

**PHASE 6 — Analysis**

```
After all evaluation runs complete:

Analysis 1 — Per axis robustness comparison
Group models by axis and compare MAE degradation:

Axis 1 (Depth):
  Clean MAE:        A1=x, A2=y, A3=z
  Underexposure MAE: A1=x, A2=y, A3=z
  Robustness drop:  how much MAE increases per model

Axis 2 (Skip Connections):
  Same structure as above for B1, B2, B3

Axis 3 (Normalization):
  Same structure for C1, C2, C3

Analysis 2 — Planar vs Boundary degradation
For each model compare how much more boundaries degrade
versus planar regions under each stress condition

Analysis 3 — Degradation type comparison
Which of the 4 degradations causes most damage overall
and does the ranking change across architectures

Output: summary tables + visualisations saved to BASE/benchmarks/
```

---

**PHASE 7 — Secondary Evaluation on iBims-1 (if time allows)**

```
Take all 8 trained models (no retraining)
Run same 5 condition evaluation on iBims-1 100 images
Compare if findings from NYUv2 hold on different dataset
This section goes in paper as generalisation analysis
```

---

**COMPUTE REALITY CHECK FOR COLAB FREE TIER**

```
Each training run:    ~3-5 hours on Colab T4 GPU
8 training runs:      ~24-40 hours total
Each evaluation run:  ~30 minutes
8 models × 5 conditions = 40 evaluation runs × 30min = ~20 hours

Total compute needed: ~45-60 hours GPU time

Colab free tier gives roughly 4-6 hours per session.
You will need ~10-15 Colab sessions minimum.

Recommendation: get Colab Pro when you reach training phase.
It gives longer sessions, faster GPUs, more RAM.
For now free tier is fine for data prep and single image testing.
```

---

**IMMEDIATE NEXT STEPS (what to do right now)**

```
Today:
1. Run Cell 9 — download Wang/Ladicky GT normals       ← you are here
2. Run Cell 10 — visualise GT normal vs Cell 8 normal
3. See the quality difference with your own eyes

This week:
4. Build PyTorch Dataset class for NYUv2
5. Build one model variant (Model A1 — ResNet50 + UNet + BN)
6. Run training on NYUv2 795 images
7. Evaluate on 654 test images clean condition
8. Apply one degradation and re-evaluate
9. See your first real robustness result

After that:
10. Build remaining 7 model variants
11. Run full evaluation pipeline
12. Analyse results across axes
```

Save this. This is your entire project roadmap. Every decision from here connects back to this pipeline. Run Cell 9 now.

## 2026-03-19: Evaluation Benchmarks - Ladicky et al. (2014) and Wang et al. (2015)

**Relevant Resources:**
- **Paper:** [Wang et al. (2015) - Designing Deep Networks for Surface Normal Estimation](https://cs.nyu.edu/~fouhey/2015/deep3d/deep3d.pdf)
- **Code/Implementation:** [Marr Revisited (Aayush Bansal)](https://github.com/aayushbansal/MarrRevisited)
- **Data (Ground Truth Normals):** [NYUv2 Precomputed Normals (Zip)](https://tilted-image-normal.s3.amazonaws.com/nyu-normal.zip)

This section traces the origins of the standardized evaluation framework used in our project. Our academic rigor relies on following these established protocols.

### 1. Ladicky et al. (2014) — Ground Truth Preparation

Ladicky et al. addressed the fundamental noise issues in the raw Kinect depth data of NYUv2. Directly converting raw depth results in unreliable ground truth.

*   **The Work:** They computed clean, per-pixel surface normal maps for all 1449 NYUv2 images.
*   **The Method:** Fitting local planes to 3D point neighborhoods (Method 3) with systematic outlier rejection and smoothing.
*   **The Output:** 1449 PNG files where each pixel encodes a 3D unit vector via the standard XYZ-to-RGB mapping.
*   **Relevance:** These PNGs are the **Ground Truth** for our project. Without this preprocessing, NYUv2 depth would be too noisy for meaningful research.

### 2. Wang et al. (2015) — Standardized Infrastructure

Wang et al.'s paper *"Designing Deep Networks for Surface Normal Estimation"* established the benchmarking infrastructure the field uses today.

*   **Contribution 1: Fixed Test Split.** They defined the permanent 654-image test split from NYUv2, making direct comparisons between research papers possible.
*   **Contribution 2: Standard Metrics.** They established the five specific metrics (MAE, Med AE, and three $\delta$ thresholds) that allow researchers to evaluate models on a level playing field.
*   **Contribution 3: Baseline CNN.** They provided the first deep CNN baseline for surface normal estimation.

### 3. The Five Wang et al. Metrics — Technical Definitions

| Metric | Definition | Intuition |
| :--- | :--- | :--- |
| **MAE** | **Mean Angular Error** | Average angle (degrees) between predicted and GT vectors. Lower is better. |
| **Median AE** | **Median Angular Error** | Middle error value when sorted. Resistant to catastrophic local outliers. |
| **$\delta_1$** | **% pixels < 11.25°** | Measures high-precision predictions. Higher is better. |
| **$\delta_2$** | **% pixels < 22.5°** | Measures "acceptable" or roughly correct predictions. Higher is better. |
| **$\delta_3$** | **% pixels < 30.0°** | Measures broad directional correctness. Higher is better. |

### 4. Integration into Our Pipeline (Phase 5)

Our experiment converts model predictions into comparable numbers using these metrics. For each of the 8 model variants under 5 conditions, we generate a comprehensive results table.

*   **Primary Analysis:** The **Robustness Drop** ($MAE_{degraded} - MAE_{clean}$). A smaller drop indicates higher architectural robustness.
*   **Failure Mode Analysis:** Comparing $\delta_1$ vs. $\delta_3$ degradation. If $\delta_1$ drops but $\delta_3$ stays high, the model is losing precision. If both drop, the model is failing catastrophically.
*   **Planar vs. Boundary:** We apply these metrics separately to flat vs. edge masks—our specific analytical contribution.

**Conclusion:** By citing Wang et al. and Ladicky et al., we ensure our results are directly comparable to every SOTA paper in the field.
