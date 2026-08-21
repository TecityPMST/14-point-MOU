# 14-Point MOU Compliance Tracker

An internal, fully sourced compliance dashboard tracking implementation of the **14-point Islamabad Memorandum of Understanding** between the Islamic Republic of Iran and the United States of America, signed **17 June 2026**.

Each of the 14 articles is scored against a fixed status taxonomy and backed by a dated, tier-tagged evidence log. The dashboard is a single self-contained HTML file — no build step, no server, no dependencies.

> **Snapshot, not a feed.** The dashboard does not update itself. Every figure in it is true as of the "Last refreshed" date in its header and no later. As of the **21 Aug 2026** refresh (Day 65 of 60 — the window expired on 16 Aug without a final deal or an agreed extension; on 20 Aug the US Treasury Secretary said the sanctions campaign is intended to "collapse this regime", and General License X's nominal expiry falls today with no oil licence in force since its 7 Jul revocation): Breached 10 · At Risk / Contested 4 · Conditional 0. Open the dashboard for the current state rather than trusting this line, which is updated by hand.

---

## Repository contents

| Path | What it is |
|---|---|
| `MOU_Compliance_Tracker.html` | **Canonical dashboard.** Overwritten in place every refresh, so a bookmark stays current. |
| `index.html` | Byte-identical copy of the canonical file, kept for viewers that expect an `index.html`. |
| `MOU_Compliance_Tracker_YYYY-MM-DD_HHMMSGT.pdf` | Timestamped PDF of the current dashboard. **Only the latest is kept at root** — earlier ones move to `Archive/`. |
| `Archive/MOU_Tracker_YYYY-MM-DD.html` | One immutable HTML snapshot per refresh day. |
| `Archive/MOU_Tracker_YYYY-MM-DD_HHMMSGT.pdf` | One immutable PDF snapshot per run. Every copy is kept — this is the audit trail. |
| `_INSTRUCTIONS.md` | The operating spec the refresh runs against: trigger phrase, sourcing rules, article obligations, design tokens, workflow, quality checklist. |
| `mou_compliance_tracker.jsx` | Original React build. Superseded by the portable HTML; retained for reference only. |
| `14-point Islamabad Memorandum of Understanding … 17 Jun 2026.pdf` | Authoritative verbatim MOU text. The reference the compliance tests are drawn from. |
| `Chatham House — …` | Third-party analysis-tier reference material. |
| `_to_delete/` | Superseded root PDFs staged for deletion. Safe to empty. |
| `.github_token` | **Local only, never committed.** Fine-grained GitHub PAT used by the refresh to push. See *Publishing* below. |

### Naming and archive conventions

- Timestamps are **Singapore time (Asia/Singapore, UTC+8)**, format `YYYY-MM-DD_HHMMSGT`.
- Root holds exactly **one** timestamped PDF: the newest. Superseded ones are moved out, never deleted in place by the refresh.
- `Archive/` is append-only. The daily HTML snapshot (`MOU_Tracker_YYYY-MM-DD.html`) is overwritten if the same day is refreshed twice; the PDFs are not, so multiple runs on one day each leave their own PDF.
- The canonical HTML, `index.html` and that day's archive HTML must always be **byte-identical** (same checksum). A mismatch means a refresh failed partway.

---

## Viewing the dashboard

Open `MOU_Compliance_Tracker.html` (or `index.html`) in any browser. It builds its tiles and article cards with JavaScript at load time and needs no network access.

Interactive features: full-text search across articles and evidence, status tabs, section-tag filters, expand/collapse-all, collapsible per-article evidence logs, and a **Download PDF** button that switches to a print layout and opens the browser print dialog.

**Do not use the in-browser button to produce the repo's PDF.** A browser can't choose a save folder, and a naive HTML-to-PDF of the raw file renders blank because the content is JavaScript-generated. Use the refresh workflow's PDF step instead (below).

---

## Refresh workflow

Refreshes are run in **Claude Cowork** with this folder attached as a writable workspace, against the spec in `_INSTRUCTIONS.md`. Internet access is required — the data source is live web research, not a local corpus.

**Trigger:** say `update the MOU tracker` (or `run mou tracker`) in a Cowork session with the folder attached. The run then executes end-to-end without further prompting unless a precondition fails.

A run does the following:

1. **Resolve clocks.** Compute today's date, Day *N* of 60 from the 17 Jun 2026 signing, and the sub-clock dates.
2. **Load prior state.** Read the existing dashboard's `DATA` block to recover each article's status, status note, last-checked date and evidence.
3. **Research pass.** For each of the 14 articles, search for developments since its last-checked date, prioritising primary/official records and corroborating with press.
4. **Re-score.** Assess each article against the status taxonomy, write a one-line status note, update `lastChecked`, and prepend new evidence rows. The dated audit trail is preserved; only clearly superseded rows are trimmed.
5. **Update the data block.** Refresh statuses, the six status counts (must sum to 14), the executive digest and the clock values. The prior run's counts and date are captured into `PREV_COUNTS` / `PREV_LABEL` so each tile can show its change line.
6. **Rebuild and verify** the HTML, then write the canonical file, `index.html` and the archive snapshot.
7. **Generate the PDF** and write it to root and `Archive/`.
8. **Commit and push** to GitHub, after the integrity checks pass.
9. **Report** a change-log: status moves with rationale, notable new evidence, new UNVERIFIED flags, current counts, Day *N*/60, and the published commit SHA.

### Integrity checks — every build must pass

A truncated write fails silently and visibly breaks nothing until someone opens the file, so each build is verified **on disk** before it is published:

- the file ends with `</script></body></html>`;
- the `DATA` JSON parses;
- a **headless render produces all 14 article cards** — counted in a real browser context, never inferred from a successful write;
- `PREV_COUNTS` and `PREV_LABEL` are present in `DATA` *and* in the `const { … } = DATA;` destructuring, or the tile change-lines render blank;
- status counts sum to 14 and match the tiles;
- canonical HTML, `index.html` and the archive snapshot share one checksum.

Only after the canonical file passes are the copies and the PDF written.

### PDF generation

The dashboard renders its content at runtime, so the PDF is produced by **pre-rendering**: read the `DATA` block, statically expand all six tiles and all 14 cards with evidence logs open, then convert that static HTML with `weasyprint` (`pip install weasyprint --break-system-packages`).

Two things to keep in mind: weasyprint doesn't support the dashboard's `auto-fit` grid, so the print CSS pins the tiles to `repeat(6, 1fr)`; and articles whose evidence logs run to 70-plus rows exceed a page, so `break-inside: avoid` is applied only to short cards — otherwise long ones get pushed whole and leave near-empty pages.

### Publishing

The refresh commits and pushes automatically as Step 8, to `TecityPMST/14-point-MOU` on `main`.

A run touches five files locally — the canonical HTML, `index.html`, the day's archive HTML and two PDFs — but **only the two HTML files and the README go to GitHub**. `.gitignore` ignores the folder root by default (`/*`) and re-admits four paths by name, so PDFs, `Archive/`, `_to_delete/`, the source PDFs and the token can never enter a commit by accident. The repo stays under 1.5 MB per revision; the OneDrive folder remains the full archive.

Commit convention:

```
refresh: 2026-08-14 — Day 58/60, no status changes (B7/R6/C1)
```

**A push is only believed once verified.** `git push` reporting `Everything up-to-date` can mean the commit was already on the remote, not that this run put it there — so the run confirms `git ls-remote … main` matches local `HEAD` before reporting success.

### The token

Publishing reads a fine-grained GitHub PAT from `.github_token` at the folder root. It needs exactly two things:

- **Repository access:** Only select repositories → `14-point-MOU`
- **Permissions → Repository permissions → Contents:** Read and write

Everything else can stay at no access. `Metadata: Read-only` enables itself automatically.

**Rotation.** Fine-grained tokens expire. When one does, pushes fail with 401 and the run will say so, complete everything else, and leave the commit ready to push by hand. To replace it: generate a new token at *github.com/settings/personal-access-tokens*, overwrite `.github_token` with the new string, and delete the old token on GitHub.

**Handle it as a live credential.** The file is gitignored, but this folder is OneDrive-synced, so the token replicates to the cloud and to every device signed into the account. Keeping it scoped to this one repo with a short expiry is what keeps that exposure bounded. If the folder is ever shared, moved to a shared drive, or the repo made public to a wider audience, rotate the token first.

Manual push, if ever needed:

```
cd "C:\Users\TeoQingWei\OneDrive - Straits Developments Pte Ltd\Tecity 14-point MOU Compliance Tracker"
git push origin main
```

---

## Sourcing rules

These are non-negotiable and apply to every evidence row.

**Source tiers**, shown in the dashboard as coloured dots:

- **Official** — government and IGO primary records and readouts: OFAC / US Treasury, CENTCOM / DoD, White House, State Department, IAEA, UNSC, and official Iranian or mediator (Qatar, Pakistan, Oman) statements.
- **Press** — wires and major outlets: Reuters, AP, AFP, Bloomberg, CNN, NPR, Al Jazeera, PBS, CNBC, BBC, FT.
- **Analysis** — think tanks and specialist briefings: ISW, Soufan Center, Atlantic Council, Chatham House, Crisis Group, sanctions-law advisories.

**Rules:**

- **Wikipedia and any open-edit or crowdsourced encyclopedia are excluded as verification sources.** If a claim rests only on an excluded source, it is not asserted — it is recorded as **UNVERIFIED** with a note to re-source next run.
- **Conflict order is Official > Press > Analysis.** Where credible sources genuinely conflict, both are presented and the item is marked **contested**. Conflicts are never resolved silently.
- Every evidence row carries a **date, a one-line paraphrased fact, the source name, its tier, and a link** where one exists.
- Single-sourced or thinly sourced claims are flagged **UNVERIFIED**.
- **Paraphrase, don't reproduce.** Any quotation is under 15 words, at most one per source; article paragraphs are never copied.
- Framing is **factual and non-partisan** throughout.
- An article that yields nothing new keeps its prior status, gets `statusNote` set to "checked — no change", and has `lastChecked` updated. Movement is never invented.

### Status taxonomy

| Status | Meaning |
|---|---|
| **Confirmed** | Obligation verifiably implemented. |
| **In Progress** | Underway, clock running, or mechanism forming. |
| **Conditional / Deferred** | Contingent on the final deal; not yet actionable. |
| **Not Yet Triggered** | No mechanism or metric exists yet; nothing to verify. |
| **At Risk / Contested** | Implementation disputed, stalled, or partially reversed. |
| **Breached / Violated** | A core commitment has been actively broken. |

### Key dates

| Clock | Date |
|---|---|
| Signing (Day 0) | 17 Jun 2026 |
| Final deal due (60 days, extendable by mutual consent) | ≈ 16 Aug 2026 |
| Blockade full lift; Hormuz pre-war volume (30-day sub-clocks) | ≈ 17 Jul 2026 |
| Toll-free Hormuz passage window ends | ≈ 16 Aug 2026 |
| OFAC General License X (oil waiver) authorized through | 21 Aug 2026 |
| Force withdrawal from Iran's proximity | 30 days after the final deal — clock not started |

---

## Maintainer notes

- **Article summaries are evergreen by design.** Each `summary` field states the obligation, its clock and the compliance test only. All dated and current-state assertions belong in `statusNote` and the evidence log, which are rewritten every run. Keeping dated claims out of summaries stops them rotting between refreshes.
- **Don't run two refreshes concurrently.** Overlapping runs race on the same files and can leave two timestamped PDFs at root, one of them built from a superseded data block. If a scheduled refresh exists, check its fire time before triggering a manual one.
- **Trust the disk, not the write.** Confirm renders and checksums against the files as saved. The PDF is built independently of the browser render, so a broken HTML file can still yield a plausible-looking PDF.

---

## Disclaimer

This tracker is an internal research and monitoring aid. It is **not** legal, financial, compliance or investment advice, and it carries no affiliation with, endorsement by, or authority from any government, international organisation or party to the MOU.

Assessments are editorial judgements applied to openly reported information and may be incomplete, contested or overtaken by events between refreshes. Evidence entries are **paraphrased summaries** of source reporting, not quotations; the linked sources are authoritative and this file is not. Anything relied on for a decision should be verified against the primary record.

Third-party documents stored alongside the tracker for reference remain the copyright of their publishers and are included here for internal use only. Check their licensing before this repository is shared beyond the team or made public.
