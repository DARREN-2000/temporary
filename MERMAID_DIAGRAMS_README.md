# Mermaid Diagram Codes - Quick Reference

## 📍 Your Question
**"Where are the Mermaid diagram codes which I have to get?"**

## ✅ Answer
**Right here in the repository!** I've created 2 files with the Mermaid code you need.

---

## 📥 Files to Download

### 1. Model Performance Chart (Figure 5.1)
**File:** `figure5_1_model_performance.mmd`
- Type: Bar chart (xychart-beta)
- Shows: 4 LLM models with coverage percentages
- Use for: Chapter 5, Figure 5.1

### 2. Network Architecture Diagram (Figure 6.1)
**File:** `figure6_1_network_architecture.mmd`
- Type: Flowchart (left-to-right)
- Shows: Azure Private Link network solution
- Use for: Chapter 6, Figure 6.1

### 3. Rendering Instructions
**File:** `MERMAID_RENDERING_GUIDE.md`
- Complete step-by-step guide
- Multiple rendering options
- Troubleshooting tips

---

## 🚀 How to Use (Super Simple)

### Step 1: Get the Code
Download the `.mmd` files from this repository.

### Step 2: Render to PDF

**Option A: Online (Easiest - No Installation)**

1. Go to: **https://mermaid.live**
2. Open `figure5_1_model_performance.mmd` in a text editor
3. Copy all the code
4. Paste into Mermaid Live Editor (left side)
5. Click "Actions" → "PNG" or "PDF"
6. Save as `model_performance.pdf`
7. Repeat for `figure6_1_network_architecture.mmd` → save as `enterprise_network.pdf`

**Option B: Command Line (Faster if you have npm)**

```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i figure5_1_model_performance.mmd -o model_performance.pdf
mmdc -i figure6_1_network_architecture.mmd -o enterprise_network.pdf
```

### Step 3: Use in Thesis

Place the PDFs in your thesis `bilder/` folder:
```
your-thesis/
└── bilder/
    ├── model_performance.pdf
    └── enterprise_network.pdf
```

Your LaTeX files already reference these!

---

## 📊 Preview of Diagrams

### Figure 5.1: Model Performance
```
Bar Chart showing:
- Gemma 3 27B:    45.06% (highest)
- Qwen 2.5 32B:   43.08%
- Phi 3.5 Mini:   34.26%
- Yi Coder 9B:     0.00%
```

### Figure 6.1: Network Architecture
```
Flow: Corporate Network → Firewall → Azure Private Link → Azure OpenAI
      (CARIAD)             |            (✓ Allowed)         (LLM)
                           |
                           ↓ (❌ Blocked)
                      Internet (Public)
```

---

## ⏱️ Time Needed

- **Download files:** 1 minute
- **Render Diagram 1:** 5 minutes
- **Render Diagram 2:** 5 minutes
- **Total:** ~10-15 minutes

---

## 🎯 Quick Links

- **Render Online:** https://mermaid.live
- **Mermaid Docs:** https://mermaid.js.org
- **Detailed Guide:** See `MERMAID_RENDERING_GUIDE.md`

---

## ✅ Checklist

- [ ] Download `figure5_1_model_performance.mmd`
- [ ] Download `figure6_1_network_architecture.mmd`
- [ ] Render Figure 5.1 to PDF
- [ ] Render Figure 6.1 to PDF
- [ ] Save PDFs to `bilder/` folder
- [ ] Compile LaTeX thesis
- [ ] Verify diagrams appear in PDF

---

## 💡 Pro Tips

1. Use **PDF format** for best LaTeX quality
2. Use **transparent background** for professional look
3. **Preview** before downloading to verify
4. Keep the **original .mmd files** for future edits

---

## 🎓 Summary

**Question:** Where are the Mermaid diagram codes?  
**Answer:** In this repository - 2 .mmd files ready to use!  

**Files:**
- ✅ figure5_1_model_performance.mmd
- ✅ figure6_1_network_architecture.mmd
- ✅ MERMAID_RENDERING_GUIDE.md (detailed instructions)

**Next step:** Render them at https://mermaid.live  
**Time:** 10-15 minutes  
**Result:** Beautiful diagrams in your thesis!

---

**Your Mermaid codes are ready! Happy rendering!** 🎨✨
