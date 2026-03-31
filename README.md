# RenAIssance — OCR Pipeline for 17th-Century Spanish Printed Sources

A multi-stage OCR pipeline for early modern Spanish manuscripts, 
built for the GSoC 2026 RenAIssance test.

## Pipeline Overview

1. **PDF → Images** — PyMuPDF with automatic double-spread detection
2. **Computer Vision Preprocessing** — CLAHE contrast enhancement, 
   fastNlMeans denoising, Otsu binarization, upscaling to 300dpi+
3. **Tesseract OCR** — `spa_old` language model, PSM 4 for variable layouts
4. **Rule-Based Fixes** — Unicode long-s (ſ → s), ligature normalization 
   (ﬁ, ﬂ, ﬀ), noise line filtering
5. **Quality Gate** — Skips LLM for already-clean pages (avoids regression)
6. **Groq LLM Post-Correction** — llama-3.3-70b-versatile for noisy pages, 
   with retry logic and fallback
7. **Evaluation** — CER, WER, tokenized WER, n-gram precisions (1–4)

## Results (6 sources)

| Source | CER Before LLM | CER After LLM |
|---|---|---|
| Covarrubias - Tesoro lengua | 9.44% | 9.44% (skipped — already clean) |
| Guardiola - Tratado nobleza | 12.70% | 12.70% (skipped — already clean) |
| PORCONES.228.38 – 1646 | 20.79% | ~17% |
| Buendia - Instruccion | 48.36% | ~33% |
| PORCONES.748.6 – 1650 | 54.59% | ~54% |
| PORCONES.23.5 - 1628 | 110.39% | ~107% |

## Setup
```bash
pip install pymupdf pytesseract groq pillow opencv-python jiwer textblob python-docx
```

Add your Groq API key (free at console.groq.com) to Colab Secrets as `Grok_api`.

## Design Decisions

- **Quality gate**: Prevents LLM over-correction on sources where Tesseract 
  already performs well (CER < 10%). Without this, Covarrubias CER jumped 
  from 9% to 115% after Groq post-processing.
- **LLM evaluation** (future work): Standard CER/WER penalises valid archaic 
  spelling variants. A semantic LLM evaluator scoring accuracy, long-s 
  correction, preservation, and hallucination would better capture transcription 
  quality for 17th-century Spanish.
- **Next steps**: Replace Tesseract with Kraken (purpose-built for historical 
  document OCR) to address the fundamental limitation at higher CER sources.
