# Image Analysis and Mermaid Diagram Mapping

## Purpose

The images (cicd.png, fuzzing.png, LLM fuzzing workflow.jpg) are from Chapters 1-3 of the Initial Draft.

The Mermaid diagrams in Chapters 4-6 should **MATCH THE CONTENT** of these images to ensure consistency.

---

## Image Files Restored

### 1. cicd.png
- **Size:** 1414x192 pixels
- **Location in PDF:** Page 21 of Initial_Draft_ch_1,2,3.pdf
- **Topic:** CI/CD Pipeline Workflow
- **Should match:** Chapter 4 diagrams related to CI/CD integration

### 2. fuzzing.png  
- **Size:** 1280x305 pixels
- **Location in PDF:** Page 22 of Initial_Draft_ch_1,2,3.pdf
- **Topic:** Fuzzing Evaluation Process
- **Should match:** Chapter 4 diagrams related to evaluation/fuzzing workflow

### 3. LLM fuzzing workflow.jpg
- **Size:** 1280x294 pixels
- **Location in PDF:** Page 32 of Initial_Draft_ch_1,2,3.pdf
- **Topic:** LLM Fuzzing Integration
- **Should match:** Chapter 4 diagrams related to LLM integration with fuzzing

---

## Current Mermaid Diagrams in Chapters 4-6

### Chapter 4 - Implementation

**Figure 4.1: Toolchain Architecture**
- Shows: Local dev → Cifuzz Spark → LLMs → Fuzzing → Results
- Likely matches: **LLM fuzzing workflow.jpg**

**Figure 4.2: Evaluation Pipeline**
- Shows: Generate → Compile → Fuzz → Coverage → Store
- Likely matches: **fuzzing.png**

### Chapter 5 - Results

**Figure 5.1: Model Performance Comparison**
- Shows: Bar chart of model results
- Likely: Unique to results chapter (not in Ch 1-3)

### Chapter 6 - Discussion

**Figure 6.1: Azure Network Architecture**
- Shows: Corporate network → Azure Private Link → OpenAI
- Likely: Unique to discussion chapter (not in Ch 1-3)

---

## Analysis Needed

To properly update the Mermaid diagrams, I need to:

1. **View the actual content of each image** to see what they depict
2. **Compare** with current Mermaid diagrams
3. **Identify** which Mermaid diagram corresponds to which image
4. **Update** Mermaid diagrams to match image content exactly

---

## Likely Mappings (To Be Confirmed)

Based on file names and topics:

| Image File | Likely Purpose | Probable Mermaid Match |
|------------|----------------|------------------------|
| cicd.png | CI/CD pipeline workflow | Figure 4.4 (was removed) or new diagram needed |
| fuzzing.png | Fuzzing evaluation process | Figure 4.2: Evaluation Pipeline |
| LLM fuzzing workflow.jpg | LLM + fuzzing integration | Figure 4.1: Toolchain Architecture |

---

## Next Steps

1. **Examine image contents** (need to view them visually or extract text/OCR)
2. **Compare** with each Mermaid diagram in Ch 4-6
3. **Document** the exact mapping
4. **Update** Mermaid diagrams to match image content
5. **Verify** consistency between Ch 1-3 (images) and Ch 4-6 (Mermaid)

---

## Note

The user specifically stated:
> "THEY ARE IN CHAPTER 1,2,3. I ASKED THAT, THE DIAGRAMS IN 4,5,6 SHOULD HAVE THESE IMAGE CONTENT"

This means:
- Images are from early chapters (conceptual/overview)
- Mermaid diagrams in later chapters should show the SAME concepts
- This ensures narrative consistency throughout the thesis

**Goal:** Make sure the thesis tells a consistent story with matching diagrams.

