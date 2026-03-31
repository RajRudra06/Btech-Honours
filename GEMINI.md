# Project: BTech Honours - AI Path-Finding Simulation

This directory appears to be a collection of academic materials related to a B.Tech Honours program, with a specific focus on an "AI Path-Finding Simulation" project.

## Project Overview

The core of this project is a web-based interactive simulation comparing three path-finding algorithms: Breadth-First Search (BFS), Depth-First Search (DFS), and A\* Search. The simulation allows users to set start and goal points, add obstacles on a grid, and then visualize how each algorithm finds a path. It also provides performance statistics for each algorithm (path length, nodes explored, max memory, and time).

**Technologies Used:**

*   **Frontend:** HTML, CSS, JavaScript (all embedded within a single `Path_Finding_Simulation_Assignment.html` file).

## Building and Running

This project does not require a complex build process. It is a client-side web application.

*   **To Run:** Open the `Path_Finding_Simulation_Assignment.html` file in any modern web browser.

## Development Conventions

*   The project consists of a single HTML file containing all the necessary HTML structure, CSS styling, and JavaScript logic.
*   JavaScript functions are used to handle user interactions, render the grid, implement the path-finding algorithms, and display results.
*   Basic interactive elements are present for setting start/goal, adding obstacles, running simulations, and viewing algorithm analysis.

## Key Files:

*   `Path_Finding_Simulation_Assignment.html`: The main file containing the interactive path-finding simulation.
*   `Assignment_Path_Finding_Finding_Simulation_HTML_CODE.txt`: A text version of the HTML code, likely for submission or backup.
*   `AI_ML_College_Syllabus.docx`, `~$_ML_College_Syllabus.docx`, `Artificial Intelligence_ Concepts and Techniques.pdf`, `IT CCE 5th Sem NPTEL Assignment July Dec 2025.docx`: Academic documents related to AI/ML courses and assignments.
*   `BTech Honors Supervisor List - July-Dec 2025.xlsx`: A spreadsheet likely containing supervisor information for the honours program.
*   `Coursera_Research_Course_Certificate.pdf`: A certificate, possibly related to research courses.
*   `Rudra_Rajpurohit_230911570_Assignment_Report.docx`: An assignment report, likely detailing the path-finding simulation or another academic task.
*   `Selections.txt`, `SubScalePayload2026.txt`: Other miscellaneous text files whose purpose is not immediately clear from their names.

## Usage

This directory serves as a repository for academic work, including a functional web-based AI path-finding simulation and related course materials. The HTML file can be directly opened in a browser to interact with the simulation.

## Finalized Research Direction

### RESEARCH PROBLEM STATEMENT — FINAL DETAILED VERSION

**Title:**
Geometric Perception Under Photometric Stress: How Architectural Choices in Surface Normal Estimation Respond to Structured Lighting and Colour Degradations

**Field:**
Artificial Intelligence / Machine Learning → Computer Vision → 3D Low-Level Vision → Surface Normal Estimation

**Research Path:**
This is a field-driven, understanding-first study (Path B). The task — surface normal estimation — is used as a controlled testbed to study model behaviour, architectural properties, and limitations. The task itself is not the contribution. The contribution is the analysis and findings about how architectural design mediates robustness to photometric stress. Findings are intended to be generalisable across 3D vision tasks.

**Aim:**
To systematically study how architectural design choices in surface normal estimation models determine their sensitivity or robustness to photometric stress — specifically structured lighting and colour degradations that change image appearance without altering the underlying scene geometry. The goal is to identify which architectural components (encoder depth, skip-connection strategies, normalization layers) drive photometric invariance or sensitivity in geometric prediction tasks.

**The Core Task — Surface Normal Estimation:**
Surface normal estimation is a 3D low-level vision task where a model takes a single 2D RGB image as input and predicts, for every pixel, the 3D orientation of the surface at that point — essentially which direction that surface is "facing" in 3D space. The output is a dense per-pixel map of continuous 3D unit vectors. It is not a classification task. There are no discrete labels. The model must infer geometry purely from visual cues in a 2D image.

This task is chosen because it creates a uniquely clean scientific setup: geometry and appearance are theoretically independent. A wall does not change its orientation because the lighting changes. A floor does not tilt because it is underexposed. This means that any degradation in surface normal predictions under photometric stress is entirely attributable to the model's failure to separate geometry from appearance — not to any actual change in the scene. This makes surface normal estimation an ideal testbed for studying photometric robustness in geometric prediction.

**The Stress Condition — Structured Photometric Degradations:**
The models are trained on clean, well-lit images and evaluated at inference time only on photometrically degraded inputs. No retraining or fine-tuning is done on degraded data. This is a strict train-on-clean, test-on-degraded protocol.

The degradation taxonomy consists of four structured photometric stress conditions, all of which alter image appearance while leaving scene geometry completely intact:
- **Under-exposure** — the image is darkened, reducing visibility of surface detail especially in shadow regions
- **Over-exposure** — the image is brightened, washing out highlight regions and reducing surface texture contrast
- **Shadows** — localised lighting changes are introduced, creating sharp or soft shadow boundaries across surfaces
- **Day-night / colour temperature shifts** — the overall colour balance and white balance of the image is shifted, simulating different times of day or artificial lighting conditions

These four conditions are deliberately chosen because they are appearance-only degradations. Blur, noise, and resolution loss are explicitly excluded because they destroy spatial and structural information that the model needs to estimate geometry — including them would change the scientific question from "does geometry hold up when appearance changes" to "does geometry hold up when information is destroyed," which is a different and less focused research question.

**Research Lenses:**

**Primary lens — Architecture Comparison:**
The study compares surface normal estimation architectures that differ along three specific axes:
- Encoder depth — shallow versus deep feature extraction hierarchies
- Skip-connection strategies — how much low-level spatial detail is passed from encoder to decoder
- Normalization layer type — batch normalisation, instance normalisation, and layer normalisation, which are hypothesised to behave differently under photometric stress because they operate over different statistical scopes

The hypothesis is that deeper, more abstract encoders better separate geometric features from appearance features, making them more photometrically invariant. Skip connections that pass low-level appearance information may reintroduce photometric sensitivity even when the deep encoder has abstracted it away. Normalization layers are a specific mechanism of interest because they explicitly standardise feature statistics, and their scope (batch-level vs instance-level vs layer-level) may determine how well they normalise out photometric variation.

**Supporting lens — Robustness Analysis:**
Robustness is measured as the degradation in surface normal prediction quality when moving from clean to photometrically degraded inputs, under each of the four stress conditions independently. Standard surface normal estimation metrics are used — mean angular error, percentage of pixels within angular thresholds (δ1, δ2, δ3). Robustness is reported both globally across the full image and locally, separating planar regions from object boundaries.

**Failure Analysis Focus:**
The failure analysis has a specific spatial structure. Two regions are analysed separately:
- **Planar regions** — large flat surfaces (walls, floors, ceilings) where geometry is simple and consistent. These should in theory be the most robust to photometric stress because there is no competing geometric detail
- **Object boundaries** — edges where one surface meets another, where geometry changes sharply and appearance cues (colour, texture, contrast) are most entangled with geometric cues

The hypothesis is that object boundaries will degrade significantly more under photometric stress than planar regions, because boundaries rely more heavily on appearance contrast to infer the geometric transition. The secondary hypothesis is that architectures with stronger low-level skip connections will show greater boundary degradation under photometric stress, because they are more dependent on appearance-level edge cues.

**What the Study is NOT doing:**
- It is not retraining models on degraded data or studying domain adaptation
- It is not proposing a new architecture or a new training method
- It is not doing multi-task learning combining segmentation and surface normals
- It is not studying blur, noise, or resolution degradation — those are geometric/quality degradations and belong to a different research question
- It is not comparing CNNs vs Transformers as the primary axis — the comparison is within encoder-decoder families varying in depth, skip connections, and normalization

**Expected Contributions:**
1. A structured empirical analysis of how photometric stress affects surface normal estimation across architectures — currently underexplored in literature compared to semantic segmentation robustness
2. Evidence for or against the hypothesis that encoder depth provides photometric invariance in geometric prediction tasks
3. Evidence for or against the normalization layer hypothesis — that instance norm or layer norm provide better photometric invariance than batch norm in this setting
4. A spatially decomposed failure analysis separating planar region degradation from boundary degradation under photometric stress
5. A reusable degradation evaluation protocol for photometric robustness in 3D low-level vision tasks

**Datasets (indicative):**
Standard surface normal estimation benchmarks — NYUv2, ScanNet, or iBims-1 — with photometric degradations applied synthetically at inference time using standard image processing transformations. Ground truth surface normals remain unchanged since the degradations do not alter scene geometry.

## FOUR CRITICAL Qs

1.  **Identifying Suitability:** Identifying whether a model can be used for surface normal estimation. If yes, determining what needs to be surgically removed or modified to adapt it for that task.
2.  **Confirming Axes:** Confirming and reading about the major architectural axes (e.g., encoder depth, skip connections).
3.  **Studying Models & Categories:** Studying different models and their intricacies, mapping them to the identified axes, and creating categories based on these architectural choices.
4.  **Verifying Isolation:** Making sure that models within an axis actually follow the process of isolating that axis. (i.e., checking if they truly test one variable at a time or if confounding factors make isolation impossible).

## Literature Survey Strategy
...
**The four questions you need to answer through literature survey:**

1. Which existing models work for surface normal estimation, and what architectural components need to be modified or removed to adapt a general CNN for this task
2. Which of the 10 axes are actually present and meaningfully varied across real surface normal estimation models in literature
3. What are the major models in this space, how do they differ architecturally, and can they be grouped into categories across the axes
4. For each axis you select, do real existing models actually isolate that axis cleanly enough to draw conclusions, or do confounds make isolation impossible

---

**Literature Survey Process — combining everything into one clean workflow:**

**Day 1 — Map the field in 2 hours, no reading**

Go to paperswithcode.com/task/surface-normal-estimation. Do not read anything yet. Just look at the leaderboard. Take the top 10 models by NYUv2 benchmark score. For each one spend literally 2 minutes only on the architecture diagram in the paper. Fill only these four columns:

| Paper | Encoder | Decoder | Code Available |

That is your field map. 10 rows, 2 minutes each, 20 minutes total. You now know what encoders and decoders the field actually uses in practice, which directly starts answering questions 1 and 3.

---

**Day 2 — Go deeper on 5 papers, answer questions 1 and 2**

From your Day 1 table pick the 5 papers that have code available. For each one read in this exact order and nothing else:

First the abstract — one paragraph, confirm it is CNN based and evaluated on NYUv2

Then the method section architecture diagram only — identify encoder, decoder, skip connections, normalization type if mentioned

Then the experiments table — what metrics, what dataset, what baselines

Do not read introduction, related work, or conclusion. Add three more columns to your table:

| Skip Connections | Norm Type | Metric Used |

After 5 papers you will have naturally seen which axes appear repeatedly in real models — that answers question 2. You will also have seen at least one paper describe what they changed from a base classification model to make it work for surface normals — that answers question 1.

---

**Day 3 — Google Scholar search for robustness, answer question 4**

Go to scholar.google.com and run these three searches one by one, filter to 2018-2024, look only at papers above 50 citations:

Search 1: "surface normal estimation robustness"
Search 2: "photometric robustness dense prediction"
Search 3: "image degradation surface normal CNN"

For each result that looks relevant spend 5 minutes only on abstract and experiments section. You are looking for two things — has anyone already done photometric robustness on surface normals (if yes, note what they did so you can differentiate), and does any paper do controlled axis isolation (if yes, that answers question 4 directly by showing you how it is done in practice).

Add one final column to your table:

| Robustness Study Exists |

---

**Day 4 — Connected Papers, close the gaps**

Take the single most relevant paper from your table — the one closest to your research question. Go to connectedpapers.com and paste its title. You will see a visual graph of related papers. Look at the papers connected to it that you have not read yet. Spend 10 minutes identifying any you missed that are relevant to axes or robustness. Read those abstracts only.

By end of Day 4 your table should have 8-10 rows filled across all columns. At that point your four questions are answered:

Question 1 is answered because you have seen multiple papers describe their architecture adaptations from encoder-only to encoder-decoder for surface normals

Question 2 is answered because your table shows which axes naturally vary across real models

Question 3 is answered because your table is literally the model categories grouped by encoder and decoder type

Question 4 is answered because the robustness search either found papers doing controlled isolation (showing you it is possible and how) or found nothing (confirming your topic is underexplored and that perfect isolation is an open problem you acknowledge in limitations)

---

**The one rule throughout all four days**

## THEORY.md Workflow

When the user provides a block of text and asks to "summarize and put in THEORY.md":

1.  **Refine & Contextualize:** Summarize the provided text, but critically, **expand and frame it** specifically within the context of the "Geometric Perception Under Photometric Stress" project. Connect general concepts (like CNNs or Surface Normals) directly to the project's hypotheses (e.g., "U-Net copies shadows," "ResNet ignores lighting").
2.  **No Omissions:** DO NOT shorten the content by removing technical details, mathematical formulas, or specific step-by-step examples. The goal is coherence and project-alignment, not brevity. Ensure all provided math (e.g., coordinate transforms, normalization steps) is preserved.
3.  **Format:**
    *   Create a new H2 header with the date and a descriptive title (e.g., `## 2026-03-19: Topic Name`).
    *   Use clear subsections, bullet points, and bold text for readability.
4.  **Append:** Add this new section to the end of `THEORY.md`.