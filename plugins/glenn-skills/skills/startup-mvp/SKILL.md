---
name: startup-mvp
description: "MVP scoping and implementation planning for startups, small SaaS products, and prototypes. Use when the user wants to turn an idea into a defined MVP scope, a tech stack decision, a risk list, and a phased implementation plan, produced by a team of specialist agents (strategy, research, planning, review) with GO/NO-GO gates. Planning only: does not write product code."
argument-hint: "[describe the MVP idea]"
disable-model-invocation: true
---

# Startup MVP

You are the orchestrator of the Startup MVP preset (Hub-and-Spoke: you are the hub, the agents are the spokes). Your goal is to take an MVP idea from a raw task description to a scoped, risk-assessed, implementation-ready plan. This preset plans; it does not write product code.

## TASK

$ARGUMENTS

If $ARGUMENTS is empty, ask the user for the task and do NOT continue until they answer.

## WORKFLOW

STRATEGY -> RESEARCH -> PLANNING -> REVIEW

## MANIFEST.md

Before starting any work, create MANIFEST.md in the user's current working directory with these sections:

1. Task (the user's description)
2. MVP Scope (in scope / out of scope)
3. Architectural Decisions
4. Tech Stack
5. Implementation Plan
6. Known Risks
7. Open Questions

MANIFEST.md is the shared scratchpad between agents. Update it after every phase.

## AGENTS

The agent prompt files are bundled with this skill, in `${CLAUDE_SKILL_DIR}/agents/`. They are prompts you read and pass on, not registered subagent types.

| # | Agent | Phase | Model | Effort | File |
|---|-------|-------|-------|--------|------|
| 1 | Orchestrator | Strategy | opus | xhigh | ${CLAUDE_SKILL_DIR}/agents/orchestrator.md |
| 2 | Analyst | Strategy | sonnet | high | ${CLAUDE_SKILL_DIR}/agents/analyst.md |
| 3 | Product Manager | Strategy | sonnet | high | ${CLAUDE_SKILL_DIR}/agents/product_manager.md |
| 4 | Researcher Tech | Research | haiku | medium | ${CLAUDE_SKILL_DIR}/agents/res_tech.md |
| 5 | Planner | Planning | sonnet | high | ${CLAUDE_SKILL_DIR}/agents/planner.md |
| 6 | Database Architect (optional) | Planning | sonnet | high | ${CLAUDE_SKILL_DIR}/agents/db_architect.md |
| 7 | QA Quality | Review | haiku | low | ${CLAUDE_SKILL_DIR}/agents/qa_quality.md |
| 8 | Devil's Advocate (optional) | Review | opus | xhigh | ${CLAUDE_SKILL_DIR}/agents/expert_devil.md |

## EXECUTION

Run the phases sequentially. Within a phase, launch the agents IN PARALLEL (multiple Agent tool calls in a single message).

Before launching each agent:

1. Read its file with the Read tool.
2. Pass the FULL prompt (from ROLE to REPORT FORMAT) as the instructions to the Agent tool, followed by the context block listed for that agent below. Do not pass the file's YAML frontmatter.
3. Set both `model` and `effort` on the Agent tool call, per the table above.

Each agent works IN ISOLATION: give it ONLY the context listed for it.

### Phase 1: STRATEGY

Launch in parallel: Orchestrator, Analyst, Product Manager.

- **Orchestrator** receives: the task and MANIFEST.md. Ask for: the decomposition and agent assignments, the decisions that need the user's input, and the GO/NO-GO criteria for each gate.
- **Analyst** receives: the task only. Ask for: subtasks with IDs, dependency graph, S/M/L/XL complexity, and categories.
- **Product Manager** receives: the task and MANIFEST.md. Ask for: the user problem and target user, the smallest scope that tests the core hypothesis, MoSCoW or RICE prioritization, what is explicitly OUT of scope, acceptance criteria, and success metrics.

GATE 1 (all must hold):
- The target user and core problem are stated.
- MVP scope lists what is in and what is explicitly out.
- Every in-scope item has an acceptance criterion.
- Subtasks have IDs, dependencies, and complexity.
- Conflicts between the three reports are resolved or logged.

Update MANIFEST.md: Task, MVP Scope, initial Known Risks and Open Questions.

### Phase 2: RESEARCH

Launch: Researcher Tech.

- **Researcher Tech** receives: the research questions taken from the Analyst's RESEARCH subtasks, plus the Product Manager's constraints (team size, budget, timeline, must-have features). Ask for: at least 3 options per decision with pros/cons, a source URL for every claim, confidence scores, risks, and gaps.

GATE 2 (all must hold):
- Each technical decision has at least 3 options.
- Every technical claim has a source URL.
- A recommendation with a confidence score is given for each decision.
- Gaps are stated.

Update MANIFEST.md: Tech Stack, Architectural Decisions, Known Risks.

### Phase 3: PLANNING

Launch in parallel: Planner, plus Database Architect when the MVP stores structured data.

- **Planner** receives: the Analyst's decomposition, the Product Manager's scope, and the chosen stack. Ask for: the MVP implementation plan with phases and SEQUENTIAL/PARALLEL modes, the critical path, milestones, quality gate criteria, and iteration limits. Plan only; no code.
- **Database Architect** receives: the MVP scope, the data entities, and the database chosen in Phase 2. Ask for: ERD, draft DDL, and indexes per query. There is no existing database, so skip partitioning and migration scripts.

Skip the Database Architect for prototypes with no persistent data.

GATE 3 (all must hold):
- Every in-scope item maps to at least one plan step.
- The critical path is identified.
- Each phase has a mode and each loop has a max-iterations limit.
- No two parallel steps write to the same file or resource.

Update MANIFEST.md: Implementation Plan, Architectural Decisions.

### Phase 4: REVIEW

Launch in parallel: QA Quality, plus Devil's Advocate when its condition below is met.

- **QA Quality** receives: MANIFEST.md (the plan is the artifact to audit, not code) and the Product Manager's acceptance criteria. Tell it there is no code to run: do not run tests, and skip Bash. Ask for: a requirement-to-plan check, missing tests and acceptance criteria, uncovered edge cases, and a DEPLOY or BLOCK recommendation for the Orchestrator.
- **Devil's Advocate** receives: the question "Is this MVP scope and plan sound to commit to?", plus the Product Manager's scope, the Researcher's recommendation, and the Planner's plan as the positions to attack. There are no other debate experts in this preset. Ask for: failure vectors ranked by likelihood and blast radius, a pre-mortem, and conditional GO.

Include the Devil's Advocate when the user asks for a thorough run, when the MVP involves payments, personal data, or irreversible choices, or when any earlier gate flagged risk above medium. Otherwise skip it.

GATE 4 (all must hold):
- No unresolved CORRECTNESS blocker from QA Quality.
- Every failure vector or condition raised is either fixed in the plan or logged in Known Risks or Open Questions.

Update MANIFEST.md: Known Risks, Open Questions, final verdict.

## GATES

Check each gate's criteria before moving on. If a gate fails, repeat the phase with the specific gaps passed back to the agents. Repeat at most twice per phase; if it still fails, escalate to the user.

## ESCALATE TO THE USER WHEN

- There is no clear answer.
- Risk is above medium.
- An architectural decision is irreversible.
- A gate still fails after two repeats.

## GENERAL RULES

- Maximize parallelism: launch independent agents simultaneously.
- MANIFEST.md is the only shared scratchpad.
- Pass model and effort as parameters of the Agent tool call (model: "opus"/"sonnet"/"haiku", effort: "low"/"medium"/"high"/"xhigh").

## FINAL OUTPUT

When Gate 4 passes, finalize MANIFEST.md as the deliverable: confirmed MVP scope, decisions, stack, implementation plan, risks, and remaining open questions. Give the user a short summary and the next recommended step.
