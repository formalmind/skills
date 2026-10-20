# formalmind/skills

The skills of the **formalmind swarm**: a small team of bots that keeps a Lean 4 + Mathlib
specification of a Solidity protocol proved, reconciles it with every protocol commit, and
rebuilds the protocol from the proofs. Each role is a Claude Code skill. The GitHub workflows
that schedule them live in the swarm's home repository
([asyncswap/v2-proofs](https://github.com/asyncswap/v2-proofs)).

## Installation

```
/plugin marketplace add formalmind/skills
/plugin install formalmind@formalmind
```

## Skills

| Skill | What it is |
| --- | --- |
| `formalmind:swarm` | The shared rules every role reads first: turns, `fm:` labels, notebooks, budget, `/ask` |
| `formalmind:cycle` | The rebuild cycle: rebuild from the proofs, compare with the draft, polish, re-prove |
| `formalmind:plan` | Planner: scopes the roadmap into small `fm:todo` issues |
| `formalmind:prove` | Prover: implements one issue in Lean and opens the PR |
| `formalmind:review` | Reviewer: first read of a PR |
| `formalmind:merge` | Merger: second read after the clean-build gate, and the merge |
| `formalmind:chief` | Chief: the daily audit |
| `formalmind:rebuild` | Rebuilder: the protocol's Solidity, from the proofs alone |
| `formalmind:editor` | Editor: compares draft and rebuild and moves the cycle on |
| `formalmind:consolidate` | Consolidator: carries the decisions into `official/` |
| `formalmind:assistant` | Assistant: the only role that talks to the maintainer |
| `formalmind:answer` | How any role answers an `/ask <role>` |
| `formalmind:reconcile` | The formal-mind bot: reconcile the spec with a new protocol commit |
| `formalmind:mention` | The `@formalmind` chat bot, and the voice every role writes in |
| `formalmind:lean4-cleanup` | Clear Lean linter warnings without changing what is proved |

The role skills expect what the swarm's workflow puts in their prompt: `ROLE`, `MODE`,
`REPO`, `SWARM_REPO`, `NUMBER`, the notebook issue numbers, `MAINTAINER` and so on.
`formalmind:lean4-cleanup` works anywhere.

## Releasing

The swarm's workflows check this repository out at a pinned commit: the repository variable
`FORMALMIND_SKILLS_REF` in the home, with a commit fallback in each workflow. To ship a change
to a role, merge it here, then set the variable to the new commit:
`gh variable set FORMALMIND_SKILLS_REF -R asyncswap/v2-proofs -b <sha>`.
