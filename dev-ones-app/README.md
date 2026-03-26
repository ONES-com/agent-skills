# dev-ones-app

`dev-ones-app` is an AI skill for evaluating, designing, implementing, and delivering ONES Apps.

It is intended for cases where an agent needs to help with ONES Plugin or `.opkx` development using local ONES App specifications instead of guesswork. The skill is built to keep capability decisions traceable to spec files, so implementation advice stays aligned with the actual ONES platform contract.

## What This Skill Does

This skill helps an agent:

- Assess whether a ONES App requirement is supported
- Map requirements to ONES capabilities such as OpenAPI, Hosted API, Events, Extensions, Web SDK, and schema definitions
- Design implementation plans for frontend, backend, storage, manifest, and verification
- Build and deliver ONES hosted apps with a spec-backed workflow

## Best Fit

Use this skill when you need help with:

- Feasibility analysis for a ONES App idea
- Capability mapping before implementation
- Building a hosted ONES App with `ones create`
- Packaging and validating a `.opkx` deliverable
- Understanding which scopes, events, extensions, or manifest fields are required

## How It Works

The skill follows a strict evidence-first workflow:

1. Find the correct local spec source, preferring project-level `.ones/ones-app-specs`
2. Search index files and guides before deep-searching schemas and API specs
3. Map each requirement to concrete ONES capability evidence
4. Mark unsupported or unclear items explicitly instead of inventing behavior
5. Implement and verify against the actual project structure and CLI workflow

## Key Principles

- Spec-backed, not memory-backed
- Prefer ONES hosted app defaults unless the user explicitly needs external hosting
- Use `ones create` for project creation and `ones build` for packaging
- Keep implementation decisions connected to local evidence files
- Present unsupported or unclear areas directly

## Typical Workflow

1. Clarify the requirement
2. Inspect local spec files
3. Produce a capability evidence table
4. Design the manifest, backend, frontend, storage, and verification approach
5. Implement the minimum viable vertical slice
6. Build the app and deliver the `.opkx`

## Directory Structure

| Path | Purpose |
| --- | --- |
| [`SKILL.md`](./SKILL.md) | Core machine-readable skill instructions |
| [`references/`](./references) | Playbooks and supporting references for spec search and development workflow |
| [`agents/`](./agents) | Agent-related configuration |
| [`assets/`](./assets) | Visual assets used by the skill |

## Usage

Ask the agent to use `dev-ones-app` when the task involves ONES App development. Good examples:

- "Use `dev-ones-app` to evaluate whether this ONES App requirement is supported."
- "Use `dev-ones-app` to design a hosted ONES App for this workflow."
- "Use `dev-ones-app` to build and package this app as a `.opkx`."

The detailed behavior, command rules, and output requirements are defined in [`SKILL.md`](./SKILL.md).

## Related Links

- ONES product website: [ones.com](https://ones.com)
- ONES documentation: [docs.ones.com](https://docs.ones.com)

## Repository Context

This skill is part of the public [ONES Agent Skills](../README.md) repository, which showcases AI skills created around ONES products and workflows.
