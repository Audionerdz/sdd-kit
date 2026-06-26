# Spec-Driven Development Kit

**Template to start any project using Spec-Driven Development with AI coding agents.**

This repo contains the essential elements from the [Spec-Driven Development course by DeepLearning.AI](https://www.deeplearning.ai/courses/), cleaned of AgentClinic-specific code. Use it as a starting point for new projects.

## What's included?

| Folder | Contents |
|--------|----------|
| `prompts/` | All course prompts organized by lesson |
| `example_specs/` | Example specification documents (mission, tech stack) |
| `skills/` | Reusable agent skills (changelog, feature-spec) |

## How to use

1. Copy this folder as the base for your project:

```bash
cp -r sc-spec-driven-development-files/ my-new-project/
cd my-new-project
```

2. Write your `README.md` with your project context and stakeholder input.

3. Use an AI coding agent (opencode, Claude Code, etc.) with the prompts in `prompts/` to create your constitution (`specs/mission.md`, `tech-stack.md`, `roadmap.md`).

4. Follow the SDD flow: Constitution → Feature Specs → Implementation → Validation.

## Requirements

- Git
- An AI coding agent
