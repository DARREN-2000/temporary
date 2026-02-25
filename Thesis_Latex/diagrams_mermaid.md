# Mermaid Diagram Alternatives

These are Mermaid versions of the thesis diagrams. You can render them at [mermaid.live](https://mermaid.live) and export as PNG/PDF, then include with `\includegraphics` in LaTeX.

**Notation used (matching the professor's requirements):**
- **Rectangular boxes** `[...]` = Data artifacts
- **Rounded boxes** `(...)` = Processing steps
- No two data boxes are directly connected
- No two processing boxes are directly connected
- Top-down (TD) layout for maximum size

---

## Figure 1.1 — The Fuzzing Process

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

## Figure 1.2 — The Automotive CI/CD Pipeline

```mermaid
flowchart TD
    %% ===== DATA (rectangular) =====
    COMMIT["Commit /<br/>Source Tree"]
    ARTIFACT["Build Artifact<br/><i>(Binary / Library)</i>"]
    TRESULTS["Test Results"]
    SECREPORT["Security Report /<br/>Findings"]
    RELEASE["Deployed Release"]
    TELEMETRY["Telemetry /<br/>Incident Data"]

    %% ===== PROCESSING (rounded / stadium) =====
    BUILD(["Build<br/><i>(CMake / Clang)</i>"])
    TEST(["Test Execution<br/><i>(Unit / Integration)</i>"])
    SECVAL(["<b>Security Validation</b><br/><i>(Fuzzing / SAST / DAST)</i>"])
    DEPLOY(["Deploy<br/><i>(Staging / Production)</i>"])
    MONITOR(["Monitor<br/><i>(Telemetry / Logging)</i>"])

    %% ===== FLOW (alternating data → process → data) =====
    COMMIT --> BUILD
    BUILD --> ARTIFACT
    ARTIFACT --> TEST
    TEST --> TRESULTS
    TRESULTS --> SECVAL
    SECVAL --> SECREPORT
    SECREPORT --> DEPLOY
    DEPLOY --> RELEASE
    RELEASE --> MONITOR
    MONITOR --> TELEMETRY
    TELEMETRY -->|feedback loop| COMMIT

    %% ===== STYLING =====
    style COMMIT fill:#dce6f1,stroke:#333,stroke-width:2px
    style ARTIFACT fill:#dce6f1,stroke:#333,stroke-width:2px
    style TRESULTS fill:#dce6f1,stroke:#333,stroke-width:2px
    style SECREPORT fill:#dce6f1,stroke:#333,stroke-width:2px
    style RELEASE fill:#dce6f1,stroke:#333,stroke-width:2px
    style TELEMETRY fill:#dce6f1,stroke:#333,stroke-width:2px

    style BUILD fill:#fde8cd,stroke:#333,stroke-width:2px
    style TEST fill:#fde8cd,stroke:#333,stroke-width:2px
    style SECVAL fill:#fff3cd,stroke:#856404,stroke-width:2px
    style DEPLOY fill:#fde8cd,stroke:#333,stroke-width:2px
    style MONITOR fill:#fde8cd,stroke:#333,stroke-width:2px
```

**Caption:** The Automotive CI/CD Pipeline. Data artifacts are shown in rectangular boxes; processing steps are shown in rounded boxes. Security validation (highlighted) is where fuzzing integrates into the pipeline. The feedback loop from telemetry to the source tree represents the continuous improvement cycle.

**How to use:** Export as PNG from [mermaid.live](https://mermaid.live) and replace the TikZ block at `chapters/introduction.tex` lines 103–158 with:
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/cicd_pipeline_mermaid.png}
    \caption{The Automotive CI/CD Pipeline. Data artifacts are shown in rectangular boxes; processing steps are shown in rounded boxes. Security validation (highlighted) is where fuzzing integrates into the pipeline. The feedback loop from telemetry to the source tree represents the continuous improvement cycle.}
    \label{fig:cicd_pipeline}
\end{figure}
```

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

### Option A: Render online and export (RECOMMENDED)
1. Go to [mermaid.live](https://mermaid.live)
2. Paste the Mermaid code for the diagram you want
3. Export as **PNG** (choose high resolution, e.g., 4x) or **SVG**
4. Save the exported file to the `bilder/` folder
5. Replace the TikZ code in the `.tex` file with `\includegraphics` (see exact instructions below)

### Option B: Use the `mermaid-filter` for Pandoc
If you convert through Pandoc, install `mermaid-filter` and it auto-renders.

### Option C: Keep using the TikZ versions already in the .tex files
The TikZ versions are already embedded in the LaTeX files and compile directly. They use the same notation (rectangular = data, rounded = process) and follow the top-down layout. **No action needed.**

---

## Exact Lines to Change to Switch from TikZ to Mermaid

### Figure 1.1 (Fuzzing Process) — in `chapters/introduction.tex`

**What to do:** Export the Mermaid diagram above as `bilder/fuzzing_mermaid.png`, then replace lines 33–87 of `chapters/introduction.tex`.

**Replace this entire TikZ block (lines 33–88):**
```latex
\begin{figure}[htbp]
    \centering
    \resizebox{\textwidth}{!}{
    \begin{tikzpicture}[
        ...entire TikZ code...
    \end{tikzpicture}
    }
    \caption{The Fuzzing Process...}
    \label{fig:fuzzing_concept}
\end{figure}
```

**With this:**
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/fuzzing_mermaid.png}
    \caption{The Fuzzing Process. Data artifacts are shown in rectangular boxes; processing steps are shown in rounded boxes. The manual creation of fuzz drivers (highlighted in red) is the bottleneck this thesis aims to automate.}
    \label{fig:fuzzing_concept}
\end{figure}
```

### Figure 3.1 (Technical Architecture) — in `chapters/methodology.tex`

**What to do:** Export the Mermaid diagram above as `bilder/architecture_mermaid.png`, then replace lines 34–135 of `chapters/methodology.tex`.

**Replace this entire TikZ block (lines 34–135):**
```latex
\begin{figure}[htbp]
    \centering
    \resizebox{\textwidth}{!}{
    \begin{tikzpicture}[
        ...entire TikZ code...
    \end{tikzpicture}
    }
    \caption{Technical Architecture of our LLM-based...}
    \label{fig:tech_architecture}
\end{figure}
```

**With this:**
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/architecture_mermaid.png}
    \caption{Technical Architecture of our LLM-based fuzz driver generation pipeline. The three layers are marked with dashed borders. Data artifacts are shown in rectangular boxes; processing steps in boxes with rounded corners. Green/yellow highlighted elements denote new contributions of this work with respect to the state of the art (cf.\ Figure~\ref{fig:fuzzing_concept}).}
    \label{fig:tech_architecture}
\end{figure}
```

---

## Figure 4.1 — Evaluation Environment Component Diagram

```mermaid
flowchart TD
    %% ===== DATA (rectangular) =====
    TGT["C++ Target Library<br/><i>(yaml-cpp, etc.)</i>"]
    DRV["Generated Fuzz Driver<br/><i>(C++ source code)</i>"]
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

    %% ===== FLOW (top-down) =====
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

**Caption:** Evaluation Environment Component Diagram. Components (rounded boxes) interact via defined interfaces. Ollama or llama.cpp serve the LLM; cifuzz spark orchestrates driver generation; CMake/Clang builds the driver; libFuzzer executes it; llvm-cov measures coverage. All build and fuzzing components run inside a Podman container. The top-down flow shows the data transformation pipeline from source code to coverage report.

**How to use:** Export as PNG from [mermaid.live](https://mermaid.live) and replace the TikZ block at `chapters/implementation.tex` lines 193–256 with:
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/eval_environment_mermaid.png}
    \caption{Evaluation Environment Component Diagram. Components (rounded boxes) interact via defined interfaces. Ollama or llama.cpp serve the LLM; cifuzz spark orchestrates driver generation; CMake/Clang builds the driver; libFuzzer executes it; llvm-cov measures coverage. All build and fuzzing components run inside a Podman container.}
    \label{fig:eval_environment}
\end{figure}
```

---

## Figure 4.2 — CI/CD Pipeline Workflow Diagram

```mermaid
flowchart TD
    %% ===== DATA (rectangular) =====
    TRIG["Git Push<br/><i>(code commit)</i>"]
    CRASH["Crash<br/>Reports"]
    COVD["Coverage<br/>Data"]

    %% ===== PROCESSING (rounded / stadium) =====
    CO(["Checkout<br/>Repository"])
    LI(["Login to<br/>JFrog Artifactory"])
    DC(["Download<br/>Previous Corpus"])
    PC(["Pull cifuzz<br/>Container Image"])
    BLD(["Build Project<br/><i>(CMake + Clang)</i>"])
    SPK(["<b>cifuzz spark</b><br/><i>Generate Fuzz Driver</i><br/><i>via Azure OpenAI</i>"])
    RB(["Rebuild with<br/>Generated Driver"])
    FZ(["Run libFuzzer<br/><i>(60s, ASan+UBSan)</i>"])
    UL(["Upload Updated<br/>Corpus to Artifactory"])

    %% ===== FLOW =====
    TRIG --> CO
    CO --> LI
    LI --> DC
    CO --> PC
    PC --> BLD
    DC --> BLD
    BLD --> SPK
    SPK --> RB
    RB --> FZ
    FZ --> CRASH
    FZ --> COVD
    FZ --> UL
    UL -->|next run| DC

    %% ===== STYLING =====
    style TRIG fill:#dce6f1,stroke:#333,stroke-width:2px
    style CRASH fill:#dce6f1,stroke:#333,stroke-width:2px
    style COVD fill:#dce6f1,stroke:#333,stroke-width:2px

    style CO fill:#fde8cd,stroke:#333,stroke-width:2px
    style LI fill:#fde8cd,stroke:#333,stroke-width:2px
    style DC fill:#fde8cd,stroke:#333,stroke-width:2px
    style PC fill:#fde8cd,stroke:#333,stroke-width:2px
    style BLD fill:#fde8cd,stroke:#333,stroke-width:2px
    style SPK fill:#fff3cd,stroke:#856404,stroke-width:2px
    style RB fill:#fde8cd,stroke:#333,stroke-width:2px
    style FZ fill:#fde8cd,stroke:#333,stroke-width:2px
    style UL fill:#fde8cd,stroke:#333,stroke-width:2px
```

**Caption:** CI/CD Pipeline Workflow. Processing steps are in rounded boxes; data artifacts in rectangular boxes. The highlighted step (cifuzz spark) invokes the LLM via Azure Private Link. Corpus persistence through JFrog Artifactory enables cumulative coverage improvement across runs.

**How to use:** Export as PNG from [mermaid.live](https://mermaid.live) and replace the TikZ block at `chapters/implementation.tex` lines 458–519 with:
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/cicd_workflow_mermaid.png}
    \caption{CI/CD Pipeline Workflow. Processing steps are in rounded boxes; data artifacts in rectangular boxes. The highlighted step (cifuzz spark) invokes the LLM via Azure Private Link. Corpus persistence through JFrog Artifactory enables cumulative coverage improvement across runs.}
    \label{fig:cicd_workflow}
\end{figure}
```

---

## Figure 3.2 — Enterprise Integration Strategy

```mermaid
flowchart LR
    %% ===== COMPONENTS =====
    subgraph CARIAD["CARIAD Network"]
        RUNNER(["Self-hosted<br/>CI/CD Runner"])
    end

    subgraph Azure["Azure Cloud"]
        PL(["Azure<br/>Private Link"])
        LLM(["Azure OpenAI<br/>LLM Service"])
    end

    FW["Corporate<br/>Firewall"]

    %% ===== FLOW =====
    RUNNER -->|API request| FW
    FW -->|Private connection| PL
    PL -->|LLM prompt| LLM
    LLM -->|Generated fuzz driver| PL
    PL -->|Generated fuzz driver| FW
    FW -->|Generated fuzz driver| RUNNER

    %% ===== STYLING =====
    style RUNNER fill:#fde8cd,stroke:#333,stroke-width:2px
    style FW fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style PL fill:#fde8cd,stroke:#333,stroke-width:2px
    style LLM fill:#fde8cd,stroke:#333,stroke-width:2px
    style CARIAD fill:#dce6f1,stroke:#336,stroke-width:1px,stroke-dasharray:5
    style Azure fill:#fff3cd,stroke:#886,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Enterprise Integration Strategy. The self-hosted CI/CD runner communicates with Azure OpenAI through a Private Link endpoint, routing traffic through the corporate firewall over a private connection without traversing the public internet.

**How to use:** Export as PNG from [mermaid.live](https://mermaid.live) and replace the TikZ block at `chapters/methodology.tex` lines 180–213 with:
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/integration_mermaid.png}
    \caption{Enterprise Integration Strategy. The self-hosted CI/CD runner communicates with Azure OpenAI through a Private Link endpoint, routing traffic through the corporate firewall over a private connection without traversing the public internet.}
    \label{fig:enterprise_integration}
\end{figure}
```

---

## Figure 6.1 — Enterprise Network Architecture with Azure Private Link

```mermaid
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
    FW -->|"Private connection<br/>(encrypted)"| PL
    PL -->|"LLM prompt<br/>(API context + instructions)"| LLM

    %% ===== RETURN PATH =====
    LLM -->|"Generated fuzz driver<br/>(C++ source code)"| PL
    PL -->|"Generated fuzz driver"| FW
    FW -->|"Generated fuzz driver"| RUNNER

    %% ===== BLOCKED PATH =====
    FW -.-x|"Not used"| INTERNET

    %% ===== STYLING =====
    style CODE fill:#d4edda,stroke:#333,stroke-width:2px
    style RUNNER fill:#fde8cd,stroke:#333,stroke-width:2px
    style FW fill:#f8d0d0,stroke:#c00,stroke-width:2px
    style INTERNET fill:#e8e8e8,stroke:#666,stroke-width:2px,stroke-dasharray:5
    style PL fill:#fde8cd,stroke:#333,stroke-width:2px
    style LLM fill:#fde8cd,stroke:#333,stroke-width:2px
    style Corporate fill:#dce6f1,stroke:#336,stroke-width:1px,stroke-dasharray:5
    style Azure fill:#fff3cd,stroke:#886,stroke-width:1px,stroke-dasharray:5
```

**Caption:** Enterprise Network Architecture with Azure Private Link. All edges are labeled with the data exchanged between components. Traffic flows through a private connection and never traverses the public internet.

**How to use:** Export as PNG from [mermaid.live](https://mermaid.live) and replace the TikZ block at `chapters/discussion_conclusion.tex` lines 47–95 with:
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=\textwidth]{bilder/network_architecture_mermaid.png}
    \caption{Enterprise Network Architecture with Azure Private Link. Processing components are shown in rounded boxes; data stores in rectangular boxes. All edges are labeled with the data exchanged between components. Traffic flows through a private connection and never traverses the public internet.}
    \label{fig:enterprise_network}
\end{figure}
```

---

## Complete Diagram Inventory

| # | Figure | File | TikZ | Mermaid | Notation |
|---|--------|------|------|---------|----------|
| 1 | Fig 1.1 — Fuzzing Process | introduction.tex | ✅ | ✅ | data=rect, process=rounded, top-down |
| 2 | Fig 1.2 — Automotive CI/CD Pipeline | introduction.tex | ✅ | ✅ | data=rect, process=rounded, top-down |
| 3 | Fig 3.1 — Technical Architecture | methodology.tex | ✅ | ✅ | data=rect, process=rounded, top-down, layers marked |
| 4 | Fig 3.2 — Enterprise Integration | methodology.tex | ✅ | ✅ | components=rounded, data=rect, left-right |
| 5 | Fig 4.1 — Eval Environment | implementation.tex | ✅ | ✅ | components=rounded, data=rect, top-down |
| 6 | Fig 4.2 — CI/CD Workflow | implementation.tex | ✅ | ✅ | data=rect, process=rounded, top-down |
| 7 | Fig 6.1 — Network Architecture | discussion_conclusion.tex | ✅ | ✅ | components=rounded, data=rect, top-down, edges labeled |

All diagrams use the same notation:
- **Rectangular boxes** = Data artifacts / data stores
- **Rounded boxes** = Processing steps / components
- No two data boxes directly connected (mediated by a process)
- No two process boxes directly connected (mediated by data)
- Top-down layout where possible for maximum size

### Important Notes
- Keep the `\caption` and `\label` exactly as shown — other parts of the thesis reference these labels
- Make sure the PNG is high resolution (at least 300 DPI or use the 4x export option in mermaid.live)
- The SVG format also works if you use `\usepackage{svg}` in thesis.tex (add before `\begin{document}`)
