# Evaluation plan

This is a small initial evaluation set. Run each query after connecting the API key and record the observed behavior below. No quality score is claimed until the runs are reviewed against the cited pages.

| # | Query | Expected behavior / checks |
|---|---|---|
| 1 | Compare the top 3 open-source vector databases for a startup building RAG products. | Recognize comparison intent; retrieve multiple distinct sources; findings cite source IDs; mention that “top” depends on workload and current feature/pricing details need verification. |
| 2 | Find 5 Indian B2B SaaS startups in HR tech and summarize their positioning. | Return company-related evidence with citations; avoid claiming exactly five if fewer are found; distinguish source claims from inferred positioning. |
| 3 | What are the pros and cons of a multi-agent architecture for customer support automation? | Surface tradeoffs from sources; avoid presenting one architecture as universally best; cite each major point. |
| 4 | Compare different approaches to adding memory in an AI support agent. | Retrieve evidence about distinct memory approaches; call out retention/privacy assumptions and source gaps. |
| 5 | ` ` (submit an empty question) | UI validation should prevent submission; API should return a clear 400 error for an empty question. |
| 6 | Use a blocked/unresolvable source page or a revoked API key. | The UI should show whether authentication/search or page extraction failed; it should not silently claim that there was no relevant information. Confirm fetched hosts are public and redirects to private/local addresses are rejected. |

## Run log

Fill this in during the demo. For each run, note whether search succeeded, whether pages were fetched or only snippets were used, whether the answer was grounded, and one issue to fix. Keep the source links with the notes.

| Query # | Search / fetch outcome | Grounding and usefulness | Failure or surprising behavior |
|---|---|---|---|
| 1 | Pending | Pending | Pending |
| 2 | Pending | Pending | Pending |
| 3 | Pending | Pending | Pending |
| 4 | Pending | Pending | Pending |
| 5 | Pending | Pending | Pending |
| 6 | Pending | Pending | Pending |

## Initial risk notes

- The hosted search model can still miss relevant pages or prefer weak sources.
- Page fetch failures retain cited search evidence, which is weaker than inspected page text; the UI labels them.
- Model synthesis can fail or return invalid JSON; source-ID validation and the extractive fallback handle common cases, but do not establish factual correctness.
- Confidence is heuristic and capped at Medium. A reviewer should open and check the sources before using the answer.
- The request in case 6 cannot directly inject an arbitrary URL into the fetch tool, but results are fetched over the server network. Validate resolved IPs and block private, loopback, link-local, and metadata destinations before production deployment.

## How this evaluation should improve

Run each query with and without a model key. Add expected evidence/source checks, score citation precision and answer completeness, record latency and failure mode, and keep any regressions as reproducible queries. Compare against a non-agentic search-plus-summary baseline before adding planning complexity.

