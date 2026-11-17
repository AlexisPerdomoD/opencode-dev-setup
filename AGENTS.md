# Global OpenCode Instructions

## GitHub Issues and Epics

- Before creating a GitHub issue, search the repository for duplicates or related issues.
- Before GitHub operations that depend on permissions or identity, retrieve the authenticated user context.
- Use the repository's available issue type and label conventions when creating or updating issues.
- Inspect the current repository implementation before changing an issue description. Do not infer status from an old issue body, roadmap, or report.
- Keep epic issues stable and architectural. An epic should describe purpose, scope, principles, decisions, contracts, responsibilities, and references.
- Do not put progress percentages, checklists, dates, pending-work tables, or mutable implementation status in an epic description.
- Track progress through child issues and their GitHub state. Link new work to the appropriate parent issue when the repository supports issue hierarchies.
- Keep implementation status, current gaps, acceptance criteria, dependencies, and out-of-scope items in the child issues where they can be updated independently.
- When an implementation changes semantics, align the whole issue tree as described in [Semantic Changes and the Issue Tree](#semantic-changes-and-the-issue-tree), not just the issue in hand.
- Preserve the distinction between an architectural contract and a concrete infrastructure implementation. Do not claim a provider, integration, endpoint, or capability is implemented unless it exists in the repository and is wired into the running application.
- Separate completed foundations from deferred integrations. For example, an abstraction or local stub does not mean that the production provider is implemented.
- Do not mix unrelated capabilities into an issue. Split a new concern into a separate issue and link dependencies explicitly.
- When closing an issue, verify its acceptance criteria against the current implementation and use the completed state reason. If the work is intentionally not planned, use the corresponding state reason instead.
- When an issue is blocked by an external team or decision, document the exact dependency and keep the implementation scope separate from that external task.
- Use concrete repository paths, symbols, endpoints, and contracts in issue descriptions. Remove stale names and historical routes when the implementation has changed.

## Semantic Changes and the Issue Tree

A **semantic change** is any change to the meaning of a field, column, state, transition, error code, contract, event, or flow. Code that enforces a new meaning while the issue tree still documents the old one leaves the project with two sources of truth.

When such a change appears while implementing a PR:

1. **Start at the root.** Identify the root issue of the tree being worked on (the epic), even when the task belongs to a child issue. The root owns the contract: purpose, glossary, invariants, decisions, and references.
2. **Review the whole tree.** Read the root, its sub-epics, and the task issues. List every place describing the old semantics: fields, columns, statuses, error codes, diagrams, acceptance criteria, examples, and dependencies.
3. **Keep one source of truth.** Put the contract in the root or in the sub-epic that owns it, and make the children reference it instead of restating it. Delete duplicated definitions rather than updating the same rule in several places.
4. **Align before or with the code.** Update the issues in the same PR or before it. Never merge an implementation whose documented semantics disagree with the code.
5. **Report what changed.** In the PR description, list the issues updated and the decision that changed.
6. **Respect history.** Closed issues are historical records: do not rewrite them. Fix the open tree and note any stale reference worth keeping.

When a decision replaces an earlier one, record it explicitly in the owning issue: what changed, why, and what it supersedes. Update the dependency map and the parent/child links if the change moves work between issues.

## Pull Requests

- Before creating a pull request, search for and use the repository pull request template if one exists.
- **Always link the issue with a closing keyword.** Include `Closes #<n>` in the PR description, one line per issue, so GitHub closes the linked issue automatically on merge. GitHub accepts `close`, `closes`, `closed`, `fix`, `fixes`, `fixed`, `resolve`, `resolves` and `resolved`; never rely on the title or an informal mention.
- **Closing keywords only fire against the default branch.** A task PR whose base is an integration branch (`feat/<ctx>/<se>-<slug>`) does not auto-close its issue: close it explicitly after the merge, or make sure the stable PR carries `Closes #<n>`.
- Align the pull request description with the actual diff and current runtime behavior.
- State clearly what is included, what is intentionally excluded, how it was verified, and which follow-up issues contain deferred work.
- Do not describe a stub, mock, interface, or test double as a production integration.
- When updating a merged or closed pull request for historical accuracy, preserve the fact that it is closed and avoid implying that follow-up work was included.

## Verification

- For issue or pull request alignment, inspect the working tree and recent commits before making claims about the current state.
- After GitHub updates, re-read the affected issues or pull requests and verify titles, state, links, and descriptions.
- Prefer concise, maintainable descriptions over duplicated implementation detail. Link to source files and technical documents instead of copying mutable code behavior into epics.

## Out-of-Scope Findings

When performing any task (tests, refactor, cleanup, verification) and encountering a bug, technical debt, or source-code problem **outside the requested scope**, report and stop:

1. Point at the finding with precise location (file:line)
2. Explain why it is a problem
3. Stop and wait for instructions before proceeding

This also applies to findings that surface _while verifying_ your own work, not only while writing tests.

Detecting an out-of-scope problem does **not** authorize fixing it, not "while we're at it", and not rewriting files or tests to accommodate it.

## Verifying a Baseline Without Mutating the Working Tree

When determining whether a failure is **yours** or **pre-existing**, do not alter the user's repository state:

| Forbidden                                          | Safe alternative                                                                           |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `git stash`                                        | `git worktree add /tmp/<dir> HEAD` — separate directory, leaves the current tree untouched |
| Hand-rewriting a file to simulate an earlier state | `git show HEAD:<path> > /tmp/<file>` and compare in that environment                       |
| `git checkout` over uncommitted work               | `git worktree add` + `git diff` against the worktree                                       |

General rule: **the user's working tree is read-only** unless the task explicitly requires writing to it. All diagnostic experiments happen in a separate temporary directory.

If a diagnosis genuinely requires mutating the user's tree, ask for explicit authorization first.

**When reporting a finding, perform no further working-tree verification maneuver**: report and stop.
