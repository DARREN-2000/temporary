# Mermaid Diagram Alternatives

These are Mermaid versions of the thesis diagrams. You can render them at [mermaid.live](https://mermaid.live) and export as PNG/PDF, then include with `\includegraphics` in LaTeX.

**Notation used (matching the professor's requirements):**
- **Rectangular boxes** `[...]` = Data artifacts
- **Rounded boxes** `(...)` = Processing steps
- No two data boxes are directly connected
- No two processing boxes are directly connected
- Top-down (TD) layout for maximum size

---

## Figure 1.2 — The Fuzzing Process

```mermaid
flowchart TD
    %% ===== DATA (rectangular) =====
    SC["Source Code"]
    DK["Domain Knowledge"]
    FD["Fuzz Driver"]
    SI["Seed Inputs"]
    CD["Crash Data"]
    CV["Coverage Data"]
    VR["Vulnerability Report"]
    MI["Mutated Inputs"]

    %% ===== PROCESSING (rounded / stadium) =====
    MFC(["<b>Manual Fuzz Driver Creation</b><br/><i>bottleneck</i>"])
    FE(["Fuzzing Engine<br/><i>libFuzzer / AFL++</i>"])
    VRG(["Vulnerability Report<br/>Generation"])
    IM(["Input Mutation"])

    %% ===== FLOW =====
    SC --> MFC
    DK --> MFC
    MFC --> FD
    FD --> FE
    SI --> FE
    FE --> CD
    FE --> CV
    CD --> VRG
    CV --> IM
    VRG --> VR
    IM --> MI
    MI -->|feedback loop| FE

    %% ===== STYLING =====
    style SC fill:#dce6f1,stroke:#333,stroke-width:2px
    style DK fill:#dce6f1,stroke:#333,stroke-width:2px
    style FD fill:#dce6f1,stroke:#333,stroke-width:2px
    style SI fill:#dce6f1,stroke:#333,stroke-width:2px
    style CD fill:#dce6f1,stroke:#333,stroke-width:2px
    style CV fill:#dce6f1,stroke:#333,stroke-width:2px
    style VR fill:#dce6f1,stroke:#333,stroke-width:2px
    style MI fill:#dce6f1,stroke:#333,stroke-width:2px

    style MFC fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style FE fill:#fde8cd,stroke:#333,stroke-width:2px
    style VRG fill:#fde8cd,stroke:#333,stroke-width:2px
    style IM fill:#fde8cd,stroke:#333,stroke-width:2px
```

**Caption:** The Fuzzing Process. Data artifacts are shown in rectangular boxes; processing steps are shown in rounded boxes. The manual creation of fuzz drivers (highlighted in red) is the bottleneck this thesis aims to automate.

---

## Figure 3.1 — Technical Architecture (LLM-based Fuzz Driver Generation Pipeline)

```mermaid
flowchart TD
    %% ============================================================
    %%  INPUT LAYER
    %% ============================================================
    subgraph InputLayer ["<b>Input Layer (§ 3.2.1)</b>"]
        direction TB
        SRC["Source Code<br/><i>C/C++ project</i>"]
        HDR["Public Header Files<br/><i>.h / .hpp</i>"]
        CTX(["<b>Automated Context<br/>Extraction</b><br/><i>cifuzz spark</i>"]):::newproc
        API["Extracted API Context<br/><i>signatures, types, classes</i>"]:::newdata

        SRC --> CTX
        HDR --> CTX
        CTX --> API
    end

    %% ============================================================
    %%  GENERATION LAYER
    %% ============================================================
    subgraph GenLayer ["<b>Generation Layer (§ 3.2.2)</b>"]
        direction TB
        FI["Fuzzing Instructions<br/><i>libFuzzer harness rules,<br/>safety constraints</i>"]:::newdata
        PC(["<b>Prompt Construction</b><br/><i>API context + instructions</i>"]):::newproc
        PR["Assembled Prompt"]:::newdata
        LLM["LLM Model<br/><i>local via Ollama /<br/>cloud via Azure OpenAI</i>"]
        GEN(["<b>LLM-Based Driver<br/>Generation</b>"]):::newproc
        GFD["Generated Fuzz Driver<br/><i>C++ source code</i>"]:::newdata

        FI --> PC
        PC --> PR
        PR --> GEN
        LLM --> GEN
        GEN --> GFD
    end

    %% ============================================================
    %%  EXECUTION LAYER
    %% ============================================================
    subgraph ExecLayer ["<b>Execution Layer (§ 3.2.3)</b>"]
        direction TB
        CMP(["<b>Compilation</b><br/><i>Clang + ASan/UBSan</i>"])
        BIN["Instrumented Binary"]
        SEED["Seed Inputs"]
        FEX(["<b>Fuzzing Execution</b><br/><i>libFuzzer</i>"])
        CRASH["Crash Reports"]
        COV["Coverage Data"]
        CEVAL(["<b>Coverage Evaluation</b><br/><i>quality gate</i>"]):::newproc
        CREP["Coverage Report<br/><i>pass / fail</i>"]:::newdata

        CMP --> BIN
        BIN --> FEX
        SEED --> FEX
        FEX --> CRASH
        FEX --> COV
        COV --> CEVAL
        CEVAL --> CREP
    end

    %% ============================================================
    %%  CROSS-LAYER CONNECTIONS
    %% ============================================================
    API --> PC
    GFD --> CMP

    %% ============================================================
    %%  STYLES
    %% ============================================================

    %% Existing data — light blue rectangles
    style SRC fill:#dce6f1,stroke:#333,stroke-width:2px
    style HDR fill:#dce6f1,stroke:#333,stroke-width:2px
    style LLM fill:#dce6f1,stroke:#333,stroke-width:2px
    style BIN fill:#dce6f1,stroke:#333,stroke-width:2px
    style SEED fill:#dce6f1,stroke:#333,stroke-width:2px
    style CRASH fill:#dce6f1,stroke:#333,stroke-width:2px
    style COV fill:#dce6f1,stroke:#333,stroke-width:2px

    %% Existing processing — light orange rounded
    style CMP fill:#fde8cd,stroke:#333,stroke-width:2px
    style FEX fill:#fde8cd,stroke:#333,stroke-width:2px

    %% New data (our contribution) — light green rectangles
    classDef newdata fill:#d4edda,stroke:#155724,stroke-width:2px
    %% New processing (our contribution) — light yellow rounded
    classDef newproc fill:#fff3cd,stroke:#856404,stroke-width:2px

    %% Layer subgraph styling
    style InputLayer fill:#f0f4ff,stroke:#668,stroke-width:1px,stroke-dasharray:5
    style GenLayer fill:#fffde6,stroke:#886,stroke-width:1px,stroke-dasharray:5
    style ExecLayer fill:#fff5ee,stroke:#864,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Technical Architecture of our LLM-based fuzz driver generation pipeline. The three layers are marked with dashed borders. Data artifacts are shown in rectangular boxes; processing steps in boxes with rounded corners. Green/yellow highlighted elements denote new contributions of this work with respect to the state of the art (cf. Figure 1.2).

**Legend:**
| Style | Meaning |
|-------|---------|
| Blue rectangle | Existing data |
| Green rectangle | New data (our contribution) |
| Orange rounded | Existing process |
| Yellow rounded | New process (our contribution) |

---

## How to Use These Mermaid Diagrams

### Option A: Render online and export
1. Go to [mermaid.live](https://mermaid.live)
2. Paste the Mermaid code
3. Export as PNG (high resolution) or SVG
4. Save to `bilder/` folder
5. In LaTeX: `\includegraphics[width=\textwidth]{bilder/fuzzing_mermaid.png}`

### Option B: Use the `mermaid-filter` for Pandoc
If you convert through Pandoc, install `mermaid-filter` and it auto-renders.

### Option C: Keep using the TikZ versions already in the .tex files
The TikZ versions are already embedded in the LaTeX files and compile directly. They use the same notation (rectangular = data, rounded = process) and follow the top-down layout.
