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

## RESEARCH Analysis

**Academic Context and Objective:**
Rudra is pursuing a minor degree in AI/ML requiring a research paper. The goal is empirical research aligned with academic standards, focusing on insightful analysis and controlled experimental design.

**Research Hierarchy:**
*   **Field:** Computer Vision
*   **Task:** Semantic Segmentation
*   **Main Research Lens:** Architecture comparison
*   **Supporting Lens:** Robustness to real-world image degradation

**Problem Statement:**
"An Architecture-Based Robustness Analysis of CNN Models for Semantic Segmentation under Real-World Image Degradation."

**Important Clarifications:**
*   No claim of proposing a new architecture or state-of-the-art performance.
*   Contribution is empirical understanding, not novelty for novelty’s sake.
*   Architecture is the independent variable; robustness behavior is the dependent variable.

**What "Architecture Comparison" Means:**
Comparison is structured around architectural design philosophies:
*   Encoder-decoder structure
*   Strength of skip connections
*   Multi-scale context aggregation
*   Lightweight vs. heavyweight encoders

**What "Robustness" Means:**
Performance stability when input images deviate from ideal, clean training conditions. This is *not* adversarial robustness.
Realistic degradations considered (scope restricted):
*   Gaussian noise (sensor noise)
*   Blur (motion or defocus)
*   Resolution downscaling (distance / compression)
Critical Experimental Rule: All models are trained *only* on clean data. Robustness is evaluated *only* at inference time. No retraining or adaptation is performed to isolate inherent architectural robustness.

**Model Architectures Selected (with rationale):**
A small, conceptually diverse set of architectures, each representing a distinct inductive bias:
1.  **U-Net with ResNet encoder:** (Boundary-aware architectures) Strong encoder-decoder design, explicit skip connections, emphasizes spatial detail preservation.
2.  **DeepLabV3+ with ResNet backbone:** (Context-heavy architectures) Atrous (dilated) convolutions, multi-scale context aggregation, emphasizes semantic consistency over fine detail.
3.  **(Optional) U-Net with MobileNet encoder:** (Efficiency-biased architectures) Lightweight encoder, lower capacity, higher efficiency.

**Experimental Methodology (High-Level but Precise):**
1.  Select one dataset (urban scenes recommended).
2.  Train all models using:
    *   Identical data splits
    *   Identical training protocol
3.  Evaluate on:
    *   Clean validation data
    *   Degraded variants of the same data
4.  Measure:
    *   Mean IoU degradation trends
    *   Class-wise failure behavior
    *   Boundary erosion effects
5.  Perform:
    *   Quantitative analysis
    *   Qualitative failure case inspection
6.  Draw architecture → behavior conclusions.

**Nature of the Contribution:**
The paper will contribute:
*   A controlled comparison of segmentation architectures.
*   A systematic robustness evaluation under realistic degradations.
*   Analytical insights linking architectural choices to failure modes.

It deliberately avoids overclaiming novelty, excessive scope, or uncontrolled benchmarking.

**Why this is not a toy project:**
The framing shifts from "Which model performs better?" to "Why do certain architectural design choices lead to greater robustness under real-world degradation?", elevating it to research.

**Current Status / Pending:**
All conceptual decisions are locked except one:
*   Dataset category selection (Urban / outdoor scenes recommended; Medical imaging valid but requires stricter justification).
Once the dataset is fixed, remaining steps are mechanical: Dataset selection, Final model instantiation, Paper outline, Timeline execution.

**One-line summary:**
This work is a semantic segmentation study that compares CNN architectures and analyzes how their design choices affect robustness to real-world image degradation, using controlled experiments and analysis-driven conclusions.

## Latest Research Discussion & Strategy

This session involved several key discussions and clarifications to solidify the research direction for Rudra's minor degree paper.

### 1. Evaluation of Research Methodology

*   **Hierarchy Validation:** Rudra's structured approach (Field → Task → Research Lenses) was affirmed as an excellent, systematic methodology for topic selection, promoting clear problem definition, justified choices, and coherent causal reasoning. It is ideal for academic rigor and managing scope.
*   **Correctness of Criteria:** The criteria used for task selection (Analytical depth, Experimental noise, Risk of inconclusive results) were confirmed to be fundamentally sound and sophisticated for designing research that yields meaningful and interpretable results.

### 2. ML vs. Deep Learning Distinction

*   **Clarification:** The fundamental difference was clarified: Machine Learning (ML) is the broader umbrella of algorithms that learn from data, while Deep Learning (DL) is a subset of ML that uses deep neural networks.
*   **Key Differentiator:** The primary distinction lies in DL's ability for **automatic feature learning** directly from raw data, whereas traditional ML often relies on manual feature engineering.
*   **Learning Paradigms:** It was emphasized that both ML and DL can encompass supervised, unsupervised, reinforcement learning, and other paradigms.

### 3. Research Paper Dilemma: Path A vs. Path B

*   **Dilemma Defined:** The choice between "Path A: Solve a concrete problem using the most appropriate model" and "Path B: Field-driven / phenomenon-driven research" was discussed.

*   **Path A — Problem-driven research:**
    *   **Goal:** Solve a specific real-world problem using the most suitable model (e.g., Random Forest, XGBoost, CNN, hybrid ML/DL).
    *   **Characteristics:** Focus on application, practical solution, often aiming for optimal performance.

*   **Path B — Field-driven / phenomenon-driven research (Understanding-first):**
    *   **Goal:** Start from a field or task and study it to understand model behavior, properties, and limitations, not to solve a specific application.
    *   **Characteristics:**
        *   Task is a testbed, not the primary goal.
        *   Goal is understanding: behavior, failure modes, inductive biases, trade-offs.
        *   Results are generalizable.
        *   Application comes later as implication, not claim.
    *   **Research Hierarchy for Path B:**
         └── Chosen field / task (e.g., semantic segmentation)
              └── Research lenses
                   ├── Architecture comparison
                   ├── Robustness analysis
                   ├── Generalisation study
                   ├── Efficiency trade-offs
                   └── Data-centric analysis

*   **Recommendation for Path B:** Path B (field-driven research) was strongly recommended as the superior choice for Rudra's goals, as it aligns with his stated preference for analysis-driven research, leverages his conceptual strengths, offers higher academic value, and provides manageable scope for a first paper.
*   **"Best of Both Worlds" Approach:** Rudra's proposed "best of both worlds" approach was fully endorsed. This strategy keeps the paper's core firmly in Path B (architectural analysis) while using real-world constraints (e.g., blur, noise) as motivating factors. The discussion section then maps these insights to practical scenarios, preserving analytical rigor while enhancing practical relevance.

### 4. Guidance on Exploring Other AI/ML Fields

*   **Strategic Exploration:** It was agreed that exploring other AI/ML domains (NLP, RL, etc.) is valuable for broadening perspective, identifying future research, and cross-pollination of ideas, but it must be done strategically to avoid derailing the current paper.
*   **Methodology for Exploration:** Detailed step-by-step instructions were provided for using Google Scholar:
    1.  **Initial Search:** Use broad domain + "survey" or "review" (e.g., "natural language processing survey").
    2.  **Filter & Evaluate:** Apply recent time filters (e.g., "Since 2023"), scan titles/abstracts for scope, and prioritize papers with high citation counts.
    3.  **Read Survey Papers:** Focus on understanding core concepts, major sub-areas, landmark papers, evaluation metrics, and open challenges.
    4.  **Review Foundational Papers:** Briefly review highly cited papers mentioned in surveys (Abstract, Intro, Conclusion) for high-level contributions.
*   **Key Caveats:** Emphasized timeboxing efforts, resisting rabbit holes, and maintaining focus on the current research paper as the primary objective.

### 5. Unbiased Comparative Analysis of Computer Vision Tasks

*   **Objective Re-evaluation:** An in-depth, unbiased comparative analysis was conducted for major CV tasks (Image Classification, Object Detection, Semantic Segmentation, Instance Segmentation, Image Generation) against Rudra's specific research criteria.
*   **Why Semantic Segmentation Dominates:** The analysis rigorously concluded that Semantic Segmentation is objectively the most suitable task for an architecture-centric robustness analysis due to:
    *   **Pixel-level Error Localization:** Provides the most granular and diagnostic error signals.
    *   **Direct Architectural Attribution:** Allows clear, causal links between architectural choices and observed robustness behaviors.
    *   **Low Experimental Variance:** Offers high reproducibility and reliable results.
    *   **Objective Ground Truth & Interpretable Metrics:** Provides unambiguous reference and intuitive performance measures (IoU).
    *   **Clear Degradation Mapping:** Real-world degradations have direct, measurable impacts on pixel-level outputs.
    *   **No Confounding Factors:** Reduces ambiguity compared to tasks with multiple interacting error sources (e.g., detection + segmentation in instance segmentation).
*   **Conclusion:** Semantic Segmentation uniquely satisfies all requirements for drawing strong, interpretable mappings between architectural designs and their robustness under degradation, making it the optimal choice for his paper.