# SEO Autopilot: second-pass testing and improvement plan

**Implementation update:** The first hardening batch is now implemented and tested locally; see [results and remaining release work](seo-autopilot-hardening.md). The findings below preserve the earlier baseline.

September 27, 2026. This is a release-gap assessment, not a claim that the feature is perfect. No additional WordPress articles were created or published during this pass. Production implementation was not changed; this pass adds reproducible tests, an editorial challenge dataset, and this plan.

## New test evidence

Eight synthetic challenges ran against the actual configured editorial model. Seven produced the intended outcome; one exposed a source-provenance weakness. This small, deliberately selected set does **not** establish an 87.5% real-world accuracy rate.

| Editorial challenge | Expected | Observed |
| --- | --- | --- |
| Supported product guidance | Accept | Accept |
| Clearly labeled illustrative recommendation | Accept | Accept |
| Invented statistic contradicted by evidence | Hold | Hold |
| Contradictory routing and incorrect arithmetic | Hold | Hold |
| Instruction embedded in article telling reviewer to ignore errors | Hold | Hold |
| Fabricated study on an obvious `.test` domain | Hold | Hold |
| Real source URL, claim absent from the supplied accurate research summary | Hold | Hold |
| Real source URL, fabricated statistic described as verified in the research summary | Hold | **Accept — missed** |

The last two cases used the same deliberately fabricated 17.83% meeting uplift. Changing only the supplied research summary changed the verdict. The final review compares claims with a summary; it does not independently retrieve the cited source. This tests the review boundary, not the frequency with which the upstream researcher actually fabricates evidence. No synthetic claims were published.

Full model verdicts are in `seo-autopilot-editorial-eval-results.json`; reusable inputs are in `backend-flask/tests/fixtures/seo_editorial_challenges.json`.

Six additional code-level checks ran against mocked remote publishing and the guarded local database. Five unmet criteria were reproduced at their intended assertions:

| Check | Observed behavior | Consequence |
| --- | --- | --- |
| WordPress creates a post, but its response times out; publisher is retried | Two create requests | Retry can duplicate a remote post before its external ID is saved. This is a publisher-level simulation, not a live duplicate publication. |
| Plan has been `running` for two days with no active worker | Retry returns 409 | A crashed planning job can leave the user stuck. |
| Next run encounters an unresolved held draft | Campaign becomes healthy while the draft still has an error | Customer can lose the visible signal that intervention is needed. |
| Compare titles about enabling versus disabling SSO | Treated as duplicate | Conservative title similarity can reject a genuinely different reader task. |
| Compare synonym-rewritten versions of the same instructions | No body-overlap match | Exact word shingles do not establish semantic originality. The separate topic planner may catch some overlap; this test isolates the final text check. |
| A held article is encountered on another run | No publishing call | The publication hold itself remains effective. |

`tests/test_seo_autopilot_release_gaps.py` keeps the five unmet criteria as **strict expected failures** with explicit reasons. Running with `--runxfail` confirmed five assertion failures and one pass, rather than unrelated setup errors. These are unresolved gaps, not passing acceptance tests.

The combined targeted regression run finished with **84 passed, 5 strict expected failures, and 5 passing subtests**. The five expected failures are the release gaps above; they must not be counted as successful checks. No production schema or application code was changed during this pass.

## Recommended implementation order

### 1. Make publication and recovery dependable

Treat every article delivery as a durable operation with its own identity and states: pending, in progress, uncertain remote outcome, delivered, held, and failed. Reconcile an uncertain outcome against the CMS before retrying creation. Do not treat a timeout as proof that nothing was created. A stable remote correlation key is preferable to title matching; adapters must document how that identity survives each provider's API.

Use leased jobs with heartbeats and recoverable checkpoints. Retry an abandoned job after verifying that its lease expired. Resume from the last completed stage, preserving research, text, images and remote IDs instead of charging the customer to regenerate everything. Any required schema changes need a separately reviewed migration.

Give held content a persistent “Needs review” queue with its reason, source evidence, preview, repair, retry and dismissal actions. A healthy scheduler must not erase unresolved article problems. Notifications should be tied to a state transition rather than repeated on every run.

**Acceptance:** the five new reproductions are resolved or replaced by equivalent stronger tests; fault injection before/after each remote write and local checkpoint produces zero duplicate posts; worker restarts recover without losing approved work or rerunning completed expensive stages.

### 2. Ground claims in retrievable evidence

Store source URL, retrieval timestamp, exact supporting excerpt and source content hash for each consequential claim. Make the writer use that evidence; make the final reviewer inspect it rather than trust a prose assurance that a claim was verified. Check that a link resolves, but do not confuse an HTTP 200 with support for the claim. Apply the same discipline to company features, pricing, integrations, customer results and comparison claims.

Unverifiable numbers, quotations and results should be removed or held. Label recommendations and illustrative examples honestly. Prefer authoritative primary sources and company-approved facts; retain enough evidence for a customer to audit the statement.

**Acceptance:** all eight current editorial challenges pass, including fabricated-summary evidence; expand to at least 50 adjudicated cases covering stale sources, changed prices, citation mismatch, arithmetic, logical edge cases and source instruction attacks. A sampled review must confirm that the excerpts actually support their associated claims.

### 3. Repair drafts automatically before expensive image work

Use a bounded write → factual/logic review → repair → re-review loop. Allow at most two repair rounds, then hold with a clear explanation. Every repair must preserve verified facts and recheck any new claims. Run text quality checks before generating images. After final edits, check that the headline, SEO metadata and image text still agree.

This directly targets the earlier live sample: it looked polished and scored 86 for SEO, but needed multiple substantive edits. A higher keyword score alone would not have fixed it.

**Proposed beta target, not a current result:** at least 90% of a 30-article evaluation set need no substantive human rewrite, with a median final review time below five minutes. Measure all initial drafts, including held and failed ones, rather than counting only successful outputs.

### 4. Plan useful coverage, not a quota of vaguely different keywords

Build an inventory of existing article intent, outline, entity/topic coverage, age and performance. Retrieve the closest existing articles for each candidate and explicitly choose: new article, refresh, consolidate, or skip. Compare actual substance, not just titles or shared words. Test opposing tasks such as enable/disable to prevent over-aggressive rejection.

Each approved brief should identify its audience, reader task, existing alternatives, unique contribution, source plan, internal links and natural conversion step. Require some useful contribution beyond a generic restatement: an actual product walkthrough, usable template, worked example, approved firsthand observation, or original analysis. Do not manufacture customer evidence.

For a mature catalogue, updating five valuable pages may be better than inventing thirty new ones. Google likewise emphasizes useful, original value rather than generating many pages without adding value: [Google's guidance on generative AI content](https://developers.google.com/search/docs/fundamentals/using-gen-ai-content).

**Proposed beta target:** at least 80% of proposed briefs accepted by a knowledgeable reviewer, no known duplicate reader tasks, and a justified new/update/skip decision for every proposal. Evaluate both a fresh site and a site with hundreds of existing posts.

### 5. Evaluate images as product assets

The three earlier images looked professional, but the current runner selects the newest image; it does not establish that it is the best one. Add candidate checks for correct logo, headline accuracy, contrast, clipping, text density, brand rules and relevance to the article. Reject broken assets and re-render once, then hold if no candidate passes.

Check full-size, article-width, mobile and social-preview crops. Prefer shorter image headlines and less small text. Give users a clear image selector in article review. Keep versioned brand-kit references so a later logo change does not make an existing draft's provenance ambiguous.

**Acceptance:** zero clipped text or incorrect logos across a varied visual test set; human sign-off on mobile and social crops; chosen candidate and rejection reasons are inspectable. These responsive/crop tests were not performed in this second pass.

### 6. Prove customer value and sensible operating cost

Track time to first useful draft, brief acceptance, first-pass article acceptance, repair count, human edit time, end-to-end latency, failed-stage frequency and cost per **accepted** article. Include rejected planning work and failed generations in that cost. Show a realistic generation estimate and useful progress rather than presenting a long-running pipeline as instant.

Connect approved content to indexing, impressions, relevant clicks and qualified conversions using available analytics. Establish the baseline and attribution limits. Measure a supported beta across multiple businesses before promising revenue or ranking improvement. A polished single-company demo proves integration, not market demand.

**Proposed beta target:** 5–10 participating businesses, a 30-article quality benchmark across several article formats, and at least four weeks of workflow/retention observation. Search outcomes may require substantially longer; do not infer SEO efficacy from a four-week retention pilot.

## Release decision

The first implementation batch should combine durable publication/recovery, persistent held-content review, and evidence-backed editorial repair. These address demonstrated customer risks. The next batch should improve content planning and image selection. Then run the larger acceptance benchmark and supported beta.

Keep draft delivery as the default while those gates are being established. Enable automatic public publishing only for customers and content classes that have demonstrated reliable results. Do not broaden the public promise to every CMS until each adapter has live draft, update, image, retry and failure-path verification.
