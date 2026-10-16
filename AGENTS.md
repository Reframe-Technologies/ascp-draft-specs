# Repository Guidelines

## Project Structure & Module Organization

Top-level docs (`README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) define project goals, contribution rules, and security reporting.

### `specs/`

- Holds the formal ASCP protocol specification suite. Start with `specs/README.md` for the index and reading order. Place protocol requirements in the specification that owns the behavior and use precise technical prose.
- For any file under `specs/`, `specs/AGENTS.md` takes precedence over this top-level guide on drafting, editing, structure, and review matters, including source-of-truth boundaries, normative/informative separation, extensibility, naming, and security/privacy review.
- Place new specifications here and follow existing naming patterns like `ascp-<topic>.md`.
- When adding or updating a spec, also update `specs/README.md` and ensure the master spec references remain consistent. When adding or substantially revising a protocol-facing spec, review it against the acceptance criteria in `specs/AGENTS.md`, including any relevant `Security Considerations`, `IANA Considerations`, `Privacy Considerations`, and `Operational Considerations`.

### `issues/`

- Holds issue records documenting problems, proposals, rationale, decisions, and resolutions. Preserve decision context and track the incorporation of resolutions into affected documents; the owning specifications define current protocol requirements.
- Name issue records `issue-<number>-<topic>.md`.

### `posts/`

- Holds articles and publication drafts explaining ASCP's motivation, concepts, and use cases. Use accessible explanatory prose for a broader audience, distinguish specified capabilities from proposed capabilities, and keep technical explanations consistent with the owning specifications.
- Name post drafts `post-<topic>.md`.

### `notes/`

- Holds working design notes and exploratory proposals supporting protocol development. Notes may explore new behavior and develop toward formal specifications. Clearly distinguish provisional proposals from established protocol requirements, including when using RFC 2119/8174 keywords for proposed requirements.
- Use descriptive topic-based names, such as `<topic>-design-note.md`.

## Build, Test, and Development Commands
- No build or test system is configured for this repo.
- Use standard Markdown preview tools as needed. Example: `rg "MUST|SHOULD|MAY" specs` to review normative language usage.

## Writing style and naming conventions
- Write in clear Markdown suited to the document's purpose and audience.
- Use RFC 2119/8174 keywords (MUST/SHOULD/MAY) for normative statements.
- Keep headings descriptive and sentence case; prefer short paragraphs and bullet lists for clarity.

## Testing Guidelines
- There are no automated tests or coverage requirements at this stage.

## Commit & Pull Request Guidelines
- DCO sign-off is required for every commit: `git commit -s -m "<message>"`.
- Recent history uses short, sentence-case subjects without prefixes; follow that pattern.
- PRs should include a concise description, list affected documents, and link relevant issues or discussions when applicable.

## Security & Reporting
- Follow `SECURITY.md` for reporting vulnerabilities or sensitive concerns; do not open public issues for security reports.
