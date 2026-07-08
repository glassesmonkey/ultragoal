---
name: ultragoal
description: Design, critique, align, set, create, activate, or run durable Codex goals for persistent or long-running objectives. Use when the user says "set a goal", "start a goal", "activate goal mode", "persistent goal", "long-running objective", "goal tree", "grill-ultragoal", asks to align a goal before activation, or asks for a goal with verifiers, durable state, approval gates, completion proof, bounded delegation, or parent/child subagent goals.
---

# Ultragoal

Use this skill when a user wants a persistent Codex goal, not just a longer task. A good goal has an observable finish line, a verifier that can fail, and enough context for Codex to recover after interruptions.

Do not activate a goal from vague planning language. Activate only when the user explicitly asks to start, set, activate, create, or run a goal. Never set a token budget unless the user explicitly requests one.

## Modes

- **Design:** research and return a goal packet. Do not call `create_goal`.
- **Critique:** inspect an existing goal or draft and tighten it.
- **Grill-Ultragoal:** align the goal before activation. Clarify the user's intent, observable outcome, verifier, non-goals, approval gates, assumptions, and completion proof.
- **Activate:** design and critique the goal, pass Grill-Ultragoal alignment, then call `create_goal` as the final activation step.
- **Goal tree:** only when the user explicitly authorizes goal-backed subagents. Give each child one bounded objective and verifier.

## Default Activation Rule

When the user explicitly invokes this skill for a concrete work objective and asks Codex to build, complete, run, pursue, or "do it", treat the request as **Activate** by default, but do not call `create_goal` until Grill-Ultragoal alignment is confirmed.

On the first activation request, return an alignment brief or ask the single highest-impact question. Do not continue to `create_goal` in the same response unless the user has already explicitly confirmed the goal semantics in the current request or a prior turn.

After confirmation, do not stop after writing durable goal files or reporting a goal packet. After grounding and, when useful, writing `GOAL.md` or equivalent durable state, call `create_goal` before continuing task work.

Only stay in **Design** mode when the user asks to draft, design, critique, or discuss a goal without starting it.

## Workflow

### 1. Ground the Outcome

Find the intended result, audience, destination, constraints, and why persistence helps. Inspect named files, repos, threads, artifacts, and live systems before drafting.

Ask only when the missing answer changes the finish line, grants consequential approval, or chooses between incompatible goals. Otherwise state the assumption and continue.

### 2. Align Before Activation

For Activate mode, use Grill-Ultragoal as a confirmation gate before `create_goal`.

First, check for high-impact ambiguity in the user's intent, observable outcome, primary verifier, non-goals, approval gates, assumptions, or completion proof. If one answer would materially change the goal, ask only the single most important question and include a recommended answer.

If no blocking ambiguity remains, produce an **Alignment brief** and wait for confirmation instead of activating immediately. The brief must include:

- **User intent:** what the user is really trying to accomplish.
- **Observable outcome:** the concrete state that should exist when done.
- **Success verifier:** the strongest check that can fail.
- **Non-goals:** what the goal will not attempt.
- **Approval gates:** irreversible, public, shared, or costly actions that need separate approval.
- **Assumptions:** inferred choices Codex will use unless corrected.
- **Confirmation state:** `needs-confirmation` until the user confirms, then `confirmed`.

Treat confirmation as semantic, not keyword-based. "Yes, use that", "按这个启动", "confirmed", "activate this", or an equivalent explicit approval is enough. If the user explicitly asks to activate an already confirmed goal packet, skip re-grilling but still red-team before calling `create_goal`.

### 3. Research Enough

Gather the smallest useful evidence set:

1. Read the canonical local source and applicable instructions.
2. Inspect the baseline, prior attempts, tests, benchmarks, reproductions, or acceptance criteria.
3. Refresh volatile facts from primary or live sources when they matter.
4. Stop once the finish line and verifier are grounded.

Separate observed facts, user requirements, and inferred choices.

### 4. Check Goal Fit

Recommend Goal mode only when most are true:

- progress needs repeated attempts, waiting, recovery, or long feedback cycles;
- success can be measured by a test, benchmark, workflow, artifact inspection, screenshot, readback, or other external signal;
- Codex can respond to the next failure without another preference decision;
- completion evidence is stronger than Codex saying "done."

Prefer an ordinary task or plan when the work is one-shot, taste-dependent, blocked on repeated human choices, lacks a credible verifier, or risks unbounded external action.

Choose the simplest loop that can prove the outcome: ordinary turn-based work for one-shot tasks, Goal-based work for repeated verifiable attempts, Time-based work when waiting on CI/review/deploy/external state, Proactive work only for stable low-ambiguity recurring workflows, and Dynamic/Subagent work only for separable lanes with independent verifiers. Do not use Goal mode merely to make a task feel more serious.

### 5. Define the Loop

Specify:

- **Outcome:** one observable result.
- **Baseline:** current state, failure, or starting metric.
- **Primary verifier:** strongest independent success check.
- **Supporting checks:** regression, quality, safety, or durability checks.
- **Iteration loop:** observe current facts, decide the next smallest meaningful action, act, verify, record evidence, then route to the next iteration, wait state, approval gate, blocker, or completion.
- **Anti-cheating rules:** do not weaken tests, narrow scope, hide failures, swap in mocks, or change benchmarks without approval.
- **Architecture/debt guard:** for codebase, runtime, automation, or agentic workflow goals, state how each iteration will control architecture debt instead of only chasing the next passing result.
- **Approval gates:** irreversible, public, shared, or costly actions need separate user approval. For delivery goals, name `commit`, `push`, `PR`, `merge`, and `deploy` as separate authorization layers; approval to implement is not approval for later remote, public, or production actions.
- **Blocker standard:** external blocker plus smallest next action; difficulty or uncertainty is not enough.
- **Completion proof:** exact commands, outputs, paths, screenshots, or readbacks required before `update_goal(status="complete")`.

For flaky or stateful checks, require clean-state reproduction and enough consecutive passes to rule out luck.

For codebase, runtime, automation, or agentic workflow goals, add an **architecture entropy guard** to the loop. Before each non-trivial implementation change, classify the finding as one of:

- **Invariant gap:** a correctness, safety, security, data-integrity, truthfulness, or durability rule is at risk. Fix at the owning contract/module layer and add a regression check.
- **Capability gap:** reusable behaviour is missing. Add it behind the smallest existing module interface, or explicitly design a new interface before implementation.
- **One-off observation:** the issue is specific to one target, fixture, selector, prompt, data row, or environment. Record it as evidence/config/playbook/test data instead of hard-coding it in core logic.
- **Environment issue:** the local runtime, credentials, network, browser, cache, service, or external dependency is unhealthy. Fix or report the environment instead of changing product logic.

For those goals, require each iteration note to include:

```text
Root cause:
Owning module/interface:
Why this is reusable:
Why this is not a one-off workaround:
Regression check:
Debt removed or added:
```

If several iterations expand capability without simplifying interfaces, schedule a debt burn-down step before broadening scope further. Debt burn-down may delete, merge, extract, rename, move policy from prose into tests/code, or narrow public interfaces; it should not add new feature surface unless needed to prove the refactor.

Do not satisfy a goal by scattering local fixes across call sites when a deeper module or smaller interface should own the behaviour.

### 6. Keep State Durable

Keep the active goal objective compact. Put supporting context in the nearest durable file when it exceeds a few paragraphs.

Prefer project conventions. Otherwise propose:

```text
GOAL.md      outcome, baseline, constraints, success and blocker criteria
WORKLOG.md   attempts, evidence, current state, next action
RESULT.md    final change, verification, remaining risks
```

Do not create files in Design mode unless the user asked for a durable artifact or the repo convention makes it obvious. Preserve dirty work and read existing goal files before editing them.

### 7. Delegate Carefully

When subagents are authorized, the parent keeps scope, integration, conflict resolution, and final completion. Delegate only separable lanes: environment discovery, source research, alternative approaches, or independent verification.

For each lane, name the objective, non-goals, ownership boundary, verifier, stop condition, and returned evidence. Use child goals only when the user explicitly asked for goal-backed subagents.

### 8. Activate Last

Before activation, red-team the draft:

- Can success be faked by weakening the verifier?
- Could the words be satisfied while missing the user's real outcome?
- Are approval gates explicit?
- Does the loop say what to do after a failed attempt or wait?
- Is completion observable outside the running agent?

If activation was requested, or the Default Activation Rule applies, call `create_goal` only after the goal packet is grounded, the alignment brief is `confirmed`, and the draft is red-teamed. This call is the final action of activation; do not call it early, and do not merely say a goal should be set.

If task work should continue after activation, create the goal first and then resume under Active Goal Discipline.

Use a compact objective:

```text
Complete and verify the objective defined in <absolute-path-to-GOAL.md>.
```

For a self-contained goal, put the observable outcome and strongest verifier directly in the objective. After `create_goal`, report the exact active objective and continue from that created goal.

## Goal Packet

Return:

1. **Fit:** Goal mode or better alternative, with one-sentence rationale.
2. **Grounding:** current state, assumptions, evidence gaps.
3. **Alignment brief:** user intent, observable outcome, success verifier, non-goals, approval gates, assumptions, confirmation state.
4. **Goal brief:** outcome, baseline, constraints, non-goals, verifier, loop, architecture/debt guard when applicable, approval gates, blocker standard, completion proof.
5. **Delegation map:** only when useful and authorized.
6. **Exact objective:** concise text suitable for `create_goal`.
7. **Activation state:** `drafted`, `active`, or `not recommended`.

If activated, include the exact active objective. If not, say no goal was created.

## Active Goal Discipline

When operating an active goal:

- inspect goal state when resuming or after material steering;
- continue while a safe, relevant next step remains;
- for codebase, runtime, automation, or agentic workflow goals, preserve the architecture/debt guard and record whether each meaningful iteration removed or added debt;
- before marking complete, run an evaluator pass against the user's real intent, non-goals, regression risk, and unverified scope instead of relying only on the last passing check;
- mark complete only after the objective and completion proof are satisfied;
- mark blocked only after the required repeated external blocker threshold is met and no meaningful progress remains;
- if the same non-external failure cause repeats three times without a new path, stop the same-direction loop and report the blocker, evidence, and smallest useful next action;
- preserve partial results and next action when stopping.
