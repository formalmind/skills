---
name: lean4-style
description: Style guide for writing Lean 4 / Mathlib specs and proofs that build with no linter warnings — minimal simp sets, no dead tactics, `<;>` only where it fans out, current lemma names, no unused binders. Use whenever you write or edit Lean in a formalmind project (prover, reconcile, mention bot in prove mode).
---

# Lean 4 style for specs

Write proofs that `lake build` accepts without warnings the first time. The linter
flags each habit below, and the reviewer reads every warning a PR adds as an
unfinished proof.

## Simp

- Pass `simp` / `simp only` exactly the lemmas it uses. Once a proof closes, take out
  each argument you only added on a guess. The usual unused ones are `Ne.symm h`, local
  hypotheses, `eq_self_iff_true`, `true_and`, `false_and`, `not_false_eq_true`,
  `add_zero`, and frame lemmas that another simp set already covers.
- `simp_all` already uses every hypothesis in context, so don't list hypotheses for it.
- Prefer `simp only [...]` in a long proof. It stays stable when Mathlib's simp set
  changes.

## Tactic structure

- Use `tac1 <;> tac2` only when `tac1` leaves several goals and `tac2` is needed on each.
  When there is one goal, put `tac2` on the next line.
- When `simp` closes some branches and `omega` is needed for the rest, write
  `all_goals (…simp…)` and then `all_goals omega` on its own line, not a trailing
  `<;> omega`.
- No dead tactics: no `try simp` after a goal is closed, no `first | a | b` whose `b`
  never runs, no `split_ifs` with nothing to split. If you added one to unstick a proof,
  take it out once the proof works.
- Delete any `have` that nothing uses.

## Names and binders

- Use current names. For example, `ite_eq_left` / `ite_eq_right` replace `if_pos` /
  `if_neg`, and `ite_true` / `ite_false` replace `if_true` / `if_false`. When the build
  reports a deprecation, use the replacement it names.
- If a definition needs a hypothesis or argument only in its type, name it `_h` / `_cfg`.
  In a `fun` or `intro`, use `_` for anything you don't reference.

## Never

- `set_option linter.* false`, `sorry`, `admit`, or new axioms.
- Weakening or renaming a theorem so that it builds without warnings. If a warning can't
  be avoided without that, leave it and say why in the PR or changelog entry.

## Check before you push

`lake env lean <file>` on each `.lean` file you touched prints nothing except the
"consider adding `module`" import notices, which are a separate migration. Leave
warnings in files you didn't touch alone, and keep the diff inside the issue's scope.
