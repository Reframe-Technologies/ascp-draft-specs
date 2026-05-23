# Repository Guidelines

## Project Structure & Module Organization
- `specs/` holds the draft protocol specifications (Markdown). Start with `specs/README.md` for the index and reading order.
- `specs/AGENTS.md` defines the authoritative drafting, editing, and review standards for the ASCP specification suite. For any file under `specs/`, `specs/AGENTS.md` takes precedence over this top-level guide on all spec-drafting, editing, structure, and review matters.
- Top-level docs (`README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) define project goals, contribution rules, and security reporting.
- There is no application source tree yet; this repository is spec-first.

## Build, Test, and Development Commands
- No build or test system is configured for this repo.
- Use standard Markdown preview tools as needed. Example: `rg "MUST|SHOULD|MAY" specs` to review normative language usage.

## Coding Style & Naming Conventions
- Write in clear, technical Markdown with RFC 2119/8174 keywords (MUST/SHOULD/MAY) for normative statements.
- Keep headings descriptive and sentence case; prefer short paragraphs and bullet lists for clarity.
- New spec files should live in `specs/` and follow existing naming patterns like `ascp-<topic>.md`.
- For ASCP protocol documents in `specs/`, treat `specs/AGENTS.md` as the authoritative source for source-of-truth boundaries, required section expectations, normative/informative separation, extensibility rules, naming expectations, and ASCP-specific security/privacy review. If this top-level file and `specs/AGENTS.md` overlap, follow `specs/AGENTS.md`.

## Testing Guidelines
- There are no automated tests or coverage requirements at this stage.
- When adding or updating a spec, also update `specs/README.md` and ensure the master spec references remain consistent.
- When adding or substantially revising a protocol-facing spec, review it against the acceptance criteria in `specs/AGENTS.md`, including any relevant `Security Considerations`, `IANA Considerations`, `Privacy Considerations`, and `Operational Considerations`.

## Commit & Pull Request Guidelines
- DCO sign-off is required for every commit: `git commit -s -m "<message>"`.
- Recent history uses short, sentence-case subjects without prefixes; follow that pattern.
- PRs should include a concise description, list affected spec files (for example, `specs/ascp-logsync-protocol-alsp.md`), and link relevant issues or discussions when applicable.

## Security & Reporting
- Follow `SECURITY.md` for reporting vulnerabilities or sensitive concerns; do not open public issues for security reports.
