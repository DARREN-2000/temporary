# Complete Changes Summary — Professor's Feedback Response

This document lists every professor comment, what was in the original thesis, and what was changed to address the feedback.

---

## Chapter 1 — Introduction

### Comment 1: Figure 1.2 diagram notation
**Professor:** "Please update the figure: data is in boxes and processing steps are in boxes with rounded corners. No two data boxes are directly connected and no two processing steps are directly connected."

| Before | After |
|--------|-------|
| PNG image (`bilder/fuzzing.png`) with no consistent notation | TikZ diagram with rectangular boxes = data, rounded boxes = processing. No two data/process boxes directly connected. Top-down layout at full `\textwidth`. |
| **File:** `chapters/introduction.tex` lines 33–88 | Same location, replaced with TikZ code |

---

## Chapter 2 — Literature Review

### Comment 2: LoRA theory
**Professor:** "Either in Section 2 or Section 3 provide more information about how LoRA works, i.e., about the theory behind it (see: https://arxiv.org/abs/2106.09685)"

| Before | After |
|--------|-------|
| LoRA mentioned without explanation | Added subsection in Section 2.2 with mathematical explanation: `ΔW = BA`, rank reduction formula, parameter count comparison, practical advantages |
| **File:** `chapters/literature.tex` lines 40–53 | New subsection with `\cite{Hu2021}` reference |

### Comment 3: Research gap at end of Section 2.4
**Professor:** "Here, at the end of section 2.4, summarize again the gap of the LLM-based techniques."

| Before | After |
|--------|-------|
| Section 2.4 ended without gap summary | Added subsection "Research Gap of LLM-Based Fuzzing Techniques" covering: (1) compiles-vs-meaningful-coverage gap, (2) practical constraints (incomplete docs, cloud access conflicts, manual workflows), (3) thesis positioning |
| **File:** `chapters/literature.tex` lines 89–96 | New subsection at end of Section 2.4 |

### Comment 4: V-Model citation
**Professor:** (implied) V-Model diagram should cite source

| Before | After |
|--------|-------|
| V-Model figure without attribution | Caption updated: "adapted from ISO~26262~\cite{ISO26262}" |
| **File:** `chapters/literature.tex` | Figure caption |

---

## Chapter 3 — Methodology

### Comment 5: "components" vs "layers"
**Professor:** "Do you really mean components or layers as in Section 3.2.1-3.2.3?"

| Before | After |
|--------|-------|
| "three main **components**: Input Analysis, Generation, and Execution" | "three main **layers**: the **Input Layer**, the **Generation Layer**, and the **Execution Layer**" |
| **File:** `chapters/methodology.tex` line 32 | Same location |

### Comment 6: "data flows from source code"
**Professor:** "which data flows from the source code? Do you mean the source code is the input of s.th.?"

| Before | After |
|--------|-------|
| "the data flows from source code through the LLM to the final execution engine" | "the source code and its public header files serve as the primary input to the pipeline. The Input Layer extracts API context, which flows to the Generation Layer for prompt construction and LLM-based driver generation, and finally to the Execution Layer for compilation, fuzzing, and coverage evaluation" |
| **File:** `chapters/methodology.tex` line 137 | Same location |

### Comment 7: Mark layers in Figure 3.1
**Professor:** "Mark the respective layers in Figure 3.1."

| Before | After |
|--------|-------|
| PNG image (`bilder/architecture.png`) without layer markings | TikZ diagram with three dashed-border subgraphs labeled "Input Layer (§3.2.1)", "Generation Layer (§3.2.2)", "Execution Layer (§3.2.3)" |
| **File:** `chapters/methodology.tex` lines 34–135 | Replaced with TikZ code with layer boundaries |

### Comment 8: Add cifuzz spark to Figure 3.1 + reference
**Professor:** "Add this to figure 3.1. Also give details where this comes from?"

| Before | After |
|--------|-------|
| cifuzz spark not in diagram, no citation | cifuzz spark appears as a processing node in the Input Layer of Figure 3.1. Text references `\cite{cifuzzspark}` with explanation of its provenance |
| **File:** `chapters/methodology.tex` lines 52, 141 | Diagram node + text reference |

### Comment 9: Steps in Figure (Section 3.2.2)
**Professor:** "Make sure that these steps can also be found in the Figure."

| Before | After |
|--------|-------|
| Prompt construction steps only in text | "Prompt Construction", "Fuzzing Instructions", "Extracted API Context", "LLM-Based Driver Generation" all appear as labeled nodes in Figure 3.1 Generation Layer |
| **File:** `chapters/methodology.tex` lines 57–68 | Generation Layer subgraph |

### Comment 10: "fuzzing instructions" — explain
**Professor:** "what's this. explain"

| Before | After |
|--------|-------|
| Term used without definition | Defined: "predefined rules and constraints that describe what a valid libFuzzer fuzz driver must look like, including the required entry point signature, buffer consumption patterns, and safety constraints such as avoiding global state or file I/O" |
| **File:** `chapters/methodology.tex` line 147 | Inline definition |

### Comment 11: "fuzz test case structure" — explain
**Professor:** "what's this. explain"

| Before | After |
|--------|-------|
| Term used without definition | Defined: "the driver must implement the `LLVMFuzzerTestOneInput` entry point, consume a `const uint8_t*` data buffer of a given `size_t` length, and return 0" |
| **File:** `chapters/methodology.tex` line 147 | Inline definition |

### Comment 12: "extracted API context" — explain
**Professor:** "what's this. explain"

| Before | After |
|--------|-------|
| Term used without definition | Defined: "structured function signatures, types, and class definitions produced by the Input Layer from the public header files of the target library" |
| **File:** `chapters/methodology.tex` line 147 | Inline definition |

### Comment 13: "environment" — explain
**Professor:** "what's this. explain"

| Before | After |
|--------|-------|
| Term used without definition | Defined: "the runtime configuration that determines which LLM backend serves the generation requests (e.g., a local Ollama instance or a cloud-based Azure OpenAI endpoint)" |
| **File:** `chapters/methodology.tex` line 149 | Inline definition |

### Comment 14: "Generated drivers" — explain origin
**Professor:** "where do they come from? explain"

| Before | After |
|--------|-------|
| "Generated drivers are..." without origin | "Generated drivers, that is, the C++ source files produced by the LLM in the Generation Layer, are..." |
| **File:** `chapters/methodology.tex` line 154 | Inline definition |

### Comment 15: Figure 3.1 — highlight new contributions
**Professor:** "highlight boxes that are new, i.e., concepts and steps added by you. Use a higher level of detail."

| Before | After |
|--------|-------|
| Only "LLM" and "Seed Inputs" shown as new | Four contribution categories in Figure 3.1 with legend: green rectangles = new data (Extracted API Context, Fuzzing Instructions, Generated Fuzz Driver, Coverage Report), yellow rounded = new processes (Automated Context Extraction, Prompt Construction, LLM-Based Driver Generation, Coverage Evaluation) |
| **File:** `chapters/methodology.tex` lines 34–135 | TikZ with `newdata`/`newproc` styles |

### Comment 16: Figure 3.2 — Enterprise Integration
| Before | After |
|--------|-------|
| PNG image (`bilder/integration.png`) | TikZ diagram showing CARIAD Runner → Firewall → Azure Private Link → Azure OpenAI, with labeled edges |
| **File:** `chapters/methodology.tex` lines 180–213 | Replaced with TikZ code |

---

## Chapter 4 — Implementation

### Comment 17: Phase naming mismatch
**Professor:** "In Chapter 3, Section 3.3.1, 3.3.2, and 3.3.3. the phases are named differently."

| Before | After |
|--------|-------|
| "baseline assessment, model evaluation, and fine-tuning" | "Phase 1: Local LLM Evaluation, Phase 2: Model Optimization with LoRA, and Phase 3: Enterprise CI/CD Integration" (matching Chapter 3 exactly) |
| **File:** `chapters/implementation.tex` line 6 | Same location |

### Comment 18: Example prompt
**Professor:** "Could you give an example of such a prompt?"

| Before | After |
|--------|-------|
| No prompt example | Added representative prompt in Appendix (thesis.tex) with reference from Chapter 4: "A representative example prompt is provided in Appendix~\ref{sec:appendix_prompt}" |
| **File:** `thesis.tex` lines 346–389, `chapters/implementation.tex` line 47 | Appendix + reference |

### Comment 19: Model summary table
**Professor:** "also summarize for all models in a table"

| Before | After |
|--------|-------|
| Models described only in prose | Added Table with all 14 models: columns for Model, Type, Parameters, Quantization, Disk Size, yaml-cpp Coverage, Status |
| **File:** `chapters/implementation.tex` lines 127–153 | New table |

### Comment 20: Evaluation environment graphic
**Professor:** "could you provide a graphic visualizing all the components?"

| Before | After |
|--------|-------|
| Only numbered list of steps | Added TikZ component diagram (Figure 4.1) showing Ollama/llama.cpp → cifuzz spark → CMake+Clang → libFuzzer → llvm-cov, with data artifacts and Podman container boundary. Top-down layout with clear spacing. |
| **File:** `chapters/implementation.tex` lines 193–256 | New TikZ figure |

### Comment 21: OSS-Fuzz representativeness for automotive
**Professor:** "How representative are the training examples from OSS-Fuzz for automotive case studies?"

| Before | After |
|--------|-------|
| No discussion of representativeness | Added paragraph discussing: same C/C++ language, same API patterns, same fuzz driver structure (LLVMFuzzerTestOneInput). Acknowledged they don't capture automotive-specific safety semantics. |
| **File:** `chapters/implementation.tex` line 291 | New paragraph |

### Comment 22: Comparison with OSS-Fuzz-Gen
**Professor:** "Please also discuss how it compares to other work using OSS-Fuzz, such as oss-fuzz-gen"

| Before | After |
|--------|-------|
| No comparison | Added paragraph comparing: Google infra vs enterprise CI/CD, breadth (300+ projects) vs depth (automotive), cloud-only vs hybrid local/cloud. Referenced `\cite{ossfuzzgen}` |
| **File:** `chapters/implementation.tex` line 272 | New paragraph |

### Comment 23: Training libraries representative for automotive?
**Professor:** "FreeType, LibPNG, SQLite, zlib — are they representative for automotive?"

| Before | After |
|--------|-------|
| No discussion | Added: structural patterns (binary parsing, format validation, error handling) match automotive domain tasks. These libraries appear in automotive company repositories. |
| **File:** `chapters/implementation.tex` lines 275–277 | New paragraph |

### Comment 24: Fine-tuning absolute numbers
**Professor:** "Could you give absolute numbers for all of them, e.g. in a table?"

| Before | After |
|--------|-------|
| Only percentages in bullet list | Added table with columns: Configuration, Gen. Time, Tokens Used, Time Saved, Tokens Saved. Rows for base Qwen 7B, LoRA-172, LoRA-709, reference Qwen 32B |
| **File:** `chapters/implementation.tex` lines 326–344 | New table |

### Comment 25: Elaborate requirements
**Professor:** "provide a list to detail what the performance, cost, and security requirements are"

| Before | After |
|--------|-------|
| Requirements mentioned without specifics | Added three numbered lists: Performance (3 items: 15-45 min gen time, 60s fuzzing, async execution), Cost (3 items: <$0.50/driver, local model option, no per-seat licensing), Security (4 items: no source code to external services, container isolation, private network, compliance) |
| **File:** `chapters/implementation.tex` lines 361–381 | New requirement lists |

### Comment 26: Azure data clarification
**Professor:** "don't these requests contain source code? They would send corporate code to an external cloud service?"

| Before | After |
|--------|-------|
| No clarification | Added: "The prompts sent to the LLM contain only **public header files** (.h/.hpp), which define the public API surface. They do **not** contain proprietary source code, internal logic, or trade secrets." |
| **File:** `chapters/implementation.tex` line 395 | New paragraph |

### Comment 27: Attack model for Buildah
**Professor:** "This only isolates from attacks of other tenants, not against the cloud provider, right?"

| Before | After |
|--------|-------|
| No attack model discussion | Added: Two threat categories (tenant isolation + generated code containment). Acknowledged: "container isolation does not protect against a compromised cloud provider; this is addressed by the Azure Private Link architecture and contractual mechanisms" |
| **File:** `chapters/implementation.tex` lines 409–411 | New paragraph |

### Comment 28: Azure trust / zero-trust
**Professor:** "is Azure considered to be trusted? How does this relate with the zero-trust security model?"

| Before | After |
|--------|-------|
| No discussion | Added: "zero-trust refers to the **network perimeter** (no implicit trust for any network segment), while provider trust is established through **contractual mechanisms** (Azure's compliance certifications, DPA, GDPR commitments). Classified as 'trusted-but-verified' relationship." |
| **File:** `chapters/implementation.tex` line 453 | New paragraph |

### Comment 29: Workflow diagram instead of YAML
**Professor:** "This is a quite sub-optimal representation for a workflow. Use graph-based diagrams."

| Before | After |
|--------|-------|
| Raw YAML listing of GitHub Actions workflow | TikZ workflow diagram (Figure 4.2) showing: Git Push → Checkout → Login → Download Corpus / Pull Container → Build → cifuzz spark → Rebuild → Fuzz → Crash Reports + Coverage Data, with feedback loop. YAML moved to appendix. |
| **File:** `chapters/implementation.tex` lines 472–533 | New TikZ figure |

---

## Chapter 5 — Results

### Comment 30: yaml-cpp automotive connection
**Professor:** "Does this have any connection with automotive code?"

| Before | After |
|--------|-------|
| No automotive connection mentioned | Added: "YAML is widely used in build systems, CI/CD pipelines, and deployment configurations across the automotive industry. The yaml-cpp library is found in several public repositories within automotive organizations including CARIAD." |
| **File:** `chapters/results.tex` line 17 | New sentence |

### Comment 31: Additional targets automotive connection
**Professor:** "pugixml, jsoncons, fmt, spdlog, glm — Do they have any connection with automotive code?"

| Before | After |
|--------|-------|
| No automotive connection mentioned | Added: "These libraries appear in public repositories of automotive companies including CARIAD, where they serve roles such as XML configuration parsing (pugixml), JSON telemetry (jsoncons), log output formatting (fmt, spdlog), and mathematical transforms (glm)." |
| **File:** `chapters/results.tex` line 19 | New sentence |

### Comment 32: "results revealed" → table reference
**Professor:** "results, summarized in Table 5.1, revealed..."

| Before | After |
|--------|-------|
| "The results revealed..." | "The results, summarized in Table~\ref{tab:model_performance}, revealed..." |
| **File:** `chapters/results.tex` line 36 | Word change |

### Comment 33: Tests that didn't compile/run
**Professor:** "Were there any tests that did not compile or run? If so, list them in the table"

| Before | After |
|--------|-------|
| Only 3 unsuccessful models mentioned | Expanded to all 11 unsuccessful models in Table 5.2 with columns: Model, Parameters, Failure Stage, Reason |
| **File:** `chapters/results.tex` lines 78–101 | Expanded table |

### Comment 34: OSS-Fuzz citation fix
**Professor:** (citation was wrong)

| Before | After |
|--------|-------|
| Incorrect citation number | Changed to footnote URL pointing to OSS-Fuzz dashboard |
| **File:** `chapters/results.tex` line 59 | Citation fix |

### Comment 35: "did not produce usable output" → table ref
**Professor:** "as summarized in Table 5.2"

| Before | After |
|--------|-------|
| "This was one of the more important findings" | "as summarized in Table~\ref{tab:unsuccessful_models}. This was one of the more important findings" |
| **File:** `chapters/results.tex` line 76 | Added table reference |

### Comment 36: fmt section rewrite
**Professor:** "The results are counterintuitive. Rewrite to make your theorem clearer. Facts or assumption?"

| Before | After |
|--------|-------|
| Vague attribution to "simpler API surface" | Rewritten with concrete analysis: examined generated outputs, found larger models produced syntactically correct but semantically incorrect drivers (wrong include paths, missing template instantiations), while smaller models generated simpler but correct format-string-based drivers. Cited Zhang et al. for context. Explicitly labeled as hypothesis. |
| **File:** `chapters/results.tex` lines 137–141 | Rewritten paragraph |

### Comment 37: Llama~3 line break
**Professor:** "Llama~3 so to avoid splitting into two lines"

| Before | After |
|--------|-------|
| "Llama 3" | "Llama~3" (and "Qwen~1.5~7B") |
| **File:** `chapters/results.tex` line 117 | Tilde added |

### Comment 38: fmt table introduction
**Professor:** "Table 5.3 presents the results. They are counterintuitive."

| Before | After |
|--------|-------|
| "This result was counterintuitive" | "Table~\ref{tab:fmt_results} presents the results. They are counterintuitive." |
| **File:** `chapters/results.tex` line 137 | Sentence rewrite |

### Comment 39: pugixml table introduction
**Professor:** "The results for fuzz-testing code from the pugixml library are presented in Table 5.4."

| Before | After |
|--------|-------|
| No table introduction | "The results for fuzz-testing code from the pugixml library are presented in Table~\ref{tab:pugixml_results}." |
| **File:** `chapters/results.tex` line 145 | New sentence |

### Comment 40: jsoncons table introduction
**Professor:** "Table 5.5 summarizes the results for testing jsoncons, glm, and spdlog."

| Before | After |
|--------|-------|
| No table introduction | "Table~\ref{tab:additional_libraries} summarizes the results for testing jsoncons, glm, and spdlog." |
| **File:** `chapters/results.tex` line 166 | New sentence |

### Comment 41: Performance requirements specifics
**Professor:** "What are the specific requirements of performance and security?"

| Before | After |
|--------|-------|
| "meeting performance, cost, and security requirements" without specifics | Added back-reference to Chapter 4's requirement lists with specific thresholds |
| **File:** `chapters/results.tex` line 247 | New paragraph |

### Comment 42: Azure pricing date
**Professor:** "add the date of obtaining the pricing"

| Before | After |
|--------|-------|
| "at the time of our experiments" | "as of **July 2025** (the time of our experiments)" |
| **File:** `chapters/results.tex` line 254 | Date added |

### Comment 43: Safety constraints caveat
**Professor:** "none of your test cases seems to have safety-related constraint, don't they?"

| Before | After |
|--------|-------|
| No caveat | Added: "all generated drivers focused exclusively on functional correctness and coverage of the public API surface; none of the test cases in our evaluation incorporated safety-related constraints such as ISO 26262 ASIL requirements" |
| **File:** `chapters/results.tex` line 292 | New sentence |

### Comment 44: "at least to some extent"
**Professor:** "The results support this at least to some extent."

| Before | After |
|--------|-------|
| "The results support this." | "The results support this **at least to some extent**." |
| **File:** `chapters/results.tex` line 310 | Word change |

---

## Chapter 6 — Discussion and Conclusion

### Comment 45: Figure 6.1 edge labels
**Professor:** "Label the edges in this diagram."

| Before | After |
|--------|-------|
| PNG image with only "generated fuzz driver" labeled | TikZ diagram with ALL edges labeled: "Source code + public headers", "API request (headers + fuzzing instructions)", "Private connection (encrypted)", "LLM prompt (API context + instructions)", "Generated fuzz driver (C++ source code)", "Not used" for blocked internet path |
| **File:** `chapters/discussion_conclusion.tex` lines 47–100 | Replaced with TikZ |

### Comment 46: RQ1 novel insight
**Professor:** "already previous work has shown that LLMs can generate fuzz drivers. What is the novel insight?"

| Before | After |
|--------|-------|
| Just restated that LLMs can generate drivers | Added threefold novelty: (1) C/C++ libFuzzer-style drivers specifically, (2) systematic comparison across 14 models of varying size and specialization, (3) validation within enterprise CI/CD pipeline with real constraints |
| **File:** `chapters/discussion_conclusion.tex` lines 113–123 | Expanded paragraph |

### Comment 47: Performance = timing
**Professor:** "you actually mean timing. Please add these to the respective sections."

| Before | After |
|--------|-------|
| "performance requirement" undefined | Clarified in Ch4 (requirement lists), Ch5 (back-reference), Ch6: "15–30 min, up to 45 min for complex libraries" as performance requirement. Distinguished from coverage requirement. |
| **File:** `chapters/discussion_conclusion.tex` line 89 + Ch4/Ch5 | Multiple locations |

### Comment 48: Section 6.4 redundancy
**Professor:** "Section 6.4 contains a lot of redundancy."

| Before | After |
|--------|-------|
| ~40 lines repeating Ch4/Ch5 content | Reduced to ~15 lines with forward references instead of repetition |
| **File:** `chapters/discussion_conclusion.tex` lines 164–191 | Shortened |

### Comment 49: Citations UNECE/ISO
**Professor:** "citations problems"

| Before | After |
|--------|-------|
| Wrong citation numbers | Fixed: `UNECE Regulation~155~\cite{UNECER155}`, `ISO/SAE~21434~\cite{ISO21434}` |
| **File:** `chapters/discussion_conclusion.tex` line 220 | Citation fix |

### Comment 50: Cloud provider trust
**Professor:** "as long as you trust the cloud providers, or even in case of untrusted providers?"

| Before | After |
|--------|-------|
| "These maintain security boundaries" without caveat | Added: "provided the cloud provider itself is trusted. Our architecture relies on contractual trust (Azure compliance certifications) rather than technical guarantees against a compromised provider. For higher-assurance environments, an on-premises LLM deployment would eliminate this trust dependency entirely." |
| **File:** `chapters/discussion_conclusion.tex` line 234 | New sentence |

### Comment 51: Section 6.6 redundancy
**Professor:** "This Section 6.6 is extremely redundant."

| Before | After |
|--------|-------|
| ~30 lines repeating earlier content | Replaced with concise 4-item numbered contribution list (~10 lines) |
| **File:** `chapters/discussion_conclusion.tex` lines 217–227 | Shortened |

---

## Additional Changes (Not from specific comments)

| Change | File | Details |
|--------|------|---------|
| Remove all em-dashes (`---`) from prose | All chapters | Replaced with commas, "that is", parentheses, etc. |
| Fix `**RQ1**` markdown in LaTeX | methodology.tex | Changed to `\textbf{RQ1}` |
| Fix backtick `` `llvm-cov` `` in LaTeX | methodology.tex | Changed to `\texttt{llvm-cov}` |
| Fix "I" → "we" consistency | methodology.tex, literature.tex | Academic "we" throughout |
| Add tildes to cross-references | All chapters | `Chapter~5`, `Section~\ref{}`, `Table~\ref{}`, `Figure~\ref{}` |
| Add `align=center` to edgelabels | discussion_conclusion.tex | Required for `\\` in TikZ edge labels |
| Add AUTOSAR citation | introduction.tex | First mention now has `\cite{}` |
| Fix "OSS Fuzz" → "OSS-Fuzz" | introduction.tex | Hyphenation consistency |
| Add `amsmath` package | thesis.tex | Required for LoRA math equations |
| Add `ossfuzzgen` bib entry | literature.bib | For OSS-Fuzz-Gen comparison |
| Expand abbreviation list | thesis.tex | Added 12 missing abbreviations |
| Expand appendix | thesis.tex | Model summary, LoRA script, sample driver, prompt example |
| Add `.gitignore` | root | Exclude LaTeX build artifacts |

---

## Diagrams Summary

All 6 diagrams have **both TikZ (embedded in .tex) and Mermaid (in diagrams_mermaid.md)** versions:

| # | Figure | Layout | TikZ | Mermaid | Notation |
|---|--------|--------|------|---------|----------|
| 1 | Fig 1.2 — Fuzzing Process | Top-down | ✅ | ✅ | data=rect, process=rounded |
| 2 | Fig 3.1 — Technical Architecture | Top-down | ✅ | ✅ | data=rect, process=rounded, layers marked, contributions highlighted |
| 3 | Fig 3.2 — Enterprise Integration | Left-right | ✅ | ✅ | components=rounded, data=rect |
| 4 | Fig 4.1 — Eval Environment | Top-down | ✅ | ✅ | components=rounded, data=rect |
| 5 | Fig 4.2 — CI/CD Workflow | Top-down | ✅ | ✅ | data=rect, process=rounded |
| 6 | Fig 6.1 — Network Architecture | Top-down | ✅ | ✅ | components=rounded, data=rect, all edges labeled |

**Consistent notation across ALL diagrams:**
- Rectangular boxes = Data artifacts / data stores
- Rounded boxes = Processing steps / components
- No two data boxes directly connected
- No two processing boxes directly connected
- Contribution highlights (green/yellow) in Fig 3.1

---

## Color Note

The new TikZ diagrams use **subtle pastel fills** (light blue, light green, light orange, light yellow). This is standard practice in academic theses and makes diagrams more readable. The original PNG images did not use color. Using color in new diagrams while keeping some original PNGs (like the V-Model) in grayscale is acceptable — color adds clarity. If the professor prefers uniform grayscale, all `fill=` colors can be removed from the TikZ code.

---

## How to Switch to Mermaid Diagrams

See `diagrams_mermaid.md` for:
1. Complete Mermaid code for all 6 diagrams
2. Step-by-step export instructions (render at mermaid.live → export as high-res PNG)
3. Exact LaTeX replacement code for each figure (which lines to replace in which file)

---

## Content Accuracy Audit (Latest Session)

### Issue: Gemma 3 and Phi-4 Model Classification

**Problem:** Gemma 3 (all sizes) and Phi-4 were classified as "Code-specialized" in all tables and prose. This is factually incorrect:
- **Gemma 3** is Google's general-purpose model family (not CodeGemma, which is the code-specific variant)
- **Phi-4** is Microsoft's general-purpose reasoning model (not CodePhi)

An examiner who checks the original model documentation would immediately catch this error.

| Before | After |
|--------|-------|
| Gemma 3 27B: "Code-specialized" in all tables | Gemma 3 27B: "General-purpose" |
| Gemma 3 9B/4B: "Code-specialized" | Gemma 3 9B/4B: "General-purpose" |
| Phi-4 14B: "Code-specialized" | Phi-4 14B: "General-purpose" |
| Narrative: "code-specialized models outperform general-purpose" | Narrative: "model architecture and training quality matter more than size or specialization label" |

**Files changed:** `abstract.tex`, `implementation.tex` (table + prose), `results.tex` (summary + RQ2), `discussion_conclusion.tex` (interpretation + RQ2 + conclusion)

**Impact on thesis argument:** The revised argument is actually STRONGER and more defensible:
- Before: "code-specialized > general-purpose" (contradicted by data: CodeLlama/WizardCoder are code-specialized but failed)
- After: "training data quality + architectural recency > size or specialization label" (fully consistent with all data)

---

### Issue: Missing Citations for Factual Claims

| Claim | Before | After |
|-------|--------|-------|
| Tesla OTA updates (intro L13) | No citation | Added `\cite{TeslaOTA2019}` + bib entry |
| Chrysler 1.4M recall (intro L22) | No citation for recall number | Added `\cite{Greenberg2015}` + bib entry (Wired article + NHTSA number) |

---

### Issue: Inconsistent "saves 8 hours" Claim

| Location | Before | After |
|----------|--------|-------|
| methodology.tex L27 | "saves 8 hours" | "saves 2 to 8 hours" (consistent with range used elsewhere) |

---

### Issue: "15-30 minute window" Without Source

| Location | Before | After |
|----------|--------|-------|
| methodology.tex L231 | "15-30 minute window developers expect" (no source) | References Chapter 4 requirements: "CARIAD's CI/CD feedback window (see Chapter~4, Section~\ref{sec:impl_phase3})" |

---

### Issue: Last Remaining Em-Dash

| Location | Before | After |
|----------|--------|-------|
| methodology.tex L230 | "Vulnerability"—simply put | "Vulnerability, that is," |

---

### Issue: Aufgabenstellung Placeholder

| Before | After |
|--------|-------|
| Commented-out placeholder "Hier kommt die Aufgabenstellung hin" | Proper Aufgabenstellung with thesis topic, 6 tasks, and Bearbeitungszeitraum. User should replace with official version if required. |

**File changed:** `demoFile_aufgabe.tex`
