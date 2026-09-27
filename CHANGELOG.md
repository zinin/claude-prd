# Changelog

All notable changes to prd-flow will be documented here.

## [0.2.0] - 2026-09-27

### Changed
- **Renamed from claude-prd to prd-flow.** The skills keep their names: `/claude-prd:refine-prd`
  is `/prd-flow:refine-prd`, and so on.

## [0.1.0] - 2026-08-09

### Added
- Initial release: three skills moved out of the author's personal `~/.claude/skills/`
  into a plugin, so they install from the `zinin` marketplace instead of being copied by hand.
- `idea-to-prd` skill — collaborative discovery (context exploration, one question at a
  time, 2-3 approaches with trade-offs, section-by-section design validation) that writes
  `.taskmaster/docs/prd.md` and stops there.
- `refine-prd` skill — autonomous PRD validator: auto-fixes contradictions, vague language,
  missing acceptance criteria/priorities/dependencies; asks at most 3 questions per run and
  only at genuine decision forks; records every change in a PRD changelog section.
- `refine-tasks` skill — validates `tasks.json` as a complete, self-contained projection of
  the PRD: coverage matrix over requirements/user stories/roadmap, structural checks on the
  dependency graph, and AI-agent executability rules (no manual or visual verification steps).

### Changed
- Cross-references between the skills now use the plugin namespace (`claude-prd:*`).
