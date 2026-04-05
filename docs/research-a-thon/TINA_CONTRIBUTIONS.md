# Individual Contribution Report — Tina Nguyen
## Project 4 Deliverables (Poster, Presentation, Video)

**Course:** CS 5542 Big Data Analytics & Applications
**Project:** Analytics Copilot
**Role:** Data & Frontend Lead
**Contribution (Project 4 deliverables):** 50%

---

## Contributions by File

### Research-A-Thon Poster
- `docs/research-a-thon/poster.pdf` *(contributed)* — Provided content and review for the Dataset Description section (table schema, row counts, ER relationships), Results & Evaluation section (evaluation numbers, LoRA comparison metrics), and Application section (UI feature descriptions, Streamlit Cloud deployment details). Reviewed and validated all accuracy figures, latency numbers, and dataset statistics against the actual system outputs.

### Presentation Slides
- `docs/research-a-thon/presentation.pdf` *(contributed)* — Provided content for the Demo slide (Chat Tab walkthrough, Monitor Tab feature list, live app URL), Key Results slide (metric descriptions and result interpretation), and Conclusion slide (future directions). Verified result figures match evaluation outputs from `scripts/evaluate.py` and `scripts/evaluate_adaptation.py`.

### Video
- `https://vimeo.com/1180190697` *(contributed)* — Reviewed the demo walkthrough section of the video for accuracy against the live application behavior, including the Pipeline Trace display, Monitor Tab metrics, and demo mode fallback. Confirmed the example query and response narrative matches real system output.

---

## Percentage Contribution

**50%** — Content review, data validation, and application section authorship across all three Project 4 deliverables.

---

## Tools Used

- **Anthropic Claude Code** (`claude-sonnet-4-6`) — Documentation review and content drafting
- **Streamlit Community Cloud** — Verified live application behavior for demo section accuracy

---

## Reflection

My Project 4 focus was ensuring the deliverables accurately represented the system we built — particularly the evaluation numbers and application behavior shown to a Research-A-Thon audience. The results section of the poster required careful cross-referencing with the evaluation harness output to confirm the 100% accuracy figure, the speedup, and the latency numbers were cited correctly. For the video, I reviewed the demo walkthrough against the actual running application to make sure the narrated example query and response matched what a viewer would see live.
