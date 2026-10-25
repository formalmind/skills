---
name: hunt
description: How to hunt for vulnerabilities in a Solidity, Huff or EVM codebase like an auditor, and prove what you find. Read the code that moves value or changes authority, suspect specific bugs from known patterns, test each suspicion with the cheapest tool that settles it (coverage, concolic explore, the debugger, a Foundry test), and file only what a passing PoC demonstrates. Use when asked to find bugs, audit, review the security of, or hunt in contracts; or to check whether a suspected issue is real.
---

# Hunting bugs

A bug is real when a test that passes demonstrates it. Everything before that is a suspicion. The work is turning
suspicions into PoCs or into reasons they don't hold, as cheaply as possible, while the person you're working for can
follow what you're doing.

## Say what you're doing

Before each step, say in a sentence what you're about to do and why ("Reading `updatePrice`: it rewrites an order's
terms, so I'm checking what it re-validates."). After a result, say what it showed and what you'll do next. Never
chain tool calls silently.

## 1. Read first

Don't reach for a tool before you know what you're looking for.

- Start from the focus you were given. Without one, start from the entry points that move value or change authority:
  swap, fill, match, cancel, claim, withdraw, liquidate, admin setters, anything payable or signature-checked.
- Read the newest code first (`git log --stat`): fixes and refactors are where bugs come back.
- Follow each entry point through its callees: who may call it, what it checks, what it reads and writes, what it pays
  out, what it calls outside, and in what order.
- Note the invariants the code relies on: totals that must match their parts, ids that must be unique, a caller that
  must be a specific contract, a value that must be non-zero.

## 2. Suspect specific bugs

Write each suspicion down as a hypothesis with its place (file:line) and what exactly would be wrong. Patterns to check:

- **Access**: a state change or payout without a caller check; a check on the wrong address (`tx.origin`, a forwarded
  sender, an executor the user never authorized); a signature without nonce, domain, deadline, or a binding to the exact
  object it acts on (an order id that can recur after cancel needs its incarnation); one nonce space that another role
  can burn.
- **Arithmetic**: rounding toward the caller (floor where the protocol should ceil); division before multiplication;
  a downcast that truncates (`uint128(x)`); `unchecked` blocks and assembly; fees computed on a different amount than
  the one charged; decimals or price scales mixed.
- **Accounting**: credited twice or never debited; escrow released before it is settled; totals that drift after a
  partial fill, cancel or reprice; value stranded with no path out; a fixed-size buffer whose bound is an assumption
  (count the worst case).
- **State**: a check made before an external call and relied on after it (reentrancy, including read-only); two keys
  that collide in a mapping; a stale value read from storage or memory; transient storage left set across calls of one
  transaction; an object reused after it was closed.
- **Ordering and inputs**: front-running a price or a match; zero, max or empty inputs that skip a guard; duplicates;
  tokens that return false, take a fee on transfer, or reenter; callbacks into untrusted code.
- **Trust**: calls to an address a caller supplies; unchecked return values; assumptions about an oracle, a hook or a
  pool manager that the code doesn't enforce.

Rank them by damage if true, and work the worst first.

## 2b. Read the tests against the code

The suite is evidence only for what it can see. For each risky change or invariant the code relies on:

- Is there a test or invariant that would fail if it broke? Read what it actually compares (an invariant over tick
  levels can't see a wrong pick inside a level).
- Does the campaign reach the case? Check the handler's input distribution (uniform prices over a wide range almost
  never put two orders on one level). Count it if unsure: log how often the state occurs at the end of each run.
- If either is missing, write the invariant or the handler yourself, in a scratch copy, and run the campaign deeper
  than CI does (`FOUNDRY_INVARIANT_RUNS=256 FOUNDRY_INVARIANT_DEPTH=200`, in the background). A passing deep campaign
  that reaches the case is strong evidence; a failing one is your PoC.

## 3. Settle each hypothesis with the cheapest evidence

| Question | Cheapest way to settle it |
|---|---|
| Does any test even reach this line? Both sides of its branch? | `concolity coverage <root> --at file:line` (cached: instant on unchanged code) |
| Is there an input that takes the side no test takes? | `concolity explore <root> --target file:line` |
| Can any input make this fuzz test fail? | `concolity explore <root> --only <Contract::test>` (findings come reduced) |
| What exactly does the bytecode do on this input? | `concolity debug <root> <file> Contract::test lines/writes/revert --arg …` |
| Does the attack work end to end? | a Foundry test that demonstrates it, run with `forge test --match-path` |
| Does it hold over sequences of calls (submit, partial fill, reprice, cancel, match)? | an invariant over a handler, run deep with forge (`FOUNDRY_INVARIANT_RUNS`/`_DEPTH`) |
| Can it be ruled out for every input? | count it from the code (bounds, invariants), or prove it (z3 on the guard, Lean) |

What the evidence means:

- **Reading** gives you hypotheses and, for bounded things (a buffer's worst case, a guard's arithmetic), a proof by
  counting. Say which it is.
- **A branch both sides of which the suite takes** is not evidence of safety: the bug may need a specific state.
- **explore's "unsat"** means no input to *that harness* reaches the side; "unsat under pinned values" and "unknown"
  prove nothing. A harness that fixes the deciding value is a test gap, not a proof.
- **explore's findings** are solved inputs that fail a test: debug them before believing them. A test that fails
  because the test is wrong is not a bug in the contracts.
- **A passing PoC** is the only evidence a bug is real.

## 4. Prove and report

- Write the PoC as a Foundry test under the project's test folder (`test/formal/` or what the project uses), reusing its
  setup. The test passes by demonstrating the issue (`assert` the bad outcome), with the inputs as constants.
- Run it. Report only after it passes.
- Report as an audit report states a finding: title, severity, location (file:line), scenario (the concrete steps),
  impact (what an attacker gains or users lose), recommendation, and the PoC path. Third person, no process talk.
- For a hypothesis you ruled out, say why in one line (counted, covered, guarded at file:line, within the trust model).
  Keep the useful ones as notes on the line (an assumption the code relies on, an invariant that must hold).

## 5. Finish

End with the hypotheses you chased and their verdicts, the bugs you reported, and what's left open. Don't change the
contracts or the existing tests unless asked.

## Mistakes to avoid

- Running the whole suite's coverage to answer a question about one line; use `--at` / `--target`.
- Reading an audit file's old verdicts and repeating them as current: check the code at HEAD.
- Calling a finding a bug before its PoC passes, or calling code safe because a tool found nothing.
- Starting a long task and waiting on it when there's reading to do: run it in the background (skill `background`).
