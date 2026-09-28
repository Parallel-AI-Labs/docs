# SEO Autopilot wizard review — September 28, 2026

## Changes

- Widened the wizard from MUI `md` (900px) to `lg` (1200px).
- Replaced Wix and Ghost SVG placeholders with the user-supplied PNG files and deleted the obsolete SVGs. Monochrome logos remain visible in both themes and retain their aspect ratio.
- Added a compact step counter on phones and two-column provider cards.
- Kept selected keywords above research results; bounded the results table height with a sticky header. Bulk suggestion selection preserves manually entered keywords. More than 30 starting keywords now produces a visible validation message and disables Continue.
- Clarified the weekly posting count, UTC delivery days, draft delivery, and what Launch does. Schedule controls are disabled during launch, and the draft checkbox has an accessible label.
- Review titles wrap fully, including on phones. Mobile review rows stack instead of requiring a horizontally scrolling desktop table. Approval acts on selected rows only.
- Kept long-running planning polls alive with a slower interval after the initial minute. Empty attention queues do not flash during their initial fetch. Partial-plan messages use proposed-title language and a warning when usable proposals exist.
- Limited loading feedback to the action being performed; approving proposals no longer makes Generate show a spinner.
- Clarified completion copy and corrected the Ghost API key label.

## Browser verification

Used the existing WordPress connection; no CMS post was created or published during this walkthrough.

- All five provider images loaded successfully. Wix and Ghost checked visually in light and dark themes; Wix and Ghost connection dialogs opened and closed without installation.
- Blog selection enables Continue and survives back navigation.
- Three custom keywords entered and retained across navigation.
- Live keyword research returned suggestions and available search metrics. Bulk selection/deselection preserved a custom keyword. Selecting 31 keywords displayed the limit and disabled Continue; reducing the selection re-enabled it.
- Delivery-day controls updated the weekly count. The last selected day could not be removed. Tone selection worked. Draft delivery remained enabled for the launched test.
- Measured 390 CSS-pixel layouts for connection, schedule, and populated review: document scroll width equaled viewport width. Review title styles were inspected: white-space normal, text-overflow clip, and multi-line heights.
- Created test campaign `8d0dd9d1-97d5-4483-a951-045612cd343d` (`SEO Autopilot 3`) with three keywords and Monday/Wednesday/Friday delivery. Launch disabled editable controls and transitioned to a stable planning screen. Reload resumed the same campaign.
- The live planner returned eight proposals on the selected weekdays. Approving exactly one changed the counters to one approved and seven awaiting approval. Continue reached the scheduled/draft completion screen.
- Separately checked completion for the existing manual QA campaign and verified Finish navigates to Campaigns.
- Restored the original dark-mode preference and reset the viewport override.

## Automated checks

`node --test frontend/tests/seoWizard.test.cjs`: seven passing component-transition regression tests, covering long-running polling beyond 100 checks, empty/error states, retaining partial results, selected-only approval, and selection reset on page changes.

Full frontend TypeScript check (`tsc --noEmit --skipLibCheck`) and `git diff --check` passed after the final edits.

## Remaining limits

The live campaign took approximately 10 minutes 37 seconds to finish planning (06:49:13 to 06:59:50 Los Angeles time). This is a real onboarding latency issue; visual changes do not solve it. The planner produced eight of a requested maximum of 30 proposals. This pass does not establish premium pricing, production deployment health, all-provider publishing, or article originality.

After explicit user approval, the test campaign and all eight unpublished proposals were soft-deleted. The campaign was also set to Manual. Database reload verified zero active proposals and a deleted campaign; every proposal had no external delivery ID. Existing campaigns and published articles were unchanged.
