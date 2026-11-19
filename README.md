# SEC Filings Sentiment & Event Study

Extracts the MD&A section from 96 retail 10-K and 10-Q filings, scores it with the Loughran-McDonald financial sentiment dictionary, and tests whether that sentiment predicts abnormal stock returns.

**Portfolio:** [janvikgabani.github.io](https://janvikgabani.github.io)

## The result: no significant relationship

| Window | Correlation | p-value | n |
|---|---|---|---|
| 5-day | 0.119 | 0.247 | 96 |
| 10-day | 0.151 | 0.143 | 95 |
| 20-day | 0.122 | 0.244 | 93 |

All three correlations are weakly positive and **none is statistically significant**. On this sample, MD&A tone does not predict how the stock moves after filing.

That is the honest finding, and it is not a surprising one — it is roughly what semi-strong market efficiency predicts. By the time a 10-K is filed, its content is largely known from the earnings call weeks earlier. A sample of 96 filings across 4 retailers also has limited power to detect a small effect, so this is evidence of *no large effect*, not proof of none at all.

## Coverage

| | |
|---|---|
| Companies | Walmart, Costco, Target, Best Buy |
| Period | 2020–2025 |
| Filings | 96 (24 × 10-K, 72 × 10-Q) |
| MD&A extraction success | 96 / 96 |

## The hard part was the extraction

Getting MD&A out of a raw SEC submission is most of this project. Each `full-submission.txt` bundles several `<DOCUMENT>` blocks, the primary one is XBRL-laden HTML, and the section boundaries are inconsistent across filers. The extractor went through five versions:

| Version | Problem it fixed | Success |
|---|---|---|
| v1 | Baseline `Item 7. Management's Discussion` … `Item 7A. Quantitative` | 72 / 96 |
| v2–v3 | Costco still failing | 72 / 96 |
| v4 | Costco writes **`Item 7—Management's`** with an em dash, not a period | 95 / 96 |
| v5 | One Best Buy 10-Q matched only the table-of-contents entry, so a cross-reference filter was needed | **96 / 96** |

The remaining trap: the table of contents matches the same pattern as the real section. v5 rejects matches whose distance to the following `Item 7A` is too small to be a real section body.

## Method

1. **Download** — `sec-edgar-downloader` pulls 10-K and 10-Q filings per ticker.
2. **Isolate the primary document** from the multi-document submission.
3. **Strip HTML** with BeautifulSoup, dropping script, style and XBRL noise.
4. **Extract MD&A** between the section heading and the next item heading (see above).
5. **Score sentiment** as `(positive − negative) / total words`, using Loughran-McDonald — a finance-specific dictionary, because general-purpose lexicons misread words like *liability* and *crude* in filings. 354 positive and 2,355 negative terms.
6. **Abnormal returns** — company return minus SPY over the same window, at 5, 10 and 20 trading days after the filing date.
7. **Test** — Pearson correlation between sentiment and abnormal return.

## Run it

```bash
pip install -r requirements.txt
jupyter notebook notebooks/sec_mda_sentiment.ipynb
```

You need the **Loughran-McDonald master dictionary** as `LM_dictionary.csv` — it is not redistributed here. Download it from [sraf.nd.edu](https://sraf.nd.edu/loughranmcdonald-master-dictionary/) and place it beside the notebook.

The SEC requires a real contact email in the user-agent for its EDGAR endpoints; set yours in the `Downloader(...)` call.

## Files

| Path | What it is |
|---|---|
| `notebooks/sec_mda_sentiment.ipynb` | The full pipeline, including the extractor's debugging history |
| `results/filing_sentiment_and_returns.csv` | Per-filing sentiment scores, raw and abnormal returns at 5/10/20 days |

## Possible extensions

- Scale beyond 4 retailers — 96 filings is thin for detecting a small effect.
- Score *change* in sentiment against the same filer's previous filing, rather than its level, since the level is dominated by boilerplate.
- Use a shorter event window (1–3 days) where the market reaction to new information actually concentrates.
