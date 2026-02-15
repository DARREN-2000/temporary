# Citation Mapping Guide

## Bibliography Update Summary

Your `literature.bib` file has been updated to include all citations used in the LaTeX chapters. The updated file is saved as `literature_updated.bib`.

---

## What Was Added

### New Citation Entries (9 total)

1. **`autosar`** - AUTOSAR automotive architecture standard
2. **`automotive_cybersecurity`** - Survey on automotive security and privacy
3. **`gpt4`** - GPT-4 technical report from OpenAI
4. **`llm4fuzz`** - LLM-based fuzzing research (also known as Fuzz4All)
5. **`azure_openai`** - Microsoft Azure OpenAI Service documentation
6. **`azure_private_link`** - Azure Private Link service documentation
7. **`ollama`** - Ollama local LLM inference server
8. **`unsloth`** - Unsloth efficient fine-tuning library
9. **`efficient_finetuning`** - Survey on parameter-efficient fine-tuning methods

### Alias Entries (7 total)

For compatibility with your existing bibliography, I added aliases for entries that already existed but with different keys:

| LaTeX Key (used in chapters) | Original Key | Entry |
|-------------------------------|--------------|-------|
| `aflplusplus` | `AFLPlusPlus` | AFL++ fuzzer |
| `libfuzzer` | `LibFuzzer2015` | libFuzzer documentation |
| `ossfuzz` | `OSSFuzzBugs2021` | OSS-Fuzz bug study |
| `lora` | `Hu2021` | LoRA fine-tuning method |
| `titanfuzz` | `TitanFuzz2022` | TitanFuzz LLM fuzzing |
| `iso_sae_21434` | `ISO21434` | ISO/SAE automotive cybersecurity standard |
| `unece_r155` | `UNECER155` | UNECE Regulation 155 |

---

## Complete Citation List

All citations used in Chapters 4, 5, 6:

| Citation Number | LaTeX Key | Title | Used In |
|-----------------|-----------|-------|---------|
| [1] | `aflplusplus` | AFL++ fuzzer | Ch 4 |
| [4] | `autosar` | AUTOSAR standard | Ch 5, 6 |
| [5] | `automotive_cybersecurity` | Automotive security survey | Ch 6 |
| [6] | `llm4fuzz` | LLM4Fuzz/Fuzz4All | Ch 4, 5 |
| [7] | `titanfuzz` | TitanFuzz | Ch 4, 5 |
| [8] | `ossfuzz` | OSS-Fuzz study | Ch 4, 5 |
| [12] | `lora` | LoRA fine-tuning | Ch 4 |
| [15] | `iso_sae_21434` | ISO/SAE 21434 | Ch 6 |
| [16] | `gpt4` | GPT-4 | Ch 6 |
| [17] | `azure_openai` | Azure OpenAI | Ch 4 |
| [18] | `azure_private_link` | Azure Private Link | Ch 6 |
| [19] | `ollama` | Ollama server | Ch 4 |
| [23] | `libfuzzer` | libFuzzer | Ch 4 |
| [25] | `unece_r155` | UNECE R155 | Ch 6 |
| [27] | `unsloth` | Unsloth library | Ch 4 |
| [28] | `efficient_finetuning` | PEFT survey | Ch 6 |

---

## Integration Instructions

### Option 1: Use Updated File (Recommended)

1. **Backup your current file:**
   ```bash
   cp literature.bib literature_backup.bib
   ```

2. **Replace with updated version:**
   ```bash
   cp literature_updated.bib literature.bib
   ```

3. **Compile thesis:**
   ```bash
   pdflatex thesis.tex
   biber thesis
   pdflatex thesis.tex
   pdflatex thesis.tex
   ```

### Option 2: Manual Merge

If you have custom entries in your `literature.bib`:

1. **Open both files:**
   - Your original: `literature.bib`
   - Updated version: `literature_updated.bib`

2. **Copy new sections from updated file:**
   - Section 7: CLOUD & INFRASTRUCTURE (3 entries)
   - Section 8: FINE-TUNING & OPTIMIZATION (2 entries)
   - Individual new entries: `autosar`, `automotive_cybersecurity`, `gpt4`, `llm4fuzz`

3. **Add alias entries** (if you want to keep using my LaTeX citation keys)

### Option 3: Update LaTeX Citations

If you prefer to use your existing bibliography keys, update the \cite{} commands in the LaTeX files:

| Replace in LaTeX | With Existing Key |
|------------------|-------------------|
| `\cite{aflplusplus}` | `\cite{AFLPlusPlus}` |
| `\cite{libfuzzer}` | `\cite{LibFuzzer2015}` |
| `\cite{ossfuzz}` | `\cite{OSSFuzzBugs2021}` |
| `\cite{lora}` | `\cite{Hu2021}` |
| `\cite{titanfuzz}` | `\cite{TitanFuzz2022}` |
| `\cite{iso_sae_21434}` | `\cite{ISO21434}` |
| `\cite{unece_r155}` | `\cite{UNECER155}` |

Then manually add only the 9 new entries listed above.

---

## New Bibliography Sections

The updated file includes two new sections:

### Section 7: Cloud & Infrastructure

Contains entries for:
- Azure OpenAI Service
- Azure Private Link  
- Ollama

These support the enterprise CI/CD integration discussion in Chapter 4.

### Section 8: Fine-Tuning & Optimization

Contains entries for:
- Unsloth library
- Efficient fine-tuning survey

These support the LoRA fine-tuning discussion in Chapter 4.

---

## Verification

After integration, verify all citations compile correctly:

```bash
# Compile with bibliography
pdflatex thesis.tex
biber thesis

# Check for citation warnings
grep "Citation.*undefined" thesis.log

# If any undefined citations, check:
# 1. Citation key spelling in LaTeX matches .bib
# 2. Entry exists in literature.bib
# 3. Biber ran successfully
```

### Expected Output

No warnings like:
- `LaTeX Warning: Citation 'xyz' on page N undefined`
- `Package biblatex Warning: Please (re)run Biber`

If you see warnings, run the full compilation cycle again.

---

## Citation Style Notes

The updated bibliography follows your existing conventions:

**Formatting:**
- arXiv papers: `howpublished = {arXiv:XXXX.XXXXX}`
- Conference papers: `booktitle = {CONFERENCE}`
- Technical documentation: `@misc` with `url` and `note`

**Naming:**
- Tools/Projects: Capitalized names (AFL++, Ollama, Azure)
- Papers: Title case
- Standards: Acronyms (ISO, UNECE, AUTOSAR)

**Fields:**
- All entries have: author, title, year
- Technical docs: Added `note` field for description
- Web resources: Included `url` field

---

## Troubleshooting

### Issue: "Citation 'autosar' undefined"

**Cause:** New entry not in your literature.bib

**Solution:** Copy the `@misc{autosar,...}` entry from `literature_updated.bib` to your `literature.bib`

### Issue: "Multiple definitions for 'lora'"

**Cause:** Both `Hu2021` and `lora` exist (aliases)

**Solution:** This is intentional for compatibility. Biber will use the first one it finds. If you get warnings, you can remove the alias entries and update LaTeX to use original keys.

### Issue: Biber fails with "Missing field"

**Cause:** Required fields missing in bibliography entry

**Solution:** All entries in `literature_updated.bib` have required fields. If using manual merge, ensure you copy complete entries.

---

## File Comparison

**Original literature.bib:**
- Entries: ~40
- Sections: 6
- Size: ~8 KB

**Updated literature_updated.bib:**
- Entries: ~56 (includes 7 aliases + 9 new)
- Sections: 8 (added Cloud/Infrastructure, Fine-Tuning)
- Size: ~14 KB

---

## Final Checklist

Before compiling:

- [ ] Backup original literature.bib
- [ ] Choose integration option (1, 2, or 3)
- [ ] Update or replace literature.bib file
- [ ] Run complete compilation cycle
- [ ] Check for citation warnings
- [ ] Verify all citations appear in PDF
- [ ] Check References section formatting

---

## Complete Bibliography Example

Here's what a properly formatted citation looks like in both files:

**In LaTeX (chapter4_implementation.tex):**
```latex
LibFuzzer integrates directly with the LLVM toolchain \cite{libfuzzer}.
We used LoRA for efficient fine-tuning \cite{lora}.
```

**In literature.bib:**
```bibtex
@misc{libfuzzer,
  author = {Serebryany, Kostya and {LLVM Project}},
  title  = {{libFuzzer} -- a library for coverage-guided fuzz testing},
  year   = {2024},
  url    = {https://llvm.org/docs/LibFuzzer.html}
}

@misc{lora,
  author       = {Hu, Edward J. and others},
  title        = {{LoRA}: Low-Rank Adaptation of Large Language Models},
  year         = {2021},
  howpublished = {arXiv:2106.09685}
}
```

**In compiled PDF References section:**
```
[12] Edward J. Hu et al. LoRA: Low-Rank Adaptation of Large Language 
     Models. arXiv:2106.09685, 2021.

[23] Kostya Serebryany and LLVM Project. libFuzzer – a library for 
     coverage-guided fuzz testing. https://llvm.org/docs/LibFuzzer.html, 2024.
```

---

## Summary

**Files created:**
- `literature_updated.bib` - Complete bibliography with all citations

**Entries added:** 16 total
- 9 new entries (autosar, gpt4, azure services, etc.)
- 7 alias entries (for compatibility)

**Integration time:** ~5 minutes

**Your thesis bibliography is now complete!** ✅
