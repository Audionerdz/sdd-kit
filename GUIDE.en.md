# Spec-Driven Development — Work Guide

## What is SDD?

Spec-Driven Development is a workflow where **you write the specification first, then the code**, using an AI coding agent (opencode, Claude Code, etc.) that guides you with questions at each step.

## The Complete Pipeline

```
Constitution (foundation)
    ↓
Feature Specs (what to build)
    ↓
Implementation (code)
    ↓
Validation (tests + review)
    ↓
Merge to main
    ↓
Next feature → back to step 2
```

---

## 1. Constitution — The Foundational Documents

These are 3 files in `specs/` that define the project before writing any code:

| File | Answers |
|------|---------|
| `specs/mission.md` | Why does it exist? Purpose, stakeholders, target audience |
| `specs/tech-stack.md` | What do we build it with? Language, framework, database, architecture |
| `specs/roadmap.md` | In what order? Small, implementable phases |

### How is it created?

You give the agent **Prompt 1**: project context and stakeholder input. Then **Prompt 2**: "create the constitution in `specs/`".

The agent uses its question tool (equivalent to `AskUserQuestion`) to ask you about:

1. **Mission** — purpose, problem it solves, users
2. **Tech Stack** — technologies, frameworks, database
3. **Roadmap** — implementation phases in order

You answer and the agent writes the 3 files.

> **Important rule:** the agent must ask BEFORE writing. If it writes directly, it is hallucinating decisions that you should be making.

---

## 2. Feature Spec — From the Constitution to the Specification

Once the constitution is ready, each new feature follows the **feature-spec skill** (`skills/feature-spec/`):

1. The agent reads `specs/roadmap.md` and finds the first incomplete phase (`[ ]`)
2. Creates a branch: `git checkout -b phase-N-name-of-the-phase`
3. Asks you 3 grouped questions:
   - **Scope** — what the feature includes/excludes, data, behavior
   - **Decisions** — implementation options, storage, UX
   - **Context** — tone, constraints, open questions
4. Reads `specs/mission.md` and `specs/tech-stack.md` to align the spec
5. Creates `specs/YYYY-MM-DD-name/` with 3 files:
   - `requirements.md` — scope, decisions, context
   - `plan.md` — grouped and implementable tasks
   - `validation.md` — how to validate (tests, walkthrough, definition of done)

> The branch is created **before** writing any spec, so all work on the feature stays isolated.

---

## 3. Implementation — From the Spec to the Code

With the spec ready, you implement following `plan.md`. The agent:

1. Reads `requirements.md` and `plan.md`
2. Implements each group of tasks
3. Commits frequently on the same branch
4. When done, runs validation

---

## 4. Validation — Does it Meet the Spec?

`validation.md` is executed:

- **Automatic** — tests, typecheck, lint
- **Manual** — feature walkthrough, edge cases
- If something fails → back to implementation
- If everything passes → merge to main

---

## 5. Merge and Next Feature

```bash
git checkout main          # move to the target branch (the one receiving the changes)
git merge phase-N-name     # merge the feature into main
git branch -d phase-N-name
```

> **Why `git checkout main` first?** `git merge` merges **into the branch you are currently on**. If you are on `phase-N-name` and run `git merge phase-N-name`, you would be merging the branch into itself. Always stand on the branch that will **receive** the changes, not the one that has them.

Update `roadmap.md` marking the phase as `[x]`. Repeat from step 2.

---

## Projects with Existing Code (Retroactive)

If the project already exists and you never used SDD, you cannot run feature-spec because there is no `mission.md` or `tech-stack.md`. The solution:

### Creating the Retroactive Constitution

1. The agent analyzes the existing code:
   - Reads `package.json`, `tsconfig.json`, `Dockerfile`, etc.
   - Examines the folder structure, dependencies, scripts
   - Reviews the README and existing documentation
   - Looks at recent commits to understand the current state

2. The agent asks you to refine:
   - **Mission** — what does the project really exist for? (may differ from what the README says)
   - **Tech Stack** — is this what you use or what you inherited?
   - **Roadmap** — what features are missing? what technical debt is there?

3. With the constitution ready, feature-spec works normally.

### Tips for Retroactive

- **Don't rewrite history.** The constitution describes the present, it doesn't idealize the past.
- **Be honest about tech-stack.** If the project uses jQuery and PHP, that's what goes in the skill. You change it when it's time to migrate.
- **Roadmap starts from today.** Phases look forward, they don't document already-done features.

### Step-by-Step Flow for Existing Code

Here is the concrete workflow for applying SDD to a project that already has working code:

```
1. Analyze current code → 2. Create retroactive constitution → 3. Feature spec for what's next → 4. Implement
```

#### Step 1: Current State Analysis (without agent)

Before involving the agent, do a quick inventory yourself:

- **Real stack:** what languages, frameworks, DB, external APIs does the project use?
- **Existing features:** make a list of what already works
- **Technical debt:** what you know is wrong and needs fixing
- **README or docs:** if there is existing documentation, use it as input

#### Step 2: Create the Constitution (with the agent)

You give the agent a prompt like:

> "Analyze this existing project. Read the code, package.json, folder structure. Then create a constitution in `specs/` with `mission.md`, `tech-stack.md`, and `roadmap.md` that describes the project as it exists today. Before writing, ask me whatever you need."

The agent should:
1. Explore `package.json`, `tsconfig.json`, `Dockerfile`, `composer.json`, `Cargo.toml`, etc.
2. Read source files to understand the architecture
3. Review recent commits to understand the work pace
4. Ask you what it cannot deduce from the code
5. Write the constitution

> **Important:** the retroactive constitution describes the project **as it actually is**, not as you wish it were. `roadmap.md` does look forward — pending features, not already-completed ones.

> **Important for the prompt:** explicitly tell the agent to **use its question tool** (`AskUserQuestion` in Claude Code, `question` in opencode, or whatever your coding agent has) before writing the files. If you don't ask, some agents write directly and hallucinate decisions. Example: *"Create the constitution in `specs/`. **You must** use your question tool before writing."*

#### Step 3: Feature Spec for What's Next

With the constitution ready, you use the feature-spec skill normally. The difference is that the feature-spec now **coexists with legacy code** — and that's fine.

Some common situations:

| Situation | How to handle it |
|-----------|-----------------|
| The new feature touches legacy code | `plan.md` must include refactoring or adaptation tasks |
| The real stack is not ideal for the new feature | Ask the agent: "should we migrate this module or add alongside?" |
| You don't know where to start | The feature-spec interview guides you: scope → decisions → context |
| The project has no tests | The first roadmap phase can be "Add tests to module X" |

#### Step 4: Implementation in a Mixed Project

Three rules to survive:

1. **Don't mix refactor with new features.** If you need to refactor, make it a separate phase in the roadmap or an explicit task in plan.md.
2. **Don't touch code that isn't in the spec.** If the feature-spec says "add GET /api/users endpoint", you only touch that. Improving the legacy code on the side is tempting but takes you off focus.
3. **Validate against what exists.** If the legacy code has no tests, manual validation (testing the existing feature + the new one) is enough.

### Common Mistakes When Starting SDD in Legacy Projects

| Mistake | Consequence | Solution |
|---------|-------------|----------|
| Wanting to refactor everything before starting | You never start | Make a "refactor" phase in the roadmap if urgent, but don't make it a requirement |
| Putting all legacy code in the roadmap | Eternal roadmap | The roadmap is for what's **missing**, not for what already exists |
| Ignoring the real stack in the constitution | Specs describe impossible features | Be honest. If the stack sucks, the roadmap will show the necessary migrations |
| Not asking the agent questions | The agent hallucinates decisions you didn't give | Always answer the 3 feature-spec questions before it writes |

---

## When to Create Branches

| Moment | Action |
|--------|--------|
| New feature | `git checkout -b phase-N-name` (done by feature-spec) |
| Production hotfix | Branch from main, not from your feature branch |
| Experimenting | Separate branch, no spec. If it works, convert it to a feature spec |
| Changing the constitution | `git checkout -b update-constitution`. Merged like any change |

**Rule:** each feature = one branch. Don't work on multiple features in the same branch.

---

## Frequently Asked Questions

### What is the Constitution?
It's 3 documents that define why the project exists, what technologies it's built with, and in what order it is implemented. It's the foundation of the entire SDD workflow.

### What does the AskUserQuestion tool contain?
It's the agent's tool for asking you questions. In the constitution it asks about mission, tech stack, and roadmap. In feature-spec it asks about scope, decisions, and context. The agent **must never write until you answer**.

### What if I want to start a new project, say in cybersecurity?
You use the same workflow: minimal scaffold (package.json, tsconfig.json, src/index.ts, README.md), create the constitution with the agent, and follow the SDD pipeline. The project topic (cybersecurity, web, CLI, whatever) doesn't change the workflow — only the content of the specs changes.

### What if the project didn't start with SDD?
Refer back to the "Projects with Existing Code (Retroactive)" section — the agent analyzes your existing code and deduces the constitution.

### Does feature-spec work for projects without a constitution?
No. Feature-spec needs `mission.md` and `tech-stack.md` to align the spec. Without a constitution, it has no context. First create the retroactive constitution.

---

## Final Project Structure

```
my-project/
├── specs/
│   ├── mission.md              ← constitution
│   ├── tech-stack.md           ← constitution
│   ├── roadmap.md              ← constitution
│   ├── YYYY-MM-DD-feature-1/   ← feature spec
│   │   ├── requirements.md
│   │   ├── plan.md
│   │   └── validation.md
│   └── YYYY-MM-DD-feature-2/   ← next feature
│       ├── requirements.md
│       ├── plan.md
│       └── validation.md
├── src/                        ← actual code
├── skills/                     ← agent skills
│   ├── changelog/
│   │   ├── SKILL.md
│   │   └── scripts/changelog.py
│   └── feature-spec/
│       └── SKILL.md
├── prompts.md                  ← the prompts you use with the agent
├── CHANGELOG.md                ← generated by the skill
├── GUIDE.md                    ← this file
└── README.md                   ← stakeholder input + context
```
