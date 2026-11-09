---
name: verification-before-completion
description: Use before claiming that an implementation, fix, build, test suite or artifact is complete or passing. Match each claim to inspected evidence for the current relevant state.
---

# Verification Before Completion

Report observable results rather than confidence or intention.

1. Identify what would substantiate the specific claim. Choose checks appropriate to the change: behavior tests for code, parsing for structured files, visual inspection for layout, and required repository checks.
2. Execute the relevant checks and inspect the complete output, exit status and failures. A successful command is evidence only for what it actually checked.
3. Compare the result with the requested acceptance criteria. Fix failures in scope, or describe the remaining limitation and its effect.
4. State what changed, what was verified and any material untested scope. Do not describe a mock, dry run, static inspection or local fixture as a successful live integration.

Verification remains current while the tested files, environment and relevant state are unchanged. A new chat message does not invalidate evidence. New edits, failures or unresolved concerns require the relevant checks again; do not repeatedly rerun unrelated successful checks.

Scale effort to risk. Do not create tests that merely mirror implementation, or add tests for low-impact reversible prose or styling edits without a meaningful invariant. User and repository requirements take precedence. Do not trigger paid services, deployment or unrelated external actions solely to obtain evidence beyond the authorized task.

If a check cannot run, report the reason and the narrower evidence available. A missing tool, timeout or collection error is not a passing test. Completion of the requested local work does not imply production readiness or remote publication.
