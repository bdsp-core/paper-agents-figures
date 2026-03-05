# Figure Review Agents

Multi-agent pipeline for reviewing scientific manuscript figures before journal submission. Uses Claude's vision capabilities to audit figures for narrative clarity, composition, color accessibility, typography, technical format compliance, caption completeness, statistical integrity, and cross-figure consistency.

## Agents

| # | Agent | Scope | What it checks |
|---|-------|-------|----------------|
| 1 | **Story** | per-figure | Main claim, self-sufficiency, chart type, noise, redundancy |
| 2 | **Composition** | per-figure | Data-ink ratio, Tufte principles, legends, annotations, layout |
| 3 | **Color** | all figures | Colorblind safety, palette quality, cross-figure color consistency |
| 4 | **Typography** | all figures | Font consistency, sizes, abbreviations, cross-figure uniformity |
| 5 | **Format** | per-figure | DPI, dimensions, file size, text rendering (needs metadata) |
| 6 | **Caption** | per-figure | Title, panel descriptions, error bars, sample size, stats |
| 7 | **Statistics** | per-figure | Plot type vs data, sample size visibility, axis truncation |
| 8 | **Consistency** | cross-figure | Final synthesis across all agents and all figures |

## Quick start

```bash
pip install anthropic Pillow
export ANTHROPIC_API_KEY=sk-...

# Review all PNGs in a directory
python run_review.py --fig-dir manuscript/figures/

# Review specific figures
python run_review.py figure1.png figure2.png figure3.png

# Run only specific agents
python run_review.py --agents story,composition,caption --fig-dir figs/

# Dry run (see what would be reviewed)
python run_review.py --fig-dir figs/ --dry-run

# Only main text figures (skip efigure*)
python run_review.py --fig-dir figs/ --main-only
```

Reports are saved to `reports/review_YYYYMMDD_HHMMSS.md`.

## Configuring for your journal

Edit `journal_config.json` to match your target journal's requirements:

```json
{
    "journal_name": "Nature Medicine",
    "Single-column width": "88 mm",
    "Double-column width": "180 mm",
    "Max figure height": "230 mm",
    "Resolution": ">=300 DPI",
    "Preferred formats": "EPS, PDF, or TIFF",
    "Color mode": "RGB",
    "Font": "sans-serif (Arial, Helvetica), 5-7 pt",
    "File size limit": "10 MB per figure"
}
```

Or set the `JOURNAL_CONFIG_PATH` environment variable to point to a config file elsewhere:

```bash
JOURNAL_CONFIG_PATH=/path/to/my_journal.json python run_review.py --fig-dir figs/
```

## Pipeline architecture

The pipeline runs in three phases:

1. **Phase 1** (priority 1): Per-figure agents run independently on each figure
2. **Phase 2** (priority 2): Hybrid agents receive all figures simultaneously for cross-figure checks
3. **Phase 3** (priority 3): The consistency orchestrator receives all figures + all prior agent reports for final synthesis

## Requirements

- Python 3.10+
- `anthropic` Python SDK
- `Pillow` (for image metadata and downscaling)
- `ANTHROPIC_API_KEY` environment variable
