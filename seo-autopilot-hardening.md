# SEO Autopilot hardening — September 27, 2026

Implemented the first release-gate batch from [the improvement plan](seo-autopilot-improvement-plan.md). Changes remain in the working tree; they are not deployed or committed. Existing unrelated changes were preserved. No schema changes were introduced.

## Implemented

- WordPress delivery journals the correlation key and uncertain outcome **before** the remote write. A lost create response is reconciled against a unique matching remote post; retry updates that post. Zero or multiple matches hold instead of blindly creating another post. Remote IDs and uploaded media IDs are checkpointed. Journal transitions are tested after a real database reload, including JSON mutation tracking.
- Content delivery is serialized with a shared database lease. SEO planning, runs and review workers renew leases every 30 seconds; the default expiry is three minutes. Expired owners cannot renew. Stale planning and review states can be retried. This is recovery on retry, not a durable broker that automatically replays abandoned jobs.
- Held articles keep the campaign in an attention state. A paginated review queue exposes the saved article, blocker, fetched source receipts, claim excerpts, and brand image choices. Repair preserves the draft; Retry reruns delivery gates; Dismiss archives it. Tenant, campaign, content and image ownership are checked server-side.
- Editorial review independently fetches cited pages with redirect validation, time/size limits and bounded source count. Snapshots include URL, resolved URL, retrieval time, content hash and retrieved text. Claimed supporting excerpts must match fetched text, preserving punctuation and number signs. A research summary saying “verified” is insufficient.
- Generation checkpoints the draft before editorial work, allows up to two repairs, and reviews text before generating images. Unsupported drafts remain available for review. Review actions reuse the saved draft and images.
- Final novelty review combines exact text/title checks with model comparison against the closest twelve catalogue articles. Opposite tasks are not rejected merely for similar titles. Own recovered WordPress deliveries are excluded from duplicate comparisons.

## Verification

- **107 tests and 5 subtests passed**, with no expected failures, across campaigns, quality, recovery, CMS serialization, onboarding speed, credit pricing and brand artefacts. Existing dependency deprecation warnings remain.
- All eight deliberately selected editorial challenges produced their intended accept/hold outcome using the actual configured model. Supported company guidance and illustrative recommendations pass; fabricated claims, misleading research summaries, bad arithmetic and embedded instructions hold. This small benchmark is not a general accuracy estimate. Model explanations can still be inconsistent; outcomes and raw explanations are retained in the evaluation artifact.
- Two actual-model novelty challenges passed: synonym-rewritten instructions were held, and enable/disable articles remained distinct. This does not establish web-wide originality.
- A live WordPress fault test created one **unpublished** original draft, discarded its successful response, then retried. The retry updated the same post (`18539`); searching the correlation marker found exactly one post. The test draft was moved to recoverable Trash. No additional public article was published.
- Browser testing used a clearly identified QA copy of the earlier sample. Retry held the deliberate duplicate before publication. Image selection persisted. Repair found a source-attribution gap, revised once, fetched three source receipts, and returned to review without publication or new image generation. The fixture was archived after verification.
- The local API passed campaign-create idempotency, credential-free CMS pagination and malformed UUID checks. TypeScript checking passed. No build/upload was run.

## Remaining release work

1. Run the larger adjudicated editorial benchmark and a 30-article, multi-business quality/cost pilot. Include held drafts and failures in the denominator.
2. Improve candidate retrieval beyond lexical ranking; add explicit new/update/skip planning and unique-contribution briefs.
3. Add automatic visual rejection and rerendering, plus measured mobile/social crop tests. The implemented image selector is manual; automatic selection still defaults to the latest image.
4. Handle ambiguous remote outcomes through a guided operator reconciliation workflow. Current behavior intentionally holds when it cannot find a unique matching post. Media uploads can still leave an orphan after an upload response is lost.
5. Live-test failure recovery for other CMS adapters. WordPress is the only adapter tested live in this batch.
6. Deploy through the normal approved repository workflow, then repeat a live draft smoke test against the deployed servers. Keep the test campaign manual and draft delivery enabled.

The feature is materially safer, but neither this implementation nor the test set establishes that unattended publication is universally reliable.
