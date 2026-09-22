# pr-playbook

How to open pull requests that get merged. Everything here was bought with real PRs: one merged in 37 minutes, four that went nowhere. The difference was not code quality.

One rule below is absolute. It comes first.

## Rule zero: no tooling provenance, ever

Never mention, hint at, or allude to AI or LLM involvement anywhere in a pull request. Not in the title, not in the body, not in commit messages, comments, or review replies.

- No "AI-assisted". No "reviewed by N reviewer contexts". No "no human review yet". No tooling provenance of any kind.
- No co-author trailers for tools, no generator attribution, nothing in the commit footer that is not a person who did the work. The author field is you, alone.

This rule has no exceptions and no judgment calls. The cost is asymmetric: silence costs nothing, while disclosure either reads as noise or as a reason to distrust every other claim in the PR. A real pull request was closed unread partly on this.

## Length is a liability

The limit on a pull request body is the maintainer's attention. Not what the API accepts. Length is never a quality signal: a stranger who demands twenty minutes of reading is a red flag, not a contributor.

- Every sentence earns its place or gets deleted.
- Evidence compresses to commands and numbers, not prose.
- What cannot be compressed does not belong in the body. It belongs in a linked issue.

A real PR carried a 12.9 KB body and a 1348-line test file, and was closed unread. The maintainer's entire review was "That is a lot of text." Write for the maintainer who almost closed that tab.

## The premise gate

Before writing any code, all three must hold. Fixing a bug that does not exist is the most expensive failure mode: the maintainer closes the issue as user error, and your PR becomes proof you did not read the thread.

1. **Maintainer acknowledgment.** The issue is labeled, triaged, or reproduced by a maintainer, or a project member has engaged in the thread. An issue asking "would a PR be welcome?" with no answer is not an answer. No engagement: do not code. Ask on the issue and wait.
2. **Behavior contract.** The project's own docs, spec, or tests say the reported behavior is wrong. A reporter's expectation is not a contract.
3. **Default configuration.** The fix must fix the issue under the project's default configuration. If it does not, the fix is not done. "Doesn't work by default" is never a follow-up item or a scope note. It means do not submit.

Found the bug yourself? The issue comes first, and the PR waits for its triage. A pull request against a one-day-old issue nobody has engaged with is a bet that a stranger's expectation — or yours — is the project's contract.

The one fast merge behind this playbook started exactly there: an issue labeled `replicated` by the maintainer the day before the PR existed.

## Repository preflight

Read CONTRIBUTING, the pull request template, and the CI workflow before writing code, not before opening the PR.

- **CLA/DCO: sign first.** An unsigned CLA makes the PR unreviewable from minute one, and every hour spent after that is wasted.
- **The repository's template wins.** If the project has a PR template, fill it in. The body shape below is for repositories without one.
- **Build and test reality.** If you cannot build the project and run its own test path locally, in the real configuration — patched toolchains, proprietary CI and all — do not submit. There is no "Unverified" section to park that in. An unverifiable core claim means the PR does not exist yet.

## Body format

Four blocks, in this order, each only as long as its claim needs.

1. **What was broken.** The mechanism, with `file:line`, in one or two sentences.
2. **What the fix does, and why that layer.**
3. **How to verify.** The one command a maintainer can run, and what fails without the fix.
4. **Numbers, only if they exist.** Method in one line.

Rules across the whole body:

- `Fixes #N` only when merging truly closes the issue. `Refs #N` otherwise.
- One logical change per pull request. Three fixes, three pull requests.
- No unrelated refactors, renames, or formatting churn.
- Never claim something a tool did not produce. "Should be faster" is not a number.

Banned sections:

- **Follow-up.** Verified sibling defects go in a separate issue, or at most one line. Never a section.
- **Unverified.** See preflight: if you could not verify it, do not submit it.
- **Review provenance.** See rule zero.
- **Docs boilerplate.** If there is nothing user-facing to say, say nothing.

## How maintainers actually read

Skim order: title, diff stat, first paragraph. Judgment happens in seconds, and the rest of the PR confirms a judgment already made.

- The burden of proof scales with diff size, and with how little the maintainer believes the bug exists.
- A large test file from a first-time contributor is a smell, not a virtue. Match the repository's own test conventions.
- Review is volunteer time, and your PR is an interruption. A maintainer who can verify your claim with one command merges fast. One who must take your word closes the tab.

## Worked example

[odin-lang/Odin#7607](https://github.com/odin-lang/Odin/pull/7607) merged in 37 minutes with zero review comments. Its body was long. That is not why it merged. The maintainer had labeled the underlying issue `replicated` a day before the PR existed — the bug was acknowledged before any code was written — and the fix could be checked with one command. The premise gate and one-command verification did the work; the body was along for the ride.

Do not copy that body, or any body, as a template. For a small fix, the whole body looks like this:

```markdown
`os.rename` failed when the destination existed on Windows: `MoveFileEx`
was called without `MOVEFILE_REPLACE_EXISTING` (`src/os/windows.cpp:212`).

The flag is now passed at that call site, the single place the rename
path enters Win32, so every caller is covered.

Verify with `./run_tests os/rename`: the `replaces_existing` case fails
without the fix.

Refs #7602.
```

Six lines. The first sentence alone gives a maintainer the mechanism, the location, and the scope.

## Titles, commits, branches

| Field | Convention |
| --- | --- |
| Branch | `fix/` or `feat/` plus a short kebab-case scope: `fix/rename-replace-existing` |
| Commit subject | Conventional Commits, imperative, one scope, no trailing period: `fix(os): pass MOVEFILE_REPLACE_EXISTING in rename` |
| Commit body | Why, not what: the violated invariant and the blast radius. Wrap at 72 columns. |
| PR title | The commit subject without the issue number; GitHub appends it on squash. |

## Review etiquette

- Answer every review comment in its thread, with the revision that answers it. "Done" is not an answer.
- Push follow-up commits during review. Never force-push: comments lose their anchor and the diff moves under them.
- If a maintainer overrules your approach, implement their call, or state the tradeoff once and stop.

## Pre-submit checklist

- Premise gate passed: a named maintainer signal — a label, a replication, or thread engagement.
- Preflight done: CLA signed, the repository's PR template filled, the project built and its own test path run locally in the real configuration.
- `git diff --stat` shows only the stated change; `git status --porcelain` shows no stray files.
- The commit subject follows the convention, and the author is you alone.
- The body passes the skim test: title, diff stat, and first paragraph alone tell a maintainer what changed and why.

## License

[MIT](LICENSE)
