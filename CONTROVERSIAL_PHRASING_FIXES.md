# Controversial Phrasing Fixes Guide

## Overview

This document identifies all instances of controversial phrasing in your thesis that need to be changed. The main issue is language that suggests organizational conflicts or personal preferences rather than technical/research requirements.

---

## 🔴 CRITICAL ISSUE: Line 11 (Chapter 4)

### Current Text (PROBLEMATIC):

```latex
Before writing any code, we needed to select the right tools. This decision was not 
straightforward because we had constraints from multiple directions. The university 
wanted reproducible research. CARIAD needed enterprise compatibility. And we wanted 
tools that would work on the hardware we had available.
```

### Why This is Controversial:

1. **Implies organizational conflict:** "The university wanted X, CARIAD needed Y, we wanted Z" suggests three different agendas
2. **Sounds unprofessional:** Appears like you had different goals than your supervisors
3. **Personal vs. institutional:** "And we wanted" implies personal preference rather than research requirement
4. **Creates tension:** Reader may wonder why there are conflicting requirements

### RECOMMENDED CHANGE (Option 1):

```latex
Before writing any code, we needed to select the right tools. This decision was not 
straightforward because the solution needed to satisfy multiple requirements 
simultaneously: reproducibility for academic validation, enterprise compatibility 
for industrial deployment, and efficient execution on available hardware resources.
```

### Alternative Version (Option 2):

```latex
Tool selection required balancing several constraints. The research needed to be 
reproducible to meet academic standards, compatible with CARIAD's enterprise 
infrastructure for deployment, and efficient enough to run on available hardware.
```

### Why These Are Better:

- ✅ Focuses on technical requirements, not organizational politics
- ✅ Uses "the solution needed" instead of "X wanted, Y needed, we wanted"
- ✅ Professional, neutral academic tone
- ✅ No implied conflicts between stakeholders

---

## ⚠️ Issue 2: Line 25 (Chapter 4)

### Current Text:

```latex
First, we wanted to evaluate many different models, and running them through paid 
APIs would have been expensive.
```

### Problem:

- "we wanted" emphasizes personal preference
- Should focus on research objective

### RECOMMENDED CHANGE:

```latex
First, evaluating many different models through paid APIs would have been 
cost-prohibitive.
```

### Why Better:

- Passive construction removes personal emphasis
- Focuses on practical constraint, not desire

---

## ⚠️ Issue 3: Line 36 (Chapter 4)

### Current Text:

```latex
This merging step added complexity to our workflow but was necessary for the larger 
models we wanted to evaluate.
```

### Problem:

- "we wanted to evaluate" sounds like personal choice

### RECOMMENDED CHANGE:

```latex
This merging step added complexity to our workflow but was necessary for evaluating 
the larger models.
```

### Why Better:

- Removes personal want, states technical necessity
- More concise and professional

---

## ⚠️ Issue 4: Line 122 (Chapter 4)

### Current Text:

```latex
We wanted to know: does a 46B general model outperform a 7B code-specialized model?
```

### Problem:

- "We wanted to know" is too informal
- Should frame as research objective

### RECOMMENDED CHANGE:

```latex
This allowed us to investigate whether a 46B general model could outperform a 7B 
code-specialized model.
```

### Alternative:

```latex
A key research question was whether a 46B general model could outperform a 7B 
code-specialized model.
```

### Why Better:

- Academic phrasing for research question
- Professional tone

---

## ⚠️ Issue 5: Line 128 (Chapter 4)

### Current Text:

```latex
We needed consistent test targets to compare models fairly.
```

### Problem:

- "We needed" can sound subjective
- Should emphasize methodological requirement

### RECOMMENDED CHANGE:

```latex
Consistent test targets were required to ensure fair model comparison.
```

### Why Better:

- Passive voice emphasizes requirement, not personal need
- More academic tone

---

## ⚠️ Issue 6: Line 216 (Chapter 4)

### Current Text:

```latex
We chose this size specifically because we wanted to test whether fine-tuning could 
make small models competitive with larger ones.
```

### Problem:

- "we wanted to test" sounds like curiosity rather than research objective

### RECOMMENDED CHANGE:

```latex
This size was chosen to investigate whether fine-tuning could make small models 
competitive with larger ones.
```

### Why Better:

- Passive construction focuses on methodological decision
- "investigate" is more academic than "wanted to test"

---

## ⚠️ Issue 7: Line 286 (Chapter 4)

### Current Text:

```latex
Even if network connectivity worked, we needed to securely pass API keys to the 
container.
```

### Problem:

- Minor: "we needed" slightly informal for technical requirement

### RECOMMENDED CHANGE:

```latex
Even with network connectivity established, API keys needed to be securely passed 
to the container.
```

### Why Better:

- Passive voice emphasizes technical requirement
- More formal academic writing

---

## ⚠️ Issue 8: Line 429 (Chapter 4)

### Current Text:

```latex
The pipeline needed to handle these cases gracefully.
```

### Problem:

- Minor: Could be clearer that this was a design decision

### RECOMMENDED CHANGE:

```latex
The pipeline was designed to handle these cases gracefully.
```

### Why Better:

- Clarifies this was intentional design, not just a need
- More precise language

---

## Quick Reference Table

| Line | Current Issue | Recommended Change | Priority |
|------|---------------|-------------------|----------|
| 11 | "university wanted, CARIAD needed, we wanted" | "solution needed to satisfy multiple requirements" | 🔴 CRITICAL |
| 25 | "we wanted to evaluate" | "evaluating... would have been" | ⚠️ Recommended |
| 36 | "we wanted to evaluate" | "evaluating the larger models" | ⚠️ Recommended |
| 122 | "We wanted to know" | "This allowed us to investigate" | ⚠️ Recommended |
| 128 | "We needed" | "were required" | ⚠️ Recommended |
| 216 | "we wanted to test" | "to investigate whether" | ⚠️ Recommended |
| 286 | "we needed to" | "needed to be" | ⚠️ Recommended |
| 429 | "needed to" | "was designed to" | ⚠️ Recommended |

---

## Writing Style Guide

### ❌ AVOID (Controversial Patterns):

**Pattern 1: Organizational Attribution**
```
❌ The university wanted X
❌ CARIAD needed Y
❌ We wanted Z
```

**Pattern 2: Personal Preferences**
```
❌ We wanted to evaluate
❌ We wanted to know
❌ We wanted to test
```

**Pattern 3: Subjective Needs**
```
❌ We needed to do X
```

### ✅ USE INSTEAD (Professional Patterns):

**Pattern 1: Technical Requirements**
```
✅ The research required X
✅ The solution needed to satisfy X
✅ Enterprise deployment required Y
✅ Academic validation necessitated Z
```

**Pattern 2: Research Objectives**
```
✅ This allowed investigation of X
✅ The research investigated whether X
✅ A key objective was to determine X
```

**Pattern 3: Design Decisions**
```
✅ X was necessary to Y
✅ The design addressed X
✅ The methodology required X
```

**Pattern 4: Passive Construction**
```
✅ X was required
✅ Y needed to be established
✅ Z was designed to handle
```

---

## Complete Change Checklist

### File: `chapter4_implementation.tex`

- [ ] **Line 11** - Change "university wanted, CARIAD needed, we wanted" → "solution needed to satisfy requirements" (CRITICAL)
- [ ] **Line 25** - Change "we wanted to evaluate" → "evaluating... would have been"
- [ ] **Line 36** - Change "we wanted to evaluate" → "evaluating"
- [ ] **Line 122** - Change "We wanted to know" → "This allowed us to investigate"
- [ ] **Line 128** - Change "We needed" → "were required"
- [ ] **Line 216** - Change "we wanted to test" → "to investigate whether"
- [ ] **Line 286** - Change "we needed to" → "needed to be"
- [ ] **Line 429** - Change "needed to" → "was designed to"

---

## Summary

**Total Changes:** 8  
**File:** `chapter4_implementation.tex`  
**Priority:** 1 critical (Line 11), 7 recommended (all others)  
**Estimated Time:** 10-15 minutes  

**Impact:**
- Removes controversial organizational conflict language
- Establishes professional academic tone
- Focuses on technical/research requirements
- Maintains objectivity and neutrality

**Result:**
- Professional thesis suitable for FAU Erlangen submission
- No implied organizational tensions
- Clear, objective academic writing

---

## Action Plan

1. Open `chapter4_implementation.tex`
2. Use search function to find each line
3. Replace text with recommended version
4. Save file
5. Recompile thesis
6. Review for tone consistency
7. Done!

**Your thesis will sound more professional and avoid any controversy!** ✅
