# Contributing

Thanks for considering a contribution. This project is intentionally narrow in scope — please read the guidelines below before opening an issue or PR.

## Scope — what contributions are welcome

- **New transcript adapters** for additional formats (Otter, Zoom, VTT/SRT, YouTube transcripts, etc.)
- **Bug fixes** in existing adapters or the verdict pipeline
- **Documentation improvements** (clarity, examples, translation)
- **Example transcripts** demonstrating specific use cases — especially ones that anchor a calibration point not currently covered by `examples/01-strategy-meeting.md`, `examples/02-due-diligence.md`, or `examples/03-interview.md`
- **Translation** of documentation into other languages
- **Calibration runs** (run the skill on your own transcripts and submit the `tests/calibration/run-N/` outputs)

## Scope — what contributions are NOT welcome

- Changes to the **verdict rule φ** (architectural invariant I7 — verdict must remain deterministic and external to LLM substrate). Requires a successor spec.
- Adding **new dimensions** beyond Relevance, Coverage, Ordering, Robustness. Requires a successor spec.
- Removing the **diagnosis-authorization separation** (invariant I2). The LLM must never decide the final verdict.
- Wrapping the skill in a **cloud service** that changes the local-only execution model. The privacy posture is intentional.
- Changes that **dilute the Hook-as-Skill positioning** — e.g., reframing as a generic "AI meeting tool" or "summarization tool".

For any architectural change, open an issue first and reference [`docs/spec/001-mvp-conversation-audit.md`](docs/spec/001-mvp-conversation-audit.md). The spec is the contract for v0.1.x; changes require a successor spec (002, 003, …).

## How to contribute

1. **Open an issue first** describing the change you propose. This saves both you and the maintainers time before code is written.
2. **Wait for maintainer feedback.** Allow a few business days. For non-architectural changes, the response is typically fast.
3. **Fork the repository**, create a branch for your change, and open a PR. Reference the issue number in the PR description.
4. **PR title format:** `<area>: <short description>`. Examples:
   - `adapter: add Otter txt parser`
   - `docs: clarify profile override syntax`
   - `examples: add hiring-panel example with Coverage gap`

## Code & content style

- **Markdown-first.** v0.1.x is implemented entirely in markdown SKILL.md plus YAML profiles plus markdown documentation. No Python in v0.1.x.
- **Token-efficient.** SKILL.md body should not balloon past 800 lines without strong justification.
- **No emoji** in skill content or commit messages. Documentation may use sparingly (e.g., section markers).
- **Cite Paper 1** when adding claims that depend on framework grounding. The Paper 1 anchor is non-negotiable.
- **Plain prose** in documentation. No marketing-style hyperbole. The product positioning is sharp enough on substance.
- **Frontmatter** on every SKILL.md, spec, plan, and example transcript.

## Commit message format

Follow conventional commit-style prefixes when possible:

- `feat:` new feature or capability
- `fix:` bug fix
- `docs:` documentation only
- `refactor:` code change that doesn't change behavior
- `test:` calibration runs or test additions
- `chore:` build / tooling

Examples:
- `feat: add markdown_tagged heading variant fallback`
- `fix: turn_number validation rejects negative values`
- `docs: clarify CLARIFY vs REFUSE distinction in usage.md`

## License

By contributing, you agree your contribution is released under the same dual license terms (MIT for non-commercial use; commercial use requires separate license). See [`LICENSE`](LICENSE).

If you contribute substantial code or new architectural patterns, you may be acknowledged in subsequent paper revisions as appropriate.

## Questions

For general questions: open a Discussion on the GitHub repository or email `lxrsherry@gmail.com`.

For commercial licensing inquiries: email `lxrsherry@gmail.com` directly. See [`docs/services.md`](docs/services.md).
