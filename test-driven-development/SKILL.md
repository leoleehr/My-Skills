---
name: test-driven-development
description: Use for software behavior changes, bug fixes, or refactors when a regression test can meaningfully establish correctness, or when the user explicitly requests TDD. Excludes prose-only edits, static artwork and low-impact reversible configuration changes without runtime behavior.
---

# Test-Driven Development

Use red–green–refactor for observable behavior that merits regression coverage.

1. Inspect the repository's existing test conventions and affected behavior.
2. Add a focused test for the desired result or reported bug. Run it and confirm it fails for the intended reason, rather than an import, environment or syntax error.
3. Make the smallest coherent implementation change. Run the focused test until it passes.
4. Refactor if useful, keeping the relevant tests green. Run the affected suite and required project checks before reporting completion.

Test public behavior and meaningful boundaries rather than private implementation details. Prefer real collaborators where practical; use controlled substitutes for external services or nondeterministic effects. For common testing mistakes, read [writing good tests](writing-good-tests.md).

Scale verification to the change. Prose, styling without behavioral risk, throwaway prototypes and low-impact configuration do not automatically require tests or a permission question. Follow explicit user and project requirements.

Preserve existing work. If implementation already exists, write a regression test, demonstrate the defect with a reversible isolated fixture or temporary patch when practical, then restore and fix. Never delete the user's code just to recreate a test-first history. If no meaningful red phase is feasible, state that limit and verify the actual behavior.

Report the checks actually run and any remaining failures. A passing test does not prove untested integrations or production readiness.
