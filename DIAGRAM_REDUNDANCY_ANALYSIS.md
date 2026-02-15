# Diagram Redundancy Analysis

## Current Diagrams in Chapters 4-6

### Chapter 4 - Implementation
1. **Figure 4.1: Toolchain Architecture**
   - Shows: Local dev → Cifuzz Spark → LLMs → libFuzzer → Coverage/Crashes
   - Type: Complex flowchart
   - Status: UNIQUE - Shows implementation details

2. **Figure 4.2: Evaluation Pipeline**
   - Shows: Model Server → Generate → Compile? → Fuzz → Coverage → Store
   - Type: Workflow flowchart
   - Status: UNIQUE - Shows evaluation methodology

3. **Figure 4.4: Network Architecture with Azure Private Link**
   - Shows: CARIAD Network → Firewall → Azure Private Link → Azure OpenAI
   - Type: Network architecture
   - Status: ⚠️ **REDUNDANT WITH FIGURE 6.1**

### Chapter 5 - Experimental Results
1. **Figure 5.1: Model Performance Comparison**
   - Shows: Bar chart of coverage for 4 models (Gemma, Qwen, Phi, Yi)
   - Type: Data visualization (Mermaid xychart)
   - Status: UNIQUE - Shows results

### Chapter 6 - Discussion
1. **Figure 6.1: Enterprise Network Architecture with Azure Private Link**
   - Shows: Corporate Network → Firewall → Azure Private Link → Azure OpenAI
   - Type: Network architecture
   - Status: ⚠️ **REDUNDANT WITH FIGURE 4.4**

---

## REDUNDANCY DETECTED

### Figure 4.4 vs Figure 6.1 - DUPLICATE CONTENT

**Figure 4.4 (Chapter 4):**
```
CARIAD Internal Network:
- Source Code Repository
- CI/CD Runner
- Corporate Firewall

Azure Cloud:
- Azure Private Link (Private Endpoint)
- Azure OpenAI Service (GPT-4o)

Shows: Firewall blocks public internet, allows Private Link
```

**Figure 6.1 (Chapter 6):**
```
Corporate Network (CARIAD):
- GitHub Actions Self-hosted Runner
- Source Code Repository
- Corporate Firewall

Azure Cloud:
- Azure Private Link Endpoint
- Azure OpenAI LLM Service

Shows: Firewall blocks public internet, allows Private Link
```

**Verdict:** SAME DIAGRAM, slightly different styling but identical content

---

## Comparison with Initial Draft (Chapters 1-3)

**Note:** Cannot extract diagrams from PDF directly, but based on typical thesis structure:

**Chapter 1 (Introduction):**
- Usually: Problem illustration, context diagrams
- Unlikely to have: Implementation architecture, evaluation workflows

**Chapter 2 (Literature Review):**
- Usually: Conceptual frameworks, comparison tables
- Unlikely to have: Tool-specific architecture diagrams

**Chapter 3 (Methodology):**
- Usually: Research design, high-level approach
- **POSSIBLE:** May have methodology overview (but Figure 4.2 is more detailed)
- **POSSIBLE:** May have general architecture (but Figure 4.1 is implementation-specific)

**Conclusion:** Without seeing Ch 1-3 diagrams, Figures 4.1, 4.2, and 5.1 appear implementation/results-specific and unlikely to be in early chapters.

---

## RECOMMENDATION

### Keep (4 diagrams):
1. ✅ **Figure 4.1: Toolchain Architecture** - Implementation-specific, detailed
2. ✅ **Figure 4.2: Evaluation Pipeline** - Methodology workflow, unique
3. ✅ **Figure 5.1: Model Performance Comparison** - Results visualization, unique
4. ✅ **Figure 4.4 OR 6.1: Azure Network Architecture** - Keep ONE version only

### Remove (1 diagram):
1. ❌ **Remove duplicate** - Either Figure 4.4 or Figure 6.1

---

## WHICH TO KEEP: Figure 4.4 or 6.1?

### Option A: Keep Figure 4.4 (Chapter 4)
**Pros:**
- Appears in Implementation chapter where it's first discussed
- More detailed (shows CI/CD runner specifics, token limits)
- Top-to-bottom layout (TB) - may be clearer for technical flow

**Cons:**
- Less discussion context in Chapter 6

### Option B: Keep Figure 6.1 (Chapter 6)
**Pros:**
- Appears in Discussion where networking challenges are main point
- Simpler, cleaner design (left-to-right LR)
- More emphasis on security boundary

**Cons:**
- First mentioned in Chapter 4, so forward reference

### RECOMMENDED: Keep Figure 6.1, Remove Figure 4.4

**Rationale:**
- Chapter 6 discusses network architecture as "unexpected finding"
- Network security was the MAIN challenge (not just implementation detail)
- Figure 6.1 is simpler and clearer
- Chapter 4 can reference "see Figure 6.1 in Chapter 6 for architecture"

---

## FINAL DIAGRAM COUNT

**After removing redundancy:**
- Chapter 4: 2 diagrams (4.1, 4.2)
- Chapter 5: 1 diagram (5.1)
- Chapter 6: 1 diagram (6.1)

**Total: 4 unique diagrams** (down from 5)

All diagrams already in Mermaid format ✅

