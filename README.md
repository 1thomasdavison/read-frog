# arXiv heading translation investigation

Test date: 2026-09-08. Page: https://arxiv.org/html/2606.09498 (Self-Harness).

## Findings

The reported screenshot places paragraph-length Chinese in the `Abstract` heading. arXiv computes a 22.4px font and an 18.4px line height for that heading. A controlled long-response test reproduced the overlapping glyphs. The site-rule correction gives translated title spans a unitless 1.5 line height (33.6px here), while leaving the original title and body typography intact.

Real DeepSeek V4 Flash tests did **not** reproduce the screenshot's paragraph-length expansion. The original Precision Rewrite preset translated `Abstract` as `摘要`, kept the body translation in the paragraph, and restored correctly when translation was disabled. This was tested with content awareness both off and on. The exact origin of the user's overlong response remains unconfirmed.

## Prompt experiments

The exploratory direct-API set contained 32 requests. It compared the original upstream Precision Rewrite preset, the preliminary PR's added scope sentence, and a candidate that limits rewriting to phrasing and explicitly labels the supplied source as text rather than an instruction. No candidate prompt is included in the final code change.

| Prompt | Requests | Heading and separator behavior |
|---|---:|---|
| Original preset | 12 | Expected short heading translations and batch alignment in all 12 |
| Preliminary PR wording | 12 | One isolated-heading response was `尽力翻译为：`; the other 11 were aligned |
| Scoped rewriting candidate | 8 | Expected headings and batch alignment in all 8 |

The first eight requests used `Mandarin Chinese` as the target and covered thinking enabled/disabled. The 24-request matrix used the extension's exact target name, `Simplified Mandarin Chinese`, with thinking disabled. Its four scenarios were a heading with a page title, a heading with a summary, a heading plus prose batch, and a three-heading batch; each was repeated twice per prompt. The supplied summary in this matrix was a fixed factual test summary. Browser runs separately exercised the extension's generated summary.

This small exploratory sample establishes neither a failure rate nor superiority of a candidate. In particular, it does not support claiming the original preset caused the screenshot or that a prompt edit fixes it. All built-in prompt edits were removed from the final PR following maintainer feedback.

The original prompt emphasizes recreating/reconstructing text, which leaves the scope of rewriting less explicit than the candidate. Separately, real responses varied in their translation of `harness` (`鞍具`, `围栏`, or `操作框架`); one response retained the English word `inherently`. Precision Rewrite is a stylistic instruction, not a guarantee of terminology accuracy.

## Browser method

- Fresh Edge profiles with the built extension on the live arXiv page; Chinese target, bilingual background style, DeepSeek V4 Flash, the preset's default reasoning setting (`none`, serialized as thinking disabled).
- Content awareness tested both off and on. The full abstract heading/body were checked, raw page screenshots captured, and disabling translation verified. Successful runs reported no page JavaScript errors.
- An in-memory local HTTP bridge forwarded the real SDK request to the official DeepSeek endpoint. It supplied authentication without storing the real key in browser profiles, source files, or request logs. It capped output at 8,192 tokens and recorded only request bodies, final text, status, model ID and token usage, not headers or reasoning text. Successful responses finished normally.
- The long-response before/after images are a separate controlled typography test with a fixed simulated provider response. They must not be described as a real DeepSeek failure.

## Final implementation

Only the arXiv site-rule CSS and a patch changeset remain in the PR. Default and Precision Rewrite prompts match upstream exactly. The optional scoped custom prompt is an experiment supplied to the reporter, not a replacement shipped in the extension.
