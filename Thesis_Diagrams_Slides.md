# Thesis Diagrams — Slide-Ready Mermaid Code

Paste each block into **[mermaid.live](https://mermaid.live)**, then export as **PNG at 4× scale** (≈ 3840 × 2160 px) or SVG.
In PowerPoint set the image to fill the slide (`Format Picture → Size → Height: 19.05 cm, Width: 33.87 cm` for a 16:9 slide) and send it to the back.

> **Why these layouts fit a slide:**
> Every diagram uses top-down (`TD`) flow or a two-row wrapped layout so the rendered image is roughly landscape (wider than tall), matching a 16:9 PowerPoint slide. The two previously horizontal (`LR`) diagrams — Fig 1.2 and Fig 3.2 — have been restructured into slide-friendly shapes.

---

## Figure 1.1 — The Fuzzing Process

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 50, "rankSpacing": 60}} }%%
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
    MFC(["Manual Fuzz Driver Creation<br/><i>bottleneck</i>"])
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
    style SC  fill:#dce6f1,stroke:#333,stroke-width:2px
    style DK  fill:#dce6f1,stroke:#333,stroke-width:2px
    style FD  fill:#dce6f1,stroke:#333,stroke-width:2px
    style SI  fill:#dce6f1,stroke:#333,stroke-width:2px
    style CD  fill:#dce6f1,stroke:#333,stroke-width:2px
    style CV  fill:#dce6f1,stroke:#333,stroke-width:2px
    style VR  fill:#dce6f1,stroke:#333,stroke-width:2px
    style MI  fill:#dce6f1,stroke:#333,stroke-width:2px

    style MFC fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style FE  fill:#fde8cd,stroke:#333,stroke-width:2px
    style VRG fill:#fde8cd,stroke:#333,stroke-width:2px
    style IM  fill:#fde8cd,stroke:#333,stroke-width:2px
```

**Caption:** The Fuzzing Process. Rectangular boxes = data artifacts; rounded boxes = processing steps. Manual fuzz-driver creation (red) is the bottleneck this thesis automates.

---

## Figure 1.2 — The Automotive CI/CD/CT Pipeline

> **Layout note:** The original single left-to-right chain was ~1900 × 100 px — too flat for a slide.
> This version wraps the pipeline into **two rows inside a top-down parent**, producing a landscape image that fills a 16:9 slide.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 40, "rankSpacing": 55}} }%%
flowchart TD
    %% ── Row 1: Commit → … → Security Validation ─────────────────
    subgraph row1[" "]
        direction LR
        COMMIT["Commit /<br/>Source Tree"]
        BUILD(["Build<br/><i>CMake / Clang</i>"])
        ARTIFACT["Build Artifact<br/><i>Binary / Library</i>"]
        TEST(["Test Execution<br/><i>Unit / Integration</i>"])
        TRESULTS["Test Results"]
        SECVAL(["<b>Security Validation</b><br/><i>Fuzzing / SAST / DAST</i>"])

        COMMIT --> BUILD --> ARTIFACT --> TEST --> TRESULTS --> SECVAL
    end

    %% ── Row 2: Security Report → … → Telemetry ──────────────────
    subgraph row2[" "]
        direction LR
        SECREPORT["Security Report /<br/>Findings"]
        DEPLOY(["Deploy<br/><i>Staging / Production</i>"])
        RELEASE["Deployed Release"]
        MONITOR(["Monitor<br/><i>Telemetry / Logging</i>"])
        TELEMETRY["Telemetry /<br/>Incident Data"]

        SECREPORT --> DEPLOY --> RELEASE --> MONITOR --> TELEMETRY
    end

    %% ── Cross-row connections ─────────────────────────────────────
    SECVAL    --> SECREPORT
    TELEMETRY -.->|feedback loop| COMMIT

    %% ── Styling ──────────────────────────────────────────────────
    style COMMIT    fill:#dce6f1,stroke:#333,stroke-width:2px
    style ARTIFACT  fill:#dce6f1,stroke:#333,stroke-width:2px
    style TRESULTS  fill:#dce6f1,stroke:#333,stroke-width:2px
    style SECREPORT fill:#dce6f1,stroke:#333,stroke-width:2px
    style RELEASE   fill:#dce6f1,stroke:#333,stroke-width:2px
    style TELEMETRY fill:#dce6f1,stroke:#333,stroke-width:2px

    style BUILD   fill:#fde8cd,stroke:#333,stroke-width:2px
    style TEST    fill:#fde8cd,stroke:#333,stroke-width:2px
    style SECVAL  fill:#fff3cd,stroke:#856404,stroke-width:2px
    style DEPLOY  fill:#fde8cd,stroke:#333,stroke-width:2px
    style MONITOR fill:#fde8cd,stroke:#333,stroke-width:2px

    style row1 fill:transparent,stroke:#ccc,stroke-width:1px,stroke-dasharray:4
    style row2 fill:transparent,stroke:#ccc,stroke-width:1px,stroke-dasharray:4
```

**Caption:** The Automotive CI/CD/CT Pipeline. Security Validation (highlighted) is where fuzzing integrates. The dashed feedback arrow from Telemetry back to the source tree represents the continuous-improvement cycle.

---

## Figure 3.1 — Technical Architecture (LLM-based Fuzz Driver Generation Pipeline)

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "17px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 45, "rankSpacing": 55}} }%%
flowchart TD
    %% ── INPUT LAYER ──────────────────────────────────────────────
    subgraph InputLayer ["Input Layer"]
        direction TB
        SRC["Source Code<br/><i>C/C++ project</i>"]
        HDR["Public Header Files<br/><i>.h / .hpp</i>"]
        CTX(["Automated Context<br/>Extraction<br/><i>cifuzz spark</i>"])
        API["Extracted API Context<br/><i>signatures, types, classes</i>"]

        SRC --> CTX
        HDR --> CTX
        CTX --> API
    end

    %% ── GENERATION LAYER ─────────────────────────────────────────
    subgraph GenLayer ["Generation Layer"]
        direction TB
        FI["Fuzzing Instructions<br/><i>libFuzzer harness rules,<br/>safety constraints</i>"]
        PC(["Prompt Construction<br/><i>API context + instructions</i>"])
        PR["Assembled Prompt"]
        LLM["LLM Model<br/><i>local: Ollama / cloud: Azure OpenAI</i>"]
        GEN(["LLM-Based Driver<br/>Generation"])
        GFD["Generated Fuzz Driver<br/><i>C++ source code</i>"]

        FI  --> PC
        PC  --> PR
        PR  --> GEN
        LLM --> GEN
        GEN --> GFD
    end

    %% ── EXECUTION LAYER ──────────────────────────────────────────
    subgraph ExecLayer ["Execution Layer"]
        direction TB
        CMP(["Compilation<br/><i>Clang + ASan/UBSan</i>"])
        BIN["Instrumented Binary"]
        SEED["Seed Inputs"]
        FEX(["Fuzzing Execution<br/><i>libFuzzer</i>"])
        CRASH["Crash Reports"]
        COV["Coverage Data"]
        CEVAL(["Coverage Evaluation<br/><i>quality gate</i>"])
        CREP["Coverage Report<br/><i>pass / fail</i>"]

        CMP  --> BIN
        BIN  --> FEX
        SEED --> FEX
        FEX  --> CRASH
        FEX  --> COV
        COV  --> CEVAL
        CEVAL --> CREP
    end

    %% ── Cross-layer connections ───────────────────────────────────
    API --> PC
    GFD --> CMP

    %% ── Styles ───────────────────────────────────────────────────
    style SRC   fill:#dce6f1,stroke:#333,stroke-width:2px
    style HDR   fill:#dce6f1,stroke:#333,stroke-width:2px
    style LLM   fill:#dce6f1,stroke:#333,stroke-width:2px
    style BIN   fill:#dce6f1,stroke:#333,stroke-width:2px
    style SEED  fill:#dce6f1,stroke:#333,stroke-width:2px
    style CRASH fill:#dce6f1,stroke:#333,stroke-width:2px
    style COV   fill:#dce6f1,stroke:#333,stroke-width:2px

    style CMP fill:#fde8cd,stroke:#333,stroke-width:2px
    style FEX fill:#fde8cd,stroke:#333,stroke-width:2px

    style CTX   fill:#fff3cd,stroke:#856404,stroke-width:2px
    style API   fill:#d4edda,stroke:#155724,stroke-width:2px
    style FI    fill:#d4edda,stroke:#155724,stroke-width:2px
    style PC    fill:#fff3cd,stroke:#856404,stroke-width:2px
    style PR    fill:#d4edda,stroke:#155724,stroke-width:2px
    style GEN   fill:#fff3cd,stroke:#856404,stroke-width:2px
    style GFD   fill:#d4edda,stroke:#155724,stroke-width:2px
    style CEVAL fill:#fff3cd,stroke:#856404,stroke-width:2px
    style CREP  fill:#d4edda,stroke:#155724,stroke-width:2px

    style InputLayer fill:#f0f4ff,stroke:#668,stroke-width:1px,stroke-dasharray:5
    style GenLayer   fill:#fffde6,stroke:#886,stroke-width:1px,stroke-dasharray:5
    style ExecLayer  fill:#fff5ee,stroke:#864,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Technical Architecture of the LLM-based fuzz-driver generation pipeline. Three layers with dashed borders. Blue = existing data; green = new data (our contribution); orange = existing process; yellow = new process (our contribution).

**Legend:**

| Style | Meaning |
|-------|---------|
| Blue rectangle | Existing data |
| Green rectangle | New data (our contribution) |
| Orange rounded | Existing process |
| Yellow rounded | New process (our contribution) |

---

## Figure 3.2 — Enterprise Integration Strategy

> **Layout note:** The original `flowchart LR` was ~1500 × 160 px — too flat for a slide.
> This version uses `flowchart TD` so the three zones (CARIAD → Firewall → Azure) stack vertically and the bidirectional flow reads top-to-bottom, filling the slide height.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 30, "nodeSpacing": 50, "rankSpacing": 65}} }%%
flowchart TD
    subgraph CARIAD["CARIAD Network"]
        RUNNER(["Self-hosted<br/>CI/CD/CT Runner"])
    end

    FW["Corporate<br/>Firewall"]

    subgraph Azure["Azure Cloud"]
        PL(["Azure<br/>Private Link"])
        LLM(["Azure OpenAI<br/>LLM Service"])
        PL -->|LLM prompt| LLM
        LLM -->|Generated fuzz driver| PL
    end

    RUNNER -->|"API request"| FW
    FW     -->|"Private connection"| PL
    PL     -->|"Generated fuzz driver"| FW
    FW     -->|"Generated fuzz driver"| RUNNER

    style RUNNER fill:#fde8cd,stroke:#333,stroke-width:2px
    style FW     fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style PL     fill:#fde8cd,stroke:#333,stroke-width:2px
    style LLM    fill:#fde8cd,stroke:#333,stroke-width:2px
    style CARIAD fill:#dce6f1,stroke:#336,stroke-width:1px,stroke-dasharray:5
    style Azure  fill:#fff3cd,stroke:#886,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Enterprise Integration Strategy. The self-hosted CI/CD/CT runner sends API requests through the corporate firewall to Azure OpenAI via Private Link — traffic never traverses the public internet.

---

## Figure 4.1 — Evaluation Environment Component Diagram

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 50, "rankSpacing": 60}} }%%
flowchart TD
    %% ===== DATA (rectangular) =====
    TGT["C++ Target Library<br/><i>yaml-cpp, etc.</i>"]
    DRV["Generated Fuzz Driver<br/><i>C++ source code</i>"]
    BIN["Instrumented Binary"]
    COV["Coverage Report"]

    %% ===== COMPONENTS (rounded / stadium) =====
    OLL(["<b>Ollama</b><br/><i>Local LLM Server</i>"])
    LCP(["<b>llama.cpp</b><br/><i>Alternative Backend</i>"])
    CIF(["<b>cifuzz spark</b><br/><i>Driver Generation</i>"])
    CMK(["<b>CMake + Clang</b><br/><i>Build System</i>"])
    LFZ(["<b>libFuzzer</b><br/><i>Fuzzing Engine</i>"])
    LCV(["<b>llvm-cov</b><br/><i>Coverage Tool</i>"])
    POD(["<b>Podman</b><br/><i>Container Runtime</i>"])

    %% ===== FLOW =====
    OLL -->|"OpenAI API"| CIF
    LCP -->|"OpenAI API"| CIF
    TGT -->|"Header files"| CIF
    CIF -->|"C++ source"| DRV
    DRV -->|"Compile"| CMK
    CMK -->|"Link"| BIN
    BIN -->|"Execute"| LFZ
    LFZ -->|"Coverage data"| LCV
    LCV -->|"Report"| COV

    POD -.->|"manages"| CIF
    POD -.->|"manages"| CMK
    POD -.->|"manages"| LFZ

    %% ===== STYLING =====
    style TGT fill:#d4edda,stroke:#333,stroke-width:2px
    style DRV fill:#d4edda,stroke:#333,stroke-width:2px
    style BIN fill:#d4edda,stroke:#333,stroke-width:2px
    style COV fill:#d4edda,stroke:#333,stroke-width:2px

    style OLL fill:#dce6f1,stroke:#333,stroke-width:2px
    style LCP fill:#dce6f1,stroke:#333,stroke-width:2px
    style CIF fill:#fde8cd,stroke:#333,stroke-width:2px
    style CMK fill:#fde8cd,stroke:#333,stroke-width:2px
    style LFZ fill:#fde8cd,stroke:#333,stroke-width:2px
    style LCV fill:#fde8cd,stroke:#333,stroke-width:2px
    style POD fill:#e8e8e8,stroke:#666,stroke-width:2px,stroke-dasharray:5
```

**Caption:** Evaluation Environment. Ollama or llama.cpp serve the LLM; cifuzz spark orchestrates driver generation; CMake/Clang builds; libFuzzer executes; llvm-cov measures coverage. All build and fuzzing components run inside a Podman container.

---

## Figure 4.2 — CI/CD/CT Pipeline Workflow

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 50, "rankSpacing": 60}} }%%
flowchart TD
    %% ===== DATA (rectangular) =====
    TRIG["Git Push<br/><i>code commit</i>"]
    CRASH["Crash<br/>Reports"]
    COVD["Coverage<br/>Data"]

    %% ===== PROCESSING (rounded / stadium) =====
    CO(["Checkout<br/>Repository"])
    LI(["Login to<br/>JFrog Artifactory"])
    DC(["Download<br/>Previous Corpus"])
    PC(["Pull cifuzz<br/>Container Image"])
    BLD(["Build Project<br/><i>CMake + Clang</i>"])
    SPK(["<b>cifuzz spark</b><br/><i>Generate Fuzz Driver</i><br/><i>via Azure OpenAI</i>"])
    RB(["Rebuild with<br/>Generated Driver"])
    FZ(["Run libFuzzer<br/><i>60 s, ASan + UBSan</i>"])
    UL(["Upload Updated<br/>Corpus to Artifactory"])

    %% ===== FLOW =====
    TRIG --> CO
    CO   --> LI
    LI   --> DC
    CO   --> PC
    PC   --> BLD
    DC   --> BLD
    BLD  --> SPK
    SPK  --> RB
    RB   --> FZ
    FZ   --> CRASH
    FZ   --> COVD
    FZ   --> UL
    UL   -->|next run| DC

    %% ===== STYLING =====
    style TRIG  fill:#dce6f1,stroke:#333,stroke-width:2px
    style CRASH fill:#dce6f1,stroke:#333,stroke-width:2px
    style COVD  fill:#dce6f1,stroke:#333,stroke-width:2px

    style CO  fill:#fde8cd,stroke:#333,stroke-width:2px
    style LI  fill:#fde8cd,stroke:#333,stroke-width:2px
    style DC  fill:#fde8cd,stroke:#333,stroke-width:2px
    style PC  fill:#fde8cd,stroke:#333,stroke-width:2px
    style BLD fill:#fde8cd,stroke:#333,stroke-width:2px
    style SPK fill:#fff3cd,stroke:#856404,stroke-width:2px
    style RB  fill:#fde8cd,stroke:#333,stroke-width:2px
    style FZ  fill:#fde8cd,stroke:#333,stroke-width:2px
    style UL  fill:#fde8cd,stroke:#333,stroke-width:2px
```

**Caption:** CI/CD/CT Pipeline Workflow. cifuzz spark (highlighted yellow) invokes the LLM via Azure Private Link. Corpus persistence through JFrog Artifactory enables cumulative coverage improvement across runs.

---

## Figure 6.1 — Enterprise Network Architecture with Azure Private Link

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 50, "rankSpacing": 60}} }%%
flowchart TD
    %% ===== CORPORATE NETWORK =====
    subgraph Corporate["Corporate Network (CARIAD)"]
        direction TB
        CODE["Source Code<br/>Repository"]
        RUNNER(["GitHub Actions<br/>Self-hosted Runner"])
        CODE -->|"Source code +<br/>public headers"| RUNNER
    end

    %% ===== SECURITY BOUNDARY =====
    FW["Corporate<br/>Firewall"]
    INTERNET(("Public<br/>Internet"))

    %% ===== AZURE CLOUD =====
    subgraph Azure["Azure Cloud"]
        direction TB
        PL(["Azure Private Link<br/>Endpoint"])
        LLM(["Azure OpenAI<br/>LLM Service"])
    end

    %% ===== FORWARD PATH =====
    RUNNER -->|"API request<br/>(headers + fuzzing instructions)"| FW
    FW     -->|"Private connection (encrypted)"| PL
    PL     -->|"LLM prompt (API context + instructions)"| LLM

    %% ===== RETURN PATH =====
    LLM -->|"Generated fuzz driver (C++ source code)"| PL
    PL  -->|"Generated fuzz driver"| FW
    FW  -->|"Generated fuzz driver"| RUNNER

    %% ===== BLOCKED PATH =====
    FW -.-x|"Not used"| INTERNET

    %% ===== STYLING =====
    style CODE      fill:#d4edda,stroke:#333,stroke-width:2px
    style RUNNER    fill:#fde8cd,stroke:#333,stroke-width:2px
    style FW        fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style INTERNET  fill:#e8e8e8,stroke:#666,stroke-width:2px,stroke-dasharray:5
    style PL        fill:#fde8cd,stroke:#333,stroke-width:2px
    style LLM       fill:#fde8cd,stroke:#333,stroke-width:2px
    style Corporate fill:#dce6f1,stroke:#336,stroke-width:1px,stroke-dasharray:5
    style Azure     fill:#fff3cd,stroke:#886,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Enterprise Network Architecture with Azure Private Link. All edges are labeled with the data exchanged. Traffic flows through a private, encrypted connection and never traverses the public internet.

---

## Figure LoRA — How LoRA Works

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "18px"}, "flowchart": {"diagramPadding": 24, "nodeSpacing": 50, "rankSpacing": 60}} }%%
flowchart TD

    INPUT["Input  x"]

    subgraph frozen_path ["Frozen Path  (no gradient)"]
        W["Pre-trained Weight  W\n❄ not updated during training"]
    end

    subgraph lora_path ["LoRA Adapter  (trainable)"]
        DROP(["Dropout"])
        A["Matrix A\ndown-project  d → r"]
        B["Matrix B\nup-project  r → d"]
    end

    ADD(["➕  Add\nh = Wx + BAx"])
    OUTPUT["Output  h"]

    INPUT  --> W
    INPUT  --> DROP
    DROP   --> A
    A      --> B
    W      --> ADD
    B      --> ADD
    ADD    --> OUTPUT

    style INPUT  fill:#dce6f1,stroke:#333,stroke-width:2px
    style W      fill:#dce6f1,stroke:#333,stroke-width:2px
    style A      fill:#d4edda,stroke:#155724,stroke-width:2px
    style B      fill:#d4edda,stroke:#155724,stroke-width:2px
    style DROP   fill:#fde8cd,stroke:#333,stroke-width:2px
    style ADD    fill:#fde8cd,stroke:#333,stroke-width:2px
    style OUTPUT fill:#dce6f1,stroke:#333,stroke-width:2px

    style frozen_path fill:#eef4fb,stroke:#6699cc,stroke-width:1px,stroke-dasharray:5
    style lora_path   fill:#edfaf1,stroke:#27ae60,stroke-width:1px,stroke-dasharray:5
```

**Caption:** LoRA adds a small trainable bypass path (A then B) alongside each frozen pre-trained weight W. Only the low-rank matrices A and B are updated during fine-tuning; the base model weights remain unchanged.

---

## How to Put Each Diagram on Its Own Slide

1. **Open** [mermaid.live](https://mermaid.live)
2. **Paste** the code block for the diagram you want
3. **Export → PNG** — choose **4× scale** in the export dialog (produces ≈ 3840 × 2160 px at roughly landscape aspect ratio)
4. In **PowerPoint**:
   - Insert → Pictures → pick your exported PNG
   - Right-click → **Size and Position** → set **Height = 19.05 cm** and **Width = 33.87 cm** (16:9 full-bleed), **uncheck** "Lock aspect ratio" only if needed
   - Or just drag the corners to fill the slide
5. Add a text box at the bottom for the caption if desired
6. **Repeat** for each diagram (one slide per figure)

### Color Legend (same across all diagrams)

| Box color | Shape | Meaning |
|---|---|---|
| Blue `#dce6f1` | Rectangle | Existing data artifact |
| Green `#d4edda` | Rectangle | New data (our contribution) |
| Orange `#fde8cd` | Rounded | Existing processing step |
| Yellow `#fff3cd` | Rounded | New processing step (our contribution) |
| Red `#f8d0d0` | Rounded | Bottleneck / highlighted concern |
| Grey `#e8e8e8` | Rounded / dashed | Infrastructure / container runtime |
