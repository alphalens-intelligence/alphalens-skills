# Changelog

All notable changes to this project are documented in this file. The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [2.0.0] — 2025-05-27

### Changed

- **Repository flattened**: `SKILL.md` moved from `alphalens-api/SKILL.md` to the repo root. This enables direct git clone installs without a subfolder copy step.
- **`skill.yaml`** now includes `repository:` field.

### Added

- **`LICENSE`** file (MIT-0) — previously declared in `skill.yaml` but not bundled as a file.
- **`CHANGELOG.md`** — this file.

### Fixed

- **Auth contract bug** (shipped in 1.1.0): SKILL.md now requires `KEY="$ALPHALENS_API_KEY"` as the first bash command before any API call. Without this, Claude Code sends empty `API-Key:` header and receives 401 on every request.

## [1.1.0] — 2025-05-27

### Fixed

- **Auth contract bug**: SKILL.md previously stated that `$KEY` was "injected by the agent runtime", but no runtime injects `$KEY` — OpenClaw injects `$ALPHALENS_API_KEY`, Claude Code requires manual export. This caused silent 401 errors. Now requires explicit `KEY="$ALPHALENS_API_KEY"` aliasing.

## [1.0.18] — Prior to 2025-05-27

Final release of the nested `alphalens-skills/alphalens-api/` layout. See git history for prior changes.
