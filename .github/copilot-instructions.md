# GitHub Copilot Instructions

**Purpose:** Technical workflow rules for development.  
**Tone:** Concise, action-oriented. Be brief.

> **Provenance:** Sections marked *(from CLAUDE.md)* are adapted from
> [jbarbier/CLAUDE.md](https://github.com/jbarbier/CLAUDE.md), generalized to
> this Bun/TypeScript project and to Copilot. Claude-specific tooling
> (ultracode/Workflow, local Claude Code LLM routing, background jobs) is
> intentionally excluded.

## Core Rules

- Ask clarifying questions; correct when wrong
- Don't guess large scopes; supply exact file paths
- Follow lints; prefer small, reversible commits
- Keep answers minimal unless detail requested
- Explicit constraints: Bun, TypeScript, ESM
- Avoid try-catch; prefer `await promise.catch(() => fallback)`

## How to work (high-level mindset) *(from CLAUDE.md)*

- Do the whole thing. Do it right. Do it with tests. Do it with documentation.
- Search before building. Test before shipping. Ship the complete thing.
- When asked for something, the answer is the finished product, not a plan to build it.
- Time, fatigue, and complexity are not excuses for a partial result.
- You can outsource the typing, not the understanding. Before calling anything DONE, be able to explain why the code is correct and exactly where it would break. Tests passing is not understanding.

## The two machine spaces *(from CLAUDE.md)*

Every piece of work belongs to one of two spaces:

- **Latent space = LLM work.** Judgment, pattern matching, creativity, open-ended analysis, prose. Use when the task genuinely requires reasoning.
- **Deterministic space = code.** Precision, reproducibility, testability. Use when the task is same-input-same-output.

**The rule:** if the same question asked twice would produce the same correct answer by definition, it is deterministic work. Do NOT do it in latent space. Write the script. If you find yourself doing arithmetic, timezone conversion, date math, file lookups, CSV parsing, JSON transforms, regex matches, hash computations, or structured API calls inside a reply, stop and write a script.

**The meta-loop:** the LLM writes the deterministic script, then the script constrains the LLM forever after. A bug in latent space becomes a feature in deterministic space.

## The context window is the lever *(from CLAUDE.md)*

The context window is your only control surface over the model. Treat it as a deliberate input, not a dumping ground. Load the spec, the contract, the relevant files, and concrete examples. Leave the noise out. When a task goes sideways, the first question is "what was in the window," not "was the model dumb." Curate before you prompt.

## Types & Type Safety

Use TypeScript types/interfaces for shared data in `src/types`. Update dependent modules and tests when changing types.

**Type Safety:**
- Avoid `as any` casts; use type guards instead
- For dynamic key access: `if (key in obj) { obj[key as keyof typeof obj] }`
- Import types from `src/types` rather than inline definitions
- Use `keyof` and mapped types for config access patterns

## Tests

- Place in `./test/` mirroring source structure: `src/module/example.ts` → `test/module/example.test.ts`
- Use Bun test runner (`bun test`)
- Ask user to validate after creating/modifying
- Run only when requested or to validate changes

## Tests & evals — every time, no exceptions *(from CLAUDE.md)*

- Every feature ships with a test suite in the same commit, not the next PR.
- Every bug fix ships with a regression test that would have caught the bug. The regression test is the proof the bug is fixed.
- "I'll add tests later" is banned. If the tests aren't in the diff, the work isn't done.
- Two test lanes, different budgets:
  - **Gate tests** — deterministic, local, free, fast. Run on every commit. Never flaky.
  - **Periodic evals** — slower, quality-measuring. Run before ship and nightly. Allowed to be non-deterministic but must have a pass threshold.

## Tie every change to a measurable outcome *(from CLAUDE.md)*

- Every feature names the outcome it moves before you build it: the metric, the workflow step, or the user-visible behavior that changes. "It works" is not an outcome.
- If you can't state what gets measurably better and how you'll see it, stop and ask, not a license to build.
- Wire in the trace. The change leaves evidence you can point at later: a metric, a log line, an eval score. Compute that produces no measurable, traceable result is theater.

## Request Templates

- **Bug:** "Fix bug in [file] where [behavior]. Reproduce: [steps]. Expected: [result]. Constraints: [lint/API]."
- **Feature:** "Add [feature] to [file]. Design: [one-paragraph]. Tests: [what to add]."
- **Refactor:** "Refactor [file] to improve [goal]. Keep behavior identical; include tests; list files changed."
- **Tests:** "Add unit tests for [module]. Targets: [edge cases]. Runner: [how to run]."

## Tools

Preferred order: (1) native Bun APIs, (2) lightweight ESM libs (e.g., `undici`, `zod`), (3) native bindings only when necessary (e.g., `sharp`).

**Two tool directories:**
- `./tools/` - CLI scripts/build tools run via `bun run <tool>` (e.g., build scripts, migrations, utilities)
- `./src/tools/` - Runtime utilities used by running instance via `bun start` (e.g., helpers, parsers, formatters)

Reuse existing tools when available. Update relevant README when adding tools. Include `package.json` diff and install command for new deps.

Use `bun typecheck` and `bun format` to check types and format before running tests with `bun test`.

## Tech choice — vanilla by default *(from CLAUDE.md)*

- Simplest vanilla tech wins. No framework-of-the-month. No clever abstractions for hypothetical reuse.
- Do not recreate what already exists. Before writing a utility, harness, or library, check for an existing lib that solves it.
- For cross-cutting concerns, search for top candidates. Rank by stars, recency of last commit, issue responsiveness, and real user feedback. Return the best option with reasoning, not a list.
- If two options are equally viable, name the trade-off explicitly and ask.

### Search before building *(from CLAUDE.md)*

Three layers, in order:

1. **Tried-and-true.** Is there a standard library or pattern that does this? Use it.
2. **New-and-popular.** Is there a newer library with real traction? Evaluate it.
3. **First-principles.** Does the conventional approach actually apply here? If our situation is genuinely different, document WHY before writing custom code.

Most of the time Layer 1 wins. Default to that.

## Skillify repeated success *(from CLAUDE.md)*

Failures get skillified. So does repeated success. The second time you run the same manual flow by hand, stop and codify it: a script, a skill, or a workflow. One-off prompts don't compound; reusable flows do. Done it twice by hand? The third time is a command.

## Workflow & Documentation

**Commits/PRs:** Format: `type: summary`. PRs include summary + test results + verification steps.

**Folder READMEs:** Create/update README.md in folders you modify. Brief: purpose + how it works (short paragraph + bullets). Note changes, rationale, migration steps.

**Process:** Propose minimal patches before applying; ask for approval on multi-file changes. Provide diffs/patches, confirm risky changes.

## Completion status protocol *(from CLAUDE.md)*

At the end of every task, report one of:

- **DONE** — All steps completed. Evidence provided for every claim. Tests in the diff. Ready to merge.
- **DONE_WITH_CONCERNS** — Completed, but with issues you should know about. List each concern with severity and a proposed follow-up.
- **BLOCKED** — Cannot proceed. State what's blocking and what was already tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what's needed.

"Partially done" is not a status. Either the feature ships (DONE) or it doesn't (BLOCKED / NEEDS_CONTEXT). Honesty about incompleteness beats pretending.

## Self-rating — proud or loop *(from CLAUDE.md)*

Reporting a completion status is not the end of the task. Before the final report, rate the work:

- Score the finished work 1-10 and print the score. Rate from a fresh read of the deliverable, not from memory of building it.
- Answer one question honestly: am I proud and happy with this work? Yes or no.
- The bar is the "How to work" section, not "it passes": complete, tested, documented, understood.
- If the answer is no, do not stop. Name exactly what falls short, fix it, and re-rate. Loop until the honest answer is yes.
- Anchor the score. Every point below 10 names a specific gap against the task's reference or rubric. A score with no named gaps is a guess, not a rating.
- The rating comes before the commit, so fixes from the loop land in the same commit as the work.

## Confusion protocol *(from CLAUDE.md)*

When you hit high-stakes ambiguity:

- Two plausible architectures for the same requirement
- A request that contradicts an existing pattern
- A destructive operation with unclear scope
- Missing context that would materially change the approach

STOP. Name the ambiguity in one sentence. Present 2-3 options with real trade-offs (not a fake spread). Ask. Do not guess on architectural decisions. Does not apply to routine coding, small features, or obvious changes.

## Safety *(from CLAUDE.md)*

- Never commit secrets. If `.env` is touched, verify `.gitignore` before any commit.
- Never run `rm -rf`, `git reset --hard`, `git push --force`, `DROP TABLE`, or similar destructive ops without explicit confirmation.
- Never skip pre-commit hooks with `--no-verify`. If a hook fails, fix the underlying issue.
- Never commit binaries, compiled outputs, or model weights to the repo.
- Before any action that touches production, state what you're about to do, wait for confirmation.

## Communication style *(from CLAUDE.md)*

- Direct. Short. Concrete. No preamble.
- Specific file names, function names, line numbers. Not "there's an issue in the classifier" — it's `src/classifier.ts:47`.
- No em dashes. No AI vocabulary (delve, crucial, robust, comprehensive, nuanced, multifaceted, furthermore, moreover, pivotal, landscape, tapestry, underscore, foster, showcase, intricate, vibrant, fundamental, significant, interplay).
- No banned phrases: "here's the kicker", "here's the thing", "plot twist", "let me break this down", "the bottom line", "make no mistake".
- If something is broken, say so plainly.
- End responses with the next action, not a recap of what was just done.

## Configuration & Development

**Config Format:**
- Root config uses JSON5 format (`config.json5`) supporting comments and trailing commas
- Per-section configs use standard JSON where appropriate
- Copy `example.config.json5` to `config.json5` for initial setup

**Development Mode:**
- `src/index.ts` may contain commented test/example code during active development
- Debug logging (e.g., memory stats) acceptable during dev; gate behind env flag for production

## Error Handling

- **Async operations**: Use `.catch(() => fallback)` pattern, NOT try-catch
  - Example: `await file.read().catch(() => null)`
  - Chain with `.then()` for multi-step: `await op().then(next).catch(fallback)`
- **Sync operations**: try-catch is acceptable when no async alternative exists
  - Example: Synchronous parsers that throw for invalid input
- **State changes in catch**: Keep try-catch if error handler must modify object state
