# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`AGENTS.md` is the authoritative contributor manual (lesson contract, quiz schema, dependency allowlist, commit rules, CI/bot ownership). Read it before editing; this file only adds what it doesn't cover.

## What this repo is

A curriculum (542 lessons, 20 phases under `phases/`), not an app. Each lesson is a self-contained directory: `docs/en.md`, `code/main.<lang>` + `code/tests/`, `quiz.json`, `outputs/`. Languages are Python, TypeScript, Rust, Julia, stdlib-first. There is no repo-wide build or test command; everything runs per lesson.

Beyond `phases/`, several parallel content trees share the same contract style: `certifications/claude/` (tracks, lessons, assessments), `manuals/` (pinned-version manuals), `projects/`, `practice/`, `cheatsheets/`. The static site in `site/` is generated from these trees plus `README.md`, `ROADMAP.md` and `glossary/terms.md` by `site/build.js` and its `build-*.js` siblings. `site/data.js` and `catalog.json` are generated: never edit or commit them. `skills/` is the source of the learner-facing skills; `.claude/skills/` mirrors it.

## Commands

Run one lesson (from its `code/` directory):

```bash
python3 main.py && python3 -m unittest discover tests -v   # Python
npx tsx --test                                             # TypeScript
rustc --edition 2021 main.rs -o /tmp/x && /tmp/x           # Rust (single file, inline tests)
julia main.jl                                              # Julia
```

Run a single test: `python3 -m unittest tests.test_main.TestClass.test_name -v` from the lesson's `code/` dir.

Repo-wide checks (what CI's `audit` job runs):

```bash
python3 scripts/audit_lessons.py            # blocking in CI
python3 scripts/audit_certifications.py
python3 scripts/check_quiz_bias.py --check
python3 scripts/check_readme_counts.py      # advisory; CI auto-fixes on main
node --test site/test_manuals.js && node scripts/audit_manuals.js   # when manuals/ changes (must print TOTAL 0)
node --test site/test_*.js                  # site tests; CI lists them individually in curriculum.yml
```

Bulk lesson runners:

```bash
python3 scripts/lesson_run.py [--phase 14] [--strict]   # byte-compile only; --execute actually runs
python3 scripts/run_lesson_tests.py [--phase NN-slug]   # runs each lesson's tests; skips lessons whose optional deps (torch, h5py, ...) aren't installed
```

## Environment notes

- Python deps beyond the stdlib are limited to the AGENTS.md allowlist (`numpy`, `torch`, `h5py`, `zstandard`, `safetensors`). `requirements.txt` is a broader learner-facing list and is not what lessons are held to. A missing optional dep makes `run_lesson_tests.py` skip a lesson rather than fail it, so a silent skip can hide a broken environment.
- Rust uses plain `rustc`, never cargo (`Cargo.lock` and `target/` are gitignored).
- No `package.json` at the root: Node tooling is plain scripts plus `node --test`; `npx tsx` is for TypeScript lessons.
- Use `gh` for GitHub work; the GitHub MCP server is unreliable here.

## Workflow gotchas

- One commit per lesson directory, subject `feat(phase-NN/MM): <slug>`. When adding a lesson, also add the `[Title](phases/NN-phase/MM-lesson/)` link row to `README.md` (the site derives URLs from that link) and update `ROADMAP.md`.
- Diagrams are Mermaid or SVG only; the `blueprint-figure` and `motion-figure` skills produce the house-style SVGs.
- Certification lessons additionally need the exact sections `Interactive Lab`, `Practice Lab`, `Shipped Artifact`, `Verify It`, `Capstone Connection`, plus 5+ deterministic tests; certification and manual content is exempt from the book and translation pipelines.
