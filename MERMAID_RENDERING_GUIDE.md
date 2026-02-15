# Mermaid Diagram Rendering Guide

## Overview

This guide explains how to render the Mermaid diagram codes into PDF/PNG images for inclusion in your LaTeX thesis.

## Files to Render

1. **figure5_1_model_performance.mmd** - Model performance bar chart
2. **figure6_1_network_architecture.mmd** - Enterprise network architecture diagram

## Rendering Methods

### Method 1: Online (Mermaid Live Editor) - EASIEST ⭐

**Best for:** First-time users, quick results, no installation needed

**Steps:**

1. **Go to Mermaid Live Editor**
   - URL: https://mermaid.live
   - Opens in your browser

2. **Load Diagram Code**
   - Open the `.mmd` file in a text editor
   - Copy all the code
   - Paste into the left panel of Mermaid Live

3. **Preview**
   - Diagram appears automatically on the right side
   - Adjust window if needed

4. **Download**
   - Click the "Actions" button (top right)
   - Select "PNG" or "PDF"
   - Choose download location
   - Save with appropriate name:
     - `model_performance.pdf` (for Figure 5.1)
     - `enterprise_network.pdf` (for Figure 6.1)

5. **Repeat for Second Diagram**
   - Clear the editor
   - Paste second diagram code
   - Download again

**Time:** 5 minutes per diagram

### Method 2: Command Line (mermaid-cli) - AUTOMATED

**Best for:** Developers, batch processing, automation

**Prerequisites:**
- Node.js installed (https://nodejs.org)
- npm available

**Installation:**
```bash
npm install -g @mermaid-js/mermaid-cli
```

**Rendering:**
```bash
# Navigate to directory with .mmd files
cd /path/to/mermaid/files

# Render Figure 5.1
mmdc -i figure5_1_model_performance.mmd -o model_performance.pdf

# Render Figure 6.1
mmdc -i figure6_1_network_architecture.mmd -o enterprise_network.pdf
```

**Advanced options:**
```bash
# High resolution PNG
mmdc -i figure5_1_model_performance.mmd -o model_performance.png -s 2

# Transparent background
mmdc -i figure6_1_network_architecture.mmd -o enterprise_network.pdf -b transparent

# Custom dimensions
mmdc -i figure5_1_model_performance.mmd -o model_performance.pdf -w 1600 -H 900
```

**Time:** 1 minute per diagram (after installation)

### Method 3: Mermaid.ink API

**Best for:** Quick PNG exports, sharing diagrams

**Steps:**

1. **Encode Diagram**
   - Go to https://mermaid.ink
   - Paste your Mermaid code
   - Click "Create diagram"

2. **Download**
   - Right-click on generated image
   - Save image as PNG
   - Convert to PDF if needed

**Time:** 3 minutes per diagram

## Quality Settings

### Recommended Settings

**For LaTeX Thesis:**
- **Format:** PDF (best quality, scalable)
- **Background:** Transparent
- **Resolution:** 300 DPI minimum
- **Scale:** 2x (for high-DPI displays)

**Alternative PNG Settings:**
- **Format:** PNG
- **Resolution:** 1600x900 px minimum
- **DPI:** 300
- **Background:** White or transparent

### Output Quality

**Figure 5.1 (Bar Chart):**
- Minimum width: 1200px
- Recommended: 1600px
- Aspect ratio: 16:9
- Text should be crisp and readable

**Figure 6.1 (Flowchart):**
- Minimum width: 1400px
- Recommended: 1600px
- Aspect ratio: 3:2
- Arrows and text must be clear

## Integration with LaTeX

### File Placement

Place rendered PDFs in your thesis directory:

```
your-thesis/
├── thesis.tex
├── bilder/
│   ├── model_performance.pdf       ← Figure 5.1
│   └── enterprise_network.pdf      ← Figure 6.1
└── chapters/
    ├── chapter5_results.tex
    └── chapter6_discussion.tex
```

### LaTeX Code (Already Done)

Your chapter files already have the correct references:

**chapter5_results.tex:**
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.85\textwidth]{bilder/model_performance.pdf}
    \caption{Model Performance Comparison...}
    \label{fig:model_performance}
\end{figure}
```

**chapter6_discussion.tex:**
```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.85\textwidth]{bilder/enterprise_network.pdf}
    \caption{Enterprise Network Architecture...}
    \label{fig:enterprise_network}
\end{figure}
```

### Compilation

After placing PDFs:

```bash
pdflatex thesis.tex
biber thesis
pdflatex thesis.tex
pdflatex thesis.tex
```

## Troubleshooting

### Issue: Diagram doesn't render

**Solution:**
- Check Mermaid syntax (copy exactly from .mmd file)
- Ensure no extra spaces or characters
- Try a different browser
- Use Mermaid Live Editor for validation

### Issue: Text is too small

**Solution:**
- Increase scale factor: `mmdc -s 3`
- Or render larger in Mermaid Live
- Adjust LaTeX width parameter if needed

### Issue: Colors don't match

**Solution:**
- Keep default theme (base)
- Don't modify color codes
- Ensure PDF preserves colors

### Issue: PDF not found in LaTeX

**Solution:**
- Check file paths (case-sensitive)
- Verify files are in `bilder/` directory
- Ensure filenames match exactly:
  - `model_performance.pdf` (not `model_performance_chart.pdf`)
  - `enterprise_network.pdf` (not `network_architecture.pdf`)

### Issue: mermaid-cli command not found

**Solution:**
```bash
# Install globally
npm install -g @mermaid-js/mermaid-cli

# Or use npx (no installation)
npx @mermaid-js/mermaid-cli -i figure5_1.mmd -o output.pdf
```

## Verification Checklist

Before using in thesis:

- [ ] Both diagrams rendered successfully
- [ ] PDF files are not empty (>10 KB each)
- [ ] Open PDFs to verify appearance
- [ ] Text is readable and crisp
- [ ] Colors are appropriate
- [ ] No clipping or cut-off elements
- [ ] Transparent background (if using)
- [ ] Files named correctly
- [ ] Files in correct directory (`bilder/`)
- [ ] LaTeX compilation includes diagrams
- [ ] Figures appear in final PDF
- [ ] Figure captions match content

## Alternative Tools

### Diagram.codes
- URL: https://diagram.codes
- Supports Mermaid
- Online editor

### Kroki
- URL: https://kroki.io
- API for diagram generation
- Supports many formats

### VS Code Extension
- Extension: "Mermaid Preview"
- Edit and preview in IDE
- Export to PNG/SVG

## Tips for Best Results

1. **Use PDF format** whenever possible for LaTeX
2. **Test render** before final use
3. **Keep original .mmd files** for future edits
4. **Version control** your diagram sources
5. **Transparent background** for professional look
6. **Check in PDF reader** before thesis compilation
7. **Have backup PNG** in case PDF has issues

## Summary

**Easiest method:** Mermaid Live Editor (https://mermaid.live)
**Fastest method:** mermaid-cli command line tool
**Best quality:** PDF format, 2x scale, transparent background

**Time investment:**
- First time: 10-15 minutes (including setup)
- Subsequent: 5 minutes per diagram

**Result:** Professional diagrams ready for your FAU Erlangen thesis!

## Support

For issues with:
- **Mermaid syntax:** https://mermaid.js.org/intro/
- **mermaid-cli:** https://github.com/mermaid-js/mermaid-cli
- **LaTeX integration:** See thesis template documentation

Good luck with your thesis! 🎓
