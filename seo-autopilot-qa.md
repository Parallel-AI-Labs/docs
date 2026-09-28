# SEO Autopilot verification — September 27, 2026

The local feature was exercised against Parallel AI's real WordPress connection. One reviewed article was published. Application changes remain in the working tree; they have not been committed, pushed, or deployed.

## Published sample

[9 AI Lead Qualification Questions That Turn Chats Into Booked Calls](https://parallellabs.app/ai-lead-qualification-questions/)

- WordPress post ID: `18535`. Created as a draft, reviewed, then promoted by updating that same ID. The published slug resolves to exactly one post.
- Existing catalogue inspected: 449 WordPress posts, including drafts and other non-trash statuses. Topic novelty and article title/text overlap checks ran before publication. This is not a claim of exhaustive web-wide plagiarism detection.
- Three 1792×1008 brand images were generated and visually inspected. The selected image uses the company's white split layout, serif headline, blue accents, scenery, and logo. Featured media ID: `18534`.
- Public HTTP response, canonical URL, SEO title, description, featured-image dimensions, and descriptive alt text were verified. The public page was visually inspected.
- The sample received human editorial corrections before publishing. Unsupported generalizations were removed; illustrative examples are labeled; scoring rules handle hard blockers, missing information, and verified calendar availability. The final editorial gate passed. This sample must not be presented as evidence that every AI draft is publication-ready without review.
- The factual SPIN reference uses [Huthwaite's own explanation](https://www.huthwaiteinternational.com/sales-training/spin-selling), rather than treating the AI script as a research-validated conversion claim.

## Verified behavior

| Area | Evidence |
| --- | --- |
| Setup and persistence | Main-company CMS connection, optional keyword skip, campaign creation, and reload/resume exercised in the browser. |
| Review | Paginated queue, alternate-title selection, per-hook approval, regeneration, rejection, accurate completion count, Finish navigation, and campaign-menu Review entry exercised. |
| Request safety | Live API pagination and credential-free CMS response checked; repeated creation request reused the existing campaign; invalid UUID returned JSON 400. |
| Generation | Approved hook ran through real research, writing, SEO editing, canvas image generation, quality checks, and WordPress draft delivery. |
| Approval rules | Regression tests verify that unapproved and future-dated hooks are not generated, wrong weekdays are skipped, and another campaign's hooks cannot be changed. |
| Scheduling/refill | UTC weekday/date tests, same-day repeat prevention, shared database lock boundaries, and real low-queue refill. |
| Publication | Real WordPress draft-to-published update retained the existing post ID and image; failure contract checks ensure unsuccessful CMS calls do not claim success. |
| Quality | Deterministic title/body overlap detection plus separate source-aware editorial review. The live editorial review caught contradictory instructions and held the sample until corrected. |
| Other CMS | Ghost image/draft payload and Wix site-scoped Ricos payload covered by contract tests. No live Ghost, Wix, Shopify, or Webflow account publishing was performed. |

## Fixes made during takeover

- Preserve the autopilot marker when keywords are skipped; correctly consume keyword suggestion objects and constrain model-selected keywords to the requested queue.
- Select an existing brand kit automatically; add company facts and real source research to the article writer; recover from incomplete research-tool output.
- Exclude covered WordPress topics and synonyms with a separate novelty audit. Preserve a smaller useful plan instead of padding it with duplicate topics.
- Correct weekday numbering, future dates, campaign-specific date allocation, low-queue refill, and once-per-day delivery behavior.
- Add shared database locks for competing servers and idempotent campaign creation for client retries.
- Keep CMS drafts as drafts, propagate publishing failures, preserve collection status, and update an existing WordPress draft instead of creating a duplicate.
- Harden WordPress media requests, timeouts, cleanup, and alt text; wire Ghost featured images; correct Wix authentication, draft endpoint, rich-content payload, and required site settings. Wix's authentication contract follows its [official REST documentation](https://dev.wix.com/docs/api-reference/articles/rest-authentication/rest-api-authentication).
- Add paginated CMS/queue endpoints, resumable review URLs, visible recovery controls, auth-hydration handling, correct approval counts, and UTC labels.
- Fix the shared LoadingButton prop order so an explicit `disabled={false}` cannot override the loading lock.

## Automated checks

83 tests and 5 subtests passed across:

- `test_seo_autopilot_campaigns.py`
- `test_seo_autopilot_quality.py`
- `test_wix_rich_text.py`
- `test_onboarding_content_speed.py`
- `test_seo_credit_pricing.py`
- `test_brand_kit_builder_web_artifacts.py`

Tests used the guarded local database and mocked external calls. The real WordPress/LLM/image runs described above were separate live checks. Existing dependency deprecation warnings remain. Frontend `npm run type-check` and `git diff --check` passed. No frontend build/upload was run.

## Test state and release limits

- Test campaign: `915950a7-756e-4832-85e9-86fa3c674361` (`SEO Autopilot`). It is deliberately **manual**, with draft delivery selected, so this test cannot start unattended daily publication.
- The initial 30-hook plan was audited against the large existing catalogue; overlapping hooks were soft-rejected. A real refill produced five distinct hooks. After regeneration/approval/rejection UI tests, four pending hooks remain: one approved and three awaiting review. The campaign retains the partial-plan message explaining that it did not fill all 30 slots. The product now says “up to 30 distinct topics.”
- The existing migration was not changed or applied during this takeover. No direct schema changes were made.
- Deployment across all application/worker servers and post-deployment scheduler verification remain release steps. A local run is not proof that deployed workers have this code.
- Live publishing on non-WordPress providers remains unverified. Do not market those paths as end-to-end certified from this test.
- Generated content still benefits from review. Draft delivery remains the default; SEO scores and model editorial review cannot guarantee factual accuracy, originality across the entire web, search ranking, or commercial results.
