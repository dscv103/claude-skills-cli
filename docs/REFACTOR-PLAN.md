# Refactor Plan: Claude Skills CLI → GitHub Copilot Instructions

## Goal
Retarget the CLI from creating and validating Claude Agent Skills to generating, validating, and packaging GitHub Copilot instructions/skills while keeping the existing progressive-disclosure safeguards.

## Current State Snapshot
- Commands: `init`, `validate`, `doctor`, `package`, `stats`, `install`, `add-hook`.
- Skill shape: `.claude/skills/<name>/SKILL.md` with YAML frontmatter + body, references/, scripts/, assets/.
- Validators enforce name format, description size, body line/token limits, reference consistency, and hook generation.

## Proposed Copilot Instruction Model
- Folder root: `.copilot/skills/<name>/`.
- Primary file: `INSTRUCTION.md` (or `SKILL.md` kept as alias) with frontmatter:
  - `name`, `summary`, `triggers` (keywords/intents), `entrypoint` (chat vs. workspace), `inputs` (optional schema), `capabilities` (what the skill can safely do), `constraints`, `examples`.
- Optional subfolders:
  - `references/` for deeper docs, `snippets/` (copilot-ready code blocks), `scripts/` for runnable helpers.
- Packaging target: zip suitable for Copilot Workspace/Chat import (confirm supported format).

## Refactor Plan (Phased)
1) **Alignment & Compatibility**
   - Introduce `.copilot/skills` layout alongside `.claude/skills`; add detection for both during transition.
   - Keep CLI entry point but add `copilot-skills-cli` alias/binary; deprecate `claude-skills-cli` in help text.

2) **Templates & Init**
   - Replace SKILL.md template with INSTRUCTION.md fields (summary, triggers, capabilities, constraints, examples).
   - Update `init` to scaffold `.copilot/skills/<name>` and optionally generate Claude-compatible files when `--legacy` is passed.

3) **Validation Updates**
   - Adapt validators to Copilot constraints: enforce concise summary (<200 chars), bounded examples (≤2), and safe capability/constraint sections.
   - Keep progressive-disclosure guards (line/word/token limits) but tune defaults for Copilot (e.g., 75–100 lines for body, stricter on instructions without refs).
   - Validate references/snippets exist and are linked; ensure no orphaned files.

4) **Doctor & Migration**
   - Extend `doctor` to normalize frontmatter keys to Copilot schema and fix multiline summaries.
   - Add `migrate` command: convert `.claude/skills/<name>` → `.copilot/skills/<name>` (rename SKILL.md → INSTRUCTION.md, map fields, update links).

5) **Packaging & Install**
   - Update `package` to emit Copilot-friendly zip naming (`<name>-copilot-skill.zip`) and include metadata manifest if required.
   - Revise `install`/`add-hook` to target Copilot config (chat/workspace instructions file or settings JSON); keep no-op stubs for Claude hooks during transition.

6) **Stats & Reporting**
   - Adjust `stats` to surface Copilot-specific quality signals (trigger coverage, capability completeness, missing examples) and flag legacy skills.

7) **Docs & Examples**
   - Replace README framing with Copilot-first language, add migration guide, and ship example Copilot instruction in `.copilot/skills/skill-creator`.
   - Document compatibility matrix (Copilot vs. Claude) and sunset timeline for legacy flags.

8) **Testing & Release**
   - Add unit coverage for new templates/validators/migration.
   - Provide compatibility fixtures for both `.claude` and `.copilot` layouts.
   - Release plan: pre-release with dual support, final release after deprecation window.

## Open Questions / Assumptions
- Confirm Copilot import format and whether chat vs. workspace require different manifests.
- Determine official hook/activation mechanism for Copilot (if any) to replace Claude hooks.
- Token/line limits for Copilot instructions are assumed similar to Claude; adjust once official guidance is verified.
