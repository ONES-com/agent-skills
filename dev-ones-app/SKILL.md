---
name: dev-ones-app
description: Evaluate, design, implement, and deliver ONES Apps (ONES Plugin / `.opkx`) against local ONES App specs. Use for feasibility assessment, capability mapping, implementation planning, and delivery across OpenAPI, Hosted API, Events, Extensions, Web SDK, and manifest/runtime configuration.
---

## Goal
Convert ONES App requirements into spec-backed `.opkx` solutions that can be evaluated, built, and delivered.

## Scope
- Clarify requirements, assess feasibility, map capabilities, design solutions, implement features, and deliver ONES Apps (`.opkx`).
- Validate OpenAPI, Hosted API, Events, Extensions, Web SDK, and manifest/runtime capabilities against `.ones/ones-app-specs`.
- Provide a minimum viable demo path when the user needs a starting point.

## Spec Source And Selection Rules

- Prefer project-level specs: `<project>/.ones/ones-app-specs`
- If project-level specs do not exist, use global specs: `~/.ones/ones-app-specs`
- Install the `ones` CLI with `npm install -g @ones-open/cli`
- When working inside a project, run `ones specs fetch --dir "$(pwd)"` first to ensure the specs match the current project version
- Requirement evaluation, capability mapping, and implementation decisions must all be traceable to local spec files. Do not answer from memory.

## Default Flow For Requirement Evaluation

Use this default flow when the user asks "can this be done", "how do I do this", or "what scopes / events / extensions / manifest fields are required":

1. Load `references/ones-app-specs-playbook.md` first
2. Normalize the capability terms and business object terms in the requirement
3. Inspect `capability-tree.json` and `guide/guide-index.json` first to narrow the search scope
4. Then use `rg` to search `openapi-spec.yaml`, `hosted-api/*.yaml`, `schemas/**/*.json`, and `guide/**/*.mdx`
5. Output a capability evidence table. If there is no local evidence, classify it as `REQUIRES CLARIFICATION` or `unsupported` according to the playbook rules

## First Response Requirements

After receiving a requirement, the first reply should include these three sections by default:

1. Requirement understanding checklist
- List the requirement points in numbered form
- Mark high-risk assumptions as `Pending confirmation`

2. Capability mapping draft
- List items in the format "requirement point -> ONES capability type -> initial evidence entry point"
- List temporarily unmapped items separately

3. Next actions
- State which spec files will be searched next
- If critical information is missing, ask only the minimum questions that affect capability judgment

## Standard Flow From Requirement To Implementation

1. Requirement collection and clarification
- When the user requirement is incomplete, fill in the key details that affect capability judgment first: runtime mode, OAuth / scope, event source, extension entry point, storage, and delivery form
- When the user has no clear requirement, provide a minimum viable demo path

2. Requirement confirmation
- Repeat back the understanding in a numbered list
- Clearly distinguish what the user has confirmed from what is still an assumption

3. Feasibility evaluation
- Every requirement point must map to local spec evidence
- Mark it as `supported` when clear evidence exists
- Mark it as `REQUIRES CLARIFICATION` when the user's wording is too unclear to decide
- After targeted search, if there is still no matching capability evidence, mark it directly as `unsupported`

4. Solution design (complex requirements)
- Complex requirements must cover: manifest changes, backend endpoints / services, frontend pages / Web SDK, storage model, authentication / scope, and verification path

5. Task breakdown and progress tracking (complex requirements)
- Break the work into executable tasks, covering at least: spec mapping, manifest, backend, frontend, storage, and verification
- Continuously update current progress and blockers during execution

6. Development and verification
- Complete the minimum vertical slice first, then add enhancements
- At minimum, verify build, key callbacks, key permissions, and key page or API paths

7. Delivery
- Produce the `.opkx`
- Explain installation steps, verification steps, known limitations, and pending confirmations

## Mandatory Command Rules

- When creating a project, use `ones create`. Do not replace it with manual scaffolding unless the user explicitly asks for that.
- Prefer `ones build` to generate the deliverable.
- Do not use `ones dev` unless the task is local development, debugging, or issue diagnosis.
- The tools for debugging an app against a real ONES environment are `ones app`, `ones dev`, and `ones tunnel`. Before using them, the user must manually run `ones login` to ensure they are logged in. Use `ones whoami` to confirm the login status.

## Recommended Development Workflow

1. Create the project
> If the user specified a directory and it already contains `opkx.json`, this step can be skipped.

```bash
ones create <project-path>
# Optional: generate only the manifest (opkx.json)
ones create <project-path> --manifest-only
```

2. Pull the spec documents into the project workspace

```bash
cd <project-path>
ones specs fetch --dir "$(pwd)"
```

3. Build and produce the artifact (default path)

```bash
cd <project-path>
ones build
# Optional examples
ones build --output ./dist/app.opkx
ones build --command "pnpm build"
```

4. Use dev mode only when necessary (exception path)

```bash
cd <project-path>
ones dev
# Optional examples
ones dev --install
ones dev --command "pnpm dev"
```

5. Install and verify
If the user is already logged in and has confirmed that the app is installed, the following command can be used to fetch app logs:
```bash
ones app logs <app-id>
# Optional examples
ones app logs --from-opkx-json
ones app logs --tail 100
```

`ones build` is the default schema validation gate for OPKX. Do not introduce extra custom OPKX schema validation logic unless the user explicitly asks for it.

## Complex Requirement Criteria

A requirement is considered complex if any of the following conditions are met:

- It combines 2 or more capability categories, such as Events + OpenAPI, Extensions + Storage, or Web SDK + OAuth
- It involves cross-entity data flow, aggregation, permission boundaries, or multi-role workflows
- It requires engineering details such as settings pages, storage models, error handling, retries, or async callbacks

For complex requirements, present the solution first, then move to implementation.

If command behavior is uncertain, check the help first:

```bash
ones --help
ones create --help
ones specs -h
ones specs fetch -h
ones dev --help
ones build --help
```

## How To Use `.ones/ones-app-specs`
Follow [references/ones-app-specs-playbook.md](references/ones-app-specs-playbook.md) and use the files in `.ones/ones-app-specs` to evaluate requirements, design solutions, and implement features.

## Capability Mapping Principles

- Determine the runtime mode first
  - Externally hosted app: use `schemas/app_manifest_schema.json`
  - ONES hosted app (`ones create` project, default): use `schemas/app_opkx_schema.json`
- Use Guide files to understand flows, examples, and caveats
- Use Schema files to confirm manifest fields, event / extension contracts, enums, and required fields
- Use OpenAPI / Hosted API files to confirm paths, methods, scopes, parameters, and responses
- Do not invent endpoints / fields / contracts / manifest keys
- When the user's wording is too unclear to decide, mark it as `REQUIRES CLARIFICATION`
- If targeted searches across the relevant guide, schema, OpenAPI, and Hosted API files still find no local evidence, mark it explicitly as `unsupported`
- Unless the user explicitly asks for an alternative route, do not invent manual fallback solutions for missing capabilities
- Keep implementation decisions linked to evidence files so auth / scope / contract stays aligned with the spec
- Before finishing, confirm every delivered feature has corresponding evidence and run `ones build`

## Output Standard

Evaluation or implementation output should include these sections by default:

- Requirement understanding checklist
- Capability evidence table
- For complex requirements, an implementation checklist covering: manifest, backend, frontend, storage, and verification

The capability evidence table should include these fields:

- requirement
- capability type (`openapi`, `hosted_api`, `event`, `extension`, `websdk`, `schema`)
- evidence file path
- evidence pointer (`#/paths/...`, schema field, or document section)
- decision (`supported`, `unsupported`, or `REQUIRES CLARIFICATION`)

You may reuse templates and command snippets from:

- `references/ones-app-specs-playbook.md`

## Definition Of Done (DoD)

At minimum, a completed task satisfies:

- Requirement understanding and key assumptions are aligned
- Every implemented capability has local spec evidence
- Unsupported items are clearly marked as `unsupported`, and pending items are clearly marked as `REQUIRES CLARIFICATION`
- Complex requirements include a solution or a task breakdown
- The app can generate a `.opkx` through `ones build`
- Installation and minimum verification guidance has been provided

## Non-Negotiable Constraints

- Use `ones create` by default when creating a project.
- Run `ones specs fetch --dir "$(pwd)"` before capability mapping.
- Prefer `ones build` to generate the artifact, and treat it as the final OPKX schema validation gate.
- Use `ones dev` only when local development / debugging / troubleshooting requires it.
- The tools `ones app`, `ones dev`, and `ones tunnel` are for debugging an app against a real ONES environment. Before using them, the user must manually run `ones login` to ensure they are logged in. Use `ones whoami` to confirm the login status.
- Treat the schemas in `.ones/ones-app-specs` as reference material, not as an independent validation system.
- Every capability claim must be traceable to local `.ones/ones-app-specs` files.
- When evidence is missing, stop and ask for clarification.

## Common Troubleshooting

- If a `ones` subcommand does not exist, reports `unknown command`, or does not appear in `ones --help`, do not assume the command is available.
- First confirm the current CLI version, then try upgrading to the latest version and retry:

```bash
ones --version
npm install -g @ones-open/cli@latest
ones --version
ones --help
```

- If the command is still unavailable after upgrading, follow the actual `ones --help` output for the current environment and treat that command as unavailable there. Do not keep giving usage based on memory.
