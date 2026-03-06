# ONES App Specs Playbook

## Contents

1. Applicable scenarios
2. Determine the spec directory first
3. Normalize the user requirement first
4. Use index files for the first-hop lookup
5. Perform targeted searches by capability type
6. Read matched files and extract evidence
7. Output a capability evidence table
8. What to do when nothing is found
9. Move from evidence to implementation

## 1. Applicable Scenarios

When evaluating whether a requirement is feasible or validating whether a feature is implemented correctly, prefer this playbook:

- Can this requirement be implemented with a ONES App
- What event / extension / OpenAPI / Hosted API / Web SDK / manifest fields are needed
- Whether this capability belongs to a ONES-hosted app or an externally hosted app
- What OAuth scopes, callbacks, runtime, or storage declarations are required

The goal is not to read the entire `.ones/ones-app-specs` at once. Narrow the search scope first, then open only the most relevant files and extract evidence.

## 2. Determine The Spec Directory First

Prefer project-level specs, then global specs. Do not mix specs from two different sources.

```bash
if [ -d .ones/ones-app-specs ]; then
  SPEC_ROOT="$(pwd)/.ones/ones-app-specs"
else
  SPEC_ROOT="$HOME/.ones/ones-app-specs"
fi
```

If the user is already working in a ONES App project, run this first:

```bash
ones specs fetch --dir "$(pwd)"
```

## 3. Normalize The User Requirement First

First split the user's original wording into two groups:

- Capability terms: OpenAPI, OAuth, event, webhook, extension, Web SDK, storage, Manifest, Runtime, Lifecycle, externally hosted, local development
- Business object terms: issue, project, sprint, milestone, wiki, department, team, field, attachment, comment, worklog

### 3.1 Capability Term Normalization Table

| User wording | Normalized keywords | Search first in |
| --- | --- | --- |
| OpenAPI, authorization, scope, token, OAuth | `openapi` `oauth` `scope` `token` | `guide/getting-started/access-openapi.mdx`, `openapi-spec.yaml`, `schemas/app_opkx_schema.json` |
| event, webhook, subscription, callback, notification | `event` `webhook` `subscribe` `eventType` | `guide/getting-started/subscribe-to-events.mdx`, `schemas/events/*.json`, `schemas/app_opkx_schema.json` |
| extension, settings page, custom tab, entry page | `extension` `appSettingPages` `customEntries` `page_url` | `guide/getting-started/use-extensions.mdx`, `schemas/extensions/app-setting-pages-*.json`, `schemas/app_opkx_schema.json` |
| Web SDK, frontend context, get locale, get team info | `websdk` `ONES.getLocale` `ONES.getTeamInfo` | `guide/getting-started/use-websdk.mdx` |
| hosted storage, entity storage, object storage, file upload/download | `storage` `entity` `object` `upload` `download` | `guide/advanced/ones-hosted-app/storage.mdx`, `hosted-api/storage-api.yaml`, `schemas/app_opkx_schema.json` |
| Manifest, opkx, runtime, healthcheck, resources, container | `manifest` `runtime` `healthcheck` `resources` `container` | `guide/advanced/ones-hosted-app/manifest.mdx`, `schemas/app_opkx_schema.json`, `schemas/app_manifest_schema.json` |
| lifecycle, install callback, enabled callback | `lifecycle` `install` `enabled` `lifecycle_callback` | `guide/advanced/app-lifecycle.mdx`, `guide/getting-started/access-openapi.mdx`, `schemas/app_opkx_schema.json` |
| security, permissions, authentication, security boundary | `security` `auth` `oauth` | `guide/advanced/app-security.mdx`, `schemas/app_opkx_schema.json`, `openapi-spec.yaml` |
| third-party login, SSO, directory sync, message notification | `accountThirdparty` `customLoginUrl` `authLoginInfo` `directorySync` `messageNotify` `helpInfo` | `schemas/extensions/account-thirdparty/*.json`, `schemas/app_opkx_schema.json` |
| manhour validation | `manhourValidator` `validate` | `schemas/extensions/manhour-validator-*.json`, `schemas/app_opkx_schema.json` |
| externally hosted, local development, tunnel debugging | `externally hosted` `local development` | `guide/advanced/externally-hosted-app/*.mdx`, `schemas/app_manifest_schema.json` |

### 3.2 Business Object Term Normalization Table

| User wording | Search terms first |
| --- | --- |
| work item, task, requirement, bug, issue | `issue` |
| project | `project` |
| iteration, sprint | `sprint` |
| milestone | `milestone` |
| wiki, page, document | `wiki` `page` |
| department | `department` |
| team | `team` |
| comment | `comment` |
| attachment, uploaded file | `attachment` |
| worklog, manhour | `worklog` `manhour` |
| field, option, dynamic option | `field` `option` |

If the user uses Chinese business terms, convert them to these English keywords first. The hit rate is usually higher.

## 4. Use Index Files For The First-Hop Lookup

Check the indexes first. Do not open the full `openapi-spec.yaml` immediately.

### 4.1 Global Capability Index

```bash
rg -n -i 'openapi|hosted|schema|event|extension|guide|storage|oauth|runtime' \
  "$SPEC_ROOT/capability-tree.json"
```

This file is useful for determining:

- What capability surfaces exist in the current specs
- The real file paths for Hosted API, schema, and guide files
- What event / extension file lists are available

### 4.2 Guide Index

```bash
rg -n -i 'openapi|event|extension|websdk|manifest|storage|lifecycle|security|externally' \
  "$SPEC_ROOT/guide/guide-index.json"
```

This file is useful for determining:

- Which guide is closest to the current requirement
- The guide `title`, `path`, and `category`
- Which `.mdx` files should be opened first

Note: `capability-tree.json` and `guide-index.json` are navigation evidence, not the final contract proving support by themselves.

## 5. Perform Targeted Searches By Capability Type

### 5.1 Search OpenAPI / OAuth / Scope

```bash
rg -n -i 'oauth|scope|token|issue|project|sprint|wiki|department|attachment|comment|field|worklog|copilot|onesql' \
  "$SPEC_ROOT/openapi-spec.yaml"
```

Focus on confirming:

- Whether the path exists
- What the HTTP method is
- What OAuth scopes are required
- Whether the parameters, request body, and response match the requirement

### 5.2 Search Hosted API (current focus: storage)

```bash
rg -n -i 'entity_data|object|get_upload_policy_info|get_download_pre_signed_url|get_metadata|delete|introspect|storage' \
  "$SPEC_ROOT/hosted-api/storage-api.yaml"
```

Focus on confirming:

- Whether it is entity storage or object storage
- What the corresponding path and operationId are
- What schema the request / response points to

### 5.3 Search Events

```bash
find "$SPEC_ROOT/schemas/events" -type f | rg -i 'issue|project|sprint|milestone|wiki|department|user'
rg -n -i 'eventType|issueID|project|sprint|milestone|wiki|department|subscriberID' \
  "$SPEC_ROOT/schemas/events"
```

Focus on confirming:

- Whether the corresponding event file exists
- The exact enum value of `eventType`
- What fields exist in the event payload

### 5.4 Search Extensions

```bash
find "$SPEC_ROOT/schemas/extensions" -type f | rg -i 'app-setting-pages|account-thirdparty|manhour-validator'
rg -n -i 'appSettingPages|customEntries|accountThirdparty|customLoginUrl|directorySync|messageNotify|helpInfo|manhourValidator' \
  "$SPEC_ROOT/schemas/app_opkx_schema.json" "$SPEC_ROOT/schemas/extensions"
```

Focus on confirming:

- Which extension key should be declared in the manifest
- What the backend endpoint request / response contract is
- Whether there are required funcs / config / slots

### 5.5 Search Manifest / Runtime / Storage Declarations

```bash
rg -n -i 'lifecycle_callback|oauth|events|extensions|ones.storage|runtime|healthcheck|resources|container|ones-node-22' \
  "$SPEC_ROOT/schemas/app_opkx_schema.json" \
  "$SPEC_ROOT/schemas/app_manifest_schema.json"
```

Focus on confirming:

- Whether the field is allowed
- Whether this is a hosted app or an externally hosted app
- What required fields, enum values, patterns, and constraints apply

### 5.6 Search Guide Files For Flow And Example Support

```bash
rg -n -i 'open api|oauth|events|webhook|extensions|web sdk|storage|manifest|runtime|lifecycle|security' \
  "$SPEC_ROOT/guide"
```

Guide files are useful for answering:

- Recommended implementation order
- How to build a minimum demo
- What the usage examples are for callbacks, frontend pages, and storage
- Common pitfalls and caveats

## 6. Read Matched Files And Extract Evidence

After finding matches, open only the relevant snippets. Do not dump whole large files into context.

```bash
sed -n '120,220p' "$SPEC_ROOT/schemas/app_opkx_schema.json"
sed -n '1,200p' "$SPEC_ROOT/guide/getting-started/subscribe-to-events.mdx"
sed -n '900,1040p' "$SPEC_ROOT/openapi-spec.yaml"
```

Follow these rules when extracting evidence:

- `guide/**/*.mdx`: use to prove implementation flow, sample code, and caveats
- `schemas/**/*.json`: use to prove manifest fields, event payloads, extension contracts, enums, and required fields
- `openapi-spec.yaml` / `hosted-api/*.yaml`: use to prove paths, methods, scopes, parameters, and responses
- If the answer involves manifest fields, cite at least one schema file
- If the answer involves API calls, cite at least one OpenAPI / Hosted API file
- If the answer involves the minimum implementation path, it is best to cite both a guide and a contract file

## 7. Output A Capability Evidence Table

In capability evaluation or solution design, output an evidence table by default:

| requirement | capability type | evidence file path | evidence pointer | decision |
| --- | --- | --- | --- | --- |
| Subscribe to the issue-created event | event | `schemas/events/issue-created-schema.json` | `eventType = ones:project:issue:created` | supported |
| Declare the event callback in the manifest | schema | `schemas/app_opkx_schema.json` | `#/properties/app/properties/events` | supported |

Field meanings:

- `requirement`: an atomic capability split from the user requirement
- `capability type`: `openapi`, `hosted_api`, `event`, `extension`, `websdk`, `schema`
- `evidence file path`: exact file path
- `evidence pointer`: `#/properties/...`, `#/paths/...`, enum value, section title, or key field name
- `decision`: `supported`, `unsupported`, or `REQUIRES CLARIFICATION`

## 8. What To Do When Nothing Is Found

Fallback in this order:

1. Search again using English synonyms
2. Search file names first, then file contents
3. Search capability terms first, then add business object terms
4. Check `guide-index.json` for the closest guide
5. If there is still no local evidence:
   - Mark it as `REQUIRES CLARIFICATION` when the requirement itself is unclear
   - Mark it as `unsupported` when the requirement is already clear but there is no capability evidence

Do not do these things:

- Do not assume a schema / contract is supported just because a guide contains an example
- Do not invent non-existent `eventType` values or extension keys just because a similar object exists in OpenAPI
- Do not treat directory information in `capability-tree.json` as the final contract

## 9. Move From Evidence To Implementation

When a requirement has been judged as `supported`, continue by turning the evidence into an implementation checklist. At minimum, inspect the following 5 areas of change:

### 9.1 Manifest Changes

Check what needs to be added or changed in `opkx.json`:

- `app.oauth.scope`
- `app.oauth.type`
- `app.events`
- `app.extensions`
- `app.lifecycle_callback`
- `ones.storage`
- `runtime`

Preferred sources:

- `schemas/app_opkx_schema.json`
- `schemas/app_manifest_schema.json`
- `guide/advanced/ones-hosted-app/manifest.mdx`

### 9.2 Backend Changes

Check what backend interfaces or services are required:

- lifecycle callback
- event webhook
- extension endpoint
- OpenAPI / Hosted API client
- storage read / write / query

Preferred sources:

- `guide/getting-started/access-openapi.mdx`
- `guide/getting-started/subscribe-to-events.mdx`
- `guide/getting-started/use-extensions.mdx`
- `guide/advanced/ones-hosted-app/storage.mdx`
- the corresponding schema / OpenAPI / Hosted API files

### 9.3 Frontend / Web SDK Changes

If the requirement involves pages, settings entries, context information, or frontend calls:

- Confirm the extension type and response structure
- Confirm the page entry and `page_url`
- Confirm whether Web SDK is needed to get locale, team, or context

Preferred sources:

- `guide/getting-started/use-extensions.mdx`
- `guide/getting-started/use-websdk.mdx`
- `schemas/extensions/*.json`

### 9.4 Verification Changes

Prepare minimum verification actions for each key capability:

- Whether the manifest changes build successfully
- Whether callback paths exist and return the correct status code
- Whether the scope is sufficient for API calls
- Whether events can be triggered
- Whether extension pages can render
- Whether storage can complete a minimum write / query / download path

At minimum, run:

```bash
ones build
```

### 9.5 Implementation Checklist Template

For complex requirements, output an implementation checklist by default, covering at least:

| area | change | evidence |
| --- | --- | --- |
| manifest | manifest fields to change | schema / guide |
| backend | new endpoints, services, callbacks | guide / api / schema |
| frontend | new pages, Web SDK calls | guide / extension schema |
| storage | entity / object storage declarations and read-write operations | storage guide / hosted api |
| verification | build, install, trigger, observation points | guide / command |

If any item has no supporting local evidence, do not proceed to implementation. Mark it as `unsupported` or `REQUIRES CLARIFICATION` first.
