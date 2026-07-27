# JOURNAL

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/11

**Issue title:** Add support for ingesting a portfolio website URL

**Tier:** [x] Tier 2

**Problem summary:**
Right now the ingestion pipeline only pulls a candidate's profile data from their resume upload and their GitHub repos (READMEs and repo metadata) — there's no way to bring in content from a personal portfolio site, even though the `Profile` model already has a `portfolio_url` field sitting unused. This means bio text and project write-ups that only live on someone's portfolio page never make it into the vector store, so the review agent can't reference them when generating feedback. A successful fix adds a way to fetch that URL's HTML, strip out navigation/boilerplate, pull out the meaningful text (About/bio and project descriptions), and run it through the same chunk-and-embed flow the other sources use, storing it alongside the resume and GitHub content. It touches the ingestion layer (a new `web_parser.py` parser and a new method on `ingestion/pipeline.py`) and the profile API schema.

**Branch name:** 11-portfolio-url-ingestion

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

---

### Selection notes ("is this right for me?" checklist reasoning)

- **Scope fits a Tier 2 issue:** touches one new parser file plus small, well-scoped additions to the existing pipeline and schema — not a cross-cutting refactor.
- **Clear acceptance criteria:** the issue names the exact files to touch (`ingestion/parsers/`, `ingestion/pipeline.py`, `api/schemas/profile.py`), which matches the estimated 5–8 hour effort.
- **Follows an existing pattern:** `ResumeParser`/`ReadmeParser` + `IngestionPipeline.ingest_resume`/`ingest_readme` are direct templates to mirror, so the unknowns are mostly in the new part (fetching an arbitrary user-supplied URL safely) rather than in the whole pipeline shape.
- **New risk worth flagging early:** unlike the other sources, this one fetches a URL the user supplies, so SSRF protection (blocking internal/private addresses, restricting redirects) needs to be part of the implementation, not an afterthought.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/NadaFeteiha/pathreview/commit/7f7b868

**Reproduction summary:**
Since this issue describes a missing feature rather than a crash, I reproduced it by inspecting `main` along both paths a portfolio URL could take. I confirmed `ingestion/pipeline.py` has no `ingest_portfolio` method and `ingestion/parsers/` has no web/HTML parser, and that `api/routes/profiles.py` stores `portfolio_url` but never reads it back to fetch anything. I also found that `core/services/review_service.py::_run_ingestion_pipeline` references `profile.portfolio_url`, but only inside an explicit `# Placeholder: actual portfolio ingestion logic` block that fabricates a string instead of fetching real content. Full trail in `docs/issue-11-repro.md`.

**PLAN.md link:** https://github.com/NadaFeteiha/pathreview/blob/11-portfolio-url-ingestion/PLAN.md

**Walkthrough video (recommended):** Not recorded.

**Blockers or open questions:**
`core/services/review_service.py` has a review-generation-time placeholder for portfolio (and github/resume) data that this fix does not touch — see the Risks section in PLAN.md. Worth confirming with a mentor whether that's tracked as a separate issue or expected to be addressed later, since without it the ingested portfolio content isn't yet surfaced end-to-end in a generated review.
