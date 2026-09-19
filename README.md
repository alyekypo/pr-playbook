# pr-playbook

---

[![license](https://img.shields.io/github/license/alyekypo/pr-playbook?style=flat-square&color=blue)](LICENSE)
[![last commit](https://img.shields.io/github/last-commit/alyekypo/pr-playbook?style=flat-square)](https://github.com/alyekypo/pr-playbook/commits/main)
[![repo size](https://img.shields.io/github/repo-size/alyekypo/pr-playbook?style=flat-square)](https://github.com/alyekypo/pr-playbook)
[![stars](https://img.shields.io/github/stars/alyekypo/pr-playbook?style=flat-square)](https://github.com/alyekypo/pr-playbook/stargazers)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/alyekypo/pr-playbook/pulls)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow?style=flat-square)](https://www.conventionalcommits.org/en/v1.0.0/)
[![markup](https://img.shields.io/badge/markup-GFM-000000?style=flat-square&logo=github&logoColor=white)](https://github.github.com/gfm/)

A reference for opening pull requests that a maintainer can merge without a round trip: the exact section order, the evidence each section has to carry, and what to leave out. Everything here was applied to a real upstream PR, which is reproduced verbatim in [Worked example](#worked-example).

---

## Contents

- [The one rule](#the-one-rule)
- [Title and commit](#title-and-commit)
- [Body format](#body-format)
- [Section by section](#section-by-section)
- [What to send, what not to send](#what-to-send-what-not-to-send)
- [Anti-patterns](#anti-patterns)
- [Worked example](#worked-example)
- [Pre-submit checklist](#pre-submit-checklist)
- [GitHub field limits](#github-field-limits)
- [Review etiquette](#review-etiquette)
- [License](#license)

## The one rule

A pull request is a claim that a specific behavior changed for a specific reason, and the reviewer's job is to falsify that claim. Every sentence you write either helps them do that or gets in the way. If a claim has no observation behind it, label it as unverified instead of stating it.

## Title and commit

| | |
| --- | --- |
| Branch | `fix/` `feat/` `perf/` `docs/` `chore/` + a short kebab-case scope, e.g. `fix/dev-head-metadata-watcher-invalidation` |
| Commit subject | Conventional Commits, imperative, one scope, no trailing period: `fix(head-metadata): invalidate component metadata only for module-graph files` |
| Commit body | Why, not what: the violated invariant, the mechanism, the blast radius. Wrap at 72 columns. |
| PR title | The commit subject, minus the issue or PR number. GitHub appends the number on squash. |
| Attribution | The author only. No co-author trailers, no generator trailers, no agent attribution. |

Keep one logical change per pull request. If you fixed three things, send three pull requests; a bundle hides review surface and every review comment then applies to an unclear scope.

## Body format

Five sections, in this order. Sections that have nothing to say are not dropped silently: either fill them or state why they are empty.

| Section | Answers | Carries |
| --- | --- | --- |
| Changes | What changed and why | Root cause, exact paths with line numbers, the mechanism, scope boundaries |
| Measurements | How much it moved | Before and after numbers, the method, the environment, the fixture |
| Testing | What proves it | Test files and cases, what fails without the fix, adjacent suites and static checks |
| Follow-up | What you found but did not touch | Verified sibling defects, with one line of evidence each |
| Docs | User-facing impact | Docs or changeset changes, or an explicit "none, because ..." |

Rules that apply to the whole body:

- Reference the issue with `Fixes #N` only when merging the pull request really closes it. Use `Refs #N` otherwise. A wrong `Fixes` closes an issue that is not resolved.
- Never claim something a tool did not produce. "Should be faster" is not a measurement; "median 144-172 ms before, 56-72 ms after, 20 sequential requests, same fixture" is.
- No narrative of how you got there, no thinking out loud, no apology, no "small change, should be fine".
- Straight quotes, plain punctuation, no emoji decoration. Code, paths, commands and numbers in inline code.
- Screenshots are for visual output only. Timing, logs and test results belong in text so they can be searched and quoted in review.

## Section by section

### Changes

State the invariant that was violated, then the mechanism, then the fix, with paths and line numbers that a reviewer can click through to. Show the causal chain end to end: trigger -> violated invariant -> propagation -> symptom.

```markdown
## Changes

- `path/to/file.ts:124-143` - what the code did before, in one sentence, with the consequence.
- What it does now, and why that set is exactly the set that can affect the reported behavior.
- Part of the mechanism you captured directly, quoted from the tool output.
- Why an older release did not show the same symptom.
```

### Measurements

Numbers, not adjectives. Each table or bullet names the environment, the workload and how many samples. If you cannot measure, say so and say why - an honest "unverified" beats a confident guess that a reviewer later disproves.

```markdown
| revision | median |
| --- | --- |
| previous release | 48-60 ms |
| main before this change | 144-172 ms |
| this branch | 56-72 ms |
```

### Testing

Name the test file, the number of cases, and the exact observation that fails without the fix. A reviewer must be able to check your claim by reverting the source and re-running one command; include that command. Then list the adjacent suites and static checks you ran, and the ones you could not run.

```markdown
- `path/to/test.test.ts` (4 tests). The new case fails without the fix: <observation>.
- Also green: <file>, <file>, <19 tests across suites>, plus <typechecker>, <linter>, <formatter>.
- Unverified: <what you could not run, and why>.
```

### Follow-up

Defects of the same class that you verified but did not fix, one line each, with the reason they are out of scope. This is where a reviewer learns the boundaries of your change. It is also where you say "this predates my change".

### Docs

Either the documentation or changeset you added, or an explicit statement that user-facing behavior is unchanged and why that is true.

## What to send, what not to send

Send:

- The violated invariant and the mechanism, traced to `file:line`.
- A reproduction: a fixture, a test, or exact commands. If the issue lacks one, yours can be the reproduction.
- Before and after numbers with the measurement method.
- The tests you added and the ones you updated, with the observation that fails without the fix.
- The scope boundary: what you deliberately did not touch.
- The residue: what you could not verify, and the residual risk that leaves.
- A changeset or release note when the repository expects one.

Do not send:

- Attribution for tools, agents or models. Do not add co-author trailers for anything that is not a person responsible for the work.
- Claims without an observation: unmeasured performance, "obviously faster", "should be safe", "works for me".
- Screenshots of a terminal where selectable text or a number would do.
- Unrelated refactors, renames, dependency bumps or formatting churn. Each one hides review surface and can be sent separately.
- A fix for a symptom when the mechanism is still there, without saying that is what it is.
- Secrets, tokens, private hostnames, customer data, or internal URLs. Rotate anything you exposed before you open the pull request.
- Emoji, banners, marketing, "revolutionary", walls of bullet points without content.
- Self-review presented as independent review. If the change was reviewed, say by whom and on what revision.

## Anti-patterns

| Anti-pattern | Why it fails | Do instead |
| --- | --- | --- |
| "Fixes the issue" with no mechanism | The reviewer cannot tell whether the cause or a symptom was touched | Name the invariant, the propagation, the layer that now enforces it |
| Numbers without a method | Unfalsifiable, and CI cannot reproduce them | Environment, workload, sample count, exact revision |
| "Tests pass" | Passing tests can be the absence of a test | Name the case that fails without the fix |
| Fixing a neighbor while you are in the file | Review surface grows past the stated scope | Report it in Follow-up |
| A green suite as the only evidence | It proves the cases you wrote ran, not that the behavior is right | Add the boundary case that would break under the old mechanism |
| Giant single commit | Reviewers cannot follow the argument | One logical change, one commit, message carries the why |
| Force-pushing during review | Review comments lose their anchor and the diff moves | Push new commits until the review settles, then squash if the repository prefers it |

## Worked example

Real pull request: [withastro/astro#18072](https://github.com/withastro/astro/pull/18072). Branch `fix/dev-head-metadata-watcher-invalidation`, commit subject `fix(head-metadata): invalidate component metadata only for module-graph files`, 5 files, +201/-30.

<details>
<summary>The full body, as sent</summary>

```markdown
## Changes

Fixes a dev-server regression where the server re-evaluated its whole server module graph on every request while unrelated files under the project root were written during request handling. With `@astrojs/cloudflare`, wrangler/miniflare rewrites `.wrangler/state/v3/observability/miniflare-wobs-trace-store/*.sqlite-wal` during every request and the Vite watcher covers the project root, so every request paid the cost. Fixes #18065.

- `packages/astro/src/vite-plugin-head/index.ts:124-143` - `astro:head-metadata` invalidated `virtual:astro:component-metadata` for every watched file. That module is imported by the dev app entrypoint (`packages/astro/src/core/environment/dev-nonrunnable.ts:101` -> `astro/app/entrypoint/dev` -> `virtual:astro:app` -> `@astrojs/cloudflare/entrypoints/server`), so each invalidation hard-invalidated that entire importer chain and the workerd module runner re-evaluated the server graph on the next request.
- Watcher-driven invalidation is now limited to files that belong to the tracked `ssr`/`prerender` module graphs, which are exactly the modules the plugin's own `load()` hook reports, normalised with `vite.normalizePath` - the same function Vite applies before its own `getModulesByFile` lookup. Transform-hook (`index.ts:225`) and `resolveId` (`index.ts:176-196`) propagation invalidation is unchanged, so any module that can change component metadata still refreshes it.
- Defect captured in a stack trace: `FSWatcher.emit -> invalidateComponentMetadataModule` (`dist/vite-plugin-head/index.js:24`) for the `.sqlite-wal` paths above.
- `astro@7.0.3` shipped the same blanket listener, but its pinned wrangler did not write those files while requests ran (0 watcher events measured), which is why the listener stayed latent there.

## Measurements

Cloudflare dev server (workerd), identical fixture, 20 sequential requests, median per request:

| revision | median |
| --- | --- |
| `astro@7.0.3` | 48-60 ms |
| `main` before this change | 144-172 ms |
| this branch | 56-72 ms |

Per-request module-graph counters, same fixture: before, 1 re-transform of `virtual:astro:component-metadata` plus re-invalidation of `worker-entry`, `@astrojs/cloudflare/entrypoints/server`, `virtual:astro:app` and `astro/app/entrypoint/dev` on every request; after, 0 invalidations and 0 transforms in steady state.

10 parallel requests to the same page: per-request p50 518-646 ms -> 317-403 ms, metadata-module invalidations 64-72 -> 0 per 60 requests.

## Testing

- `packages/astro/test/units/vite-plugin-head/head-metadata.test.ts` (4 tests). The new case fails without the fix: a watcher event for a file outside the module graph invalidated `virtual:astro:component-metadata` once per `add`/`change`/`unlink`. A second case pins the positive path, so an over-narrow filter fails as well.
- `packages/integrations/cloudflare/test/head-metadata-invalidation.test.ts` (3 tests) against a real workerd dev server. The new case writes a file under `.wrangler/state` and asserts that the watcher dispatch for it issues no metadata invalidation; without the fix it fails by exactly +1 per event (3/3 runs). The counter is attributed to the watcher dispatch by the fixture plugin (`test/fixtures/head-metadata-invalidation/astro.config.ts`), so background module re-transforms cannot produce a false failure, and the test awaits the watcher event instead of sleeping.
- Also green: `astro-head.test.ts`, `head-propagation-prerender-env.test.ts`, and the Cloudflare dev tests `ssr-deps`, `astro-dev-platform`, `content-collections-chunked`, `dev-image-endpoint` (19 tests), plus `tsc -b` on both test projects, biome and prettier.
- Unverified: per-request numbers on real Windows hardware (path normalisation is applied exactly where Vite applies it, but measurements were taken on Linux/WSL2 only).

## Follow-up (not fixed here)

`packages/astro/src/content/vite-plugin-content-imports.ts:171` reacts to every watcher event and classifies paths by extension without confining them to the content directory (`packages/astro/src/content/utils.ts:398-422`), so a `.json`/`.yaml`/`.md` write anywhere under the project root triggers a content config reload plus invalidation of all content import modules. It is latent in this scenario - the files Cloudflare's dev runtime rewrites per request are `.sqlite*`, which classify as ignored - and it predates this change.

## Docs

No docs change: dev-only internal invalidation, no user-facing API or config. Changeset: `.changeset/olive-pugs-repeat.md` (patch).
```

</details>

Why this body works: the first bullet is the violated invariant with a clickable location; the third is a tool observation, not an opinion; the measurements name the revision and the workload; the testing section names the observation that fails without the fix; the follow-up section draws the scope boundary; the docs section states the user-facing impact is nil and why. Nothing in it asks the reviewer to take a claim on faith.

## Pre-submit checklist

Run these before you open the pull request, from the repository root:

```bash
git diff --stat                      # scope is exactly the stated change
git status --porcelain               # no stray files, no scratch fixtures
git log -1 --format='%an <%ae>%n%s'  # author is you, subject follows the convention
pnpm changeset status                # a changeset exists if the repository requires one
pnpm lint                            # or the repository's own lint command
pnpm test                            # or the narrowest covering suite
```

Then read your own diff as if it were someone else's: every changed line has to trace to the stated change, and every claim in the body has to trace to a command you ran.

## GitHub field limits

The numbers marked as probed were measured by sending an oversized value to the API and reading the rejection; the quoted message is GitHub's own. Anything not probed is a convention, not a limit - treat a stated cap as a fact only if you have seen it reject you.

| Field | Limit | How it is known |
| --- | --- | --- |
| Repository name | 100 characters | Probed: 101 characters is rejected with `name cannot be more than 100 characters` |
| Repository description | 350 characters | Probed: 400 characters is rejected with `description cannot be more than 350 characters` |
| Repository topics | 20 topics, 50 characters each, lowercase, starting with a letter or number, hyphens allowed | Probed: 21 topics is rejected with `A repository cannot have more than 20 topics.`; 51 characters is rejected with `must start with a lowercase letter or number, consist of 50 characters or less, and can include hyphens.` |
| Pull request title | No cap hit when probed: 400 characters was accepted | Keep it under 72 anyway so it survives lists, browser tabs and email subjects |
| Pull request body | No cap hit when probed: 70000 characters was accepted | Length is never the reason to cut evidence |
| Issue and pull request comment | No cap hit when probed: 70000 characters was accepted | Same |

Two consequences worth remembering: the API accepts far more than the web UI makes comfortable, and a value that is too large is rejected before it is stored, so a failed request leaves nothing behind.

## Review etiquette

- Answer every review comment in the thread it belongs to, with the change or the reason there is none. "Done" without a revision reference is not an answer.
- Push follow-up commits during review; do not rewrite history under a reviewer's comments.
- If a maintainer overrules your approach, implement their call or state the tradeoff once and stop arguing.
- After a review, the reviewed revision is frozen: any edit reopens it, so tell the reviewer what changed and ask for a re-check instead of assuming the approval still holds.
- When the pull request is approved and merged, do not force-push the branch.

## License

[MIT](LICENSE)