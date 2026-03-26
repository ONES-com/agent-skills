# ONES App Dev Playbook

Use this file when implementing app-specific frontend and backend development work.

## App API

Frontend pages in the ONES sandbox should call app backend routes through `ONES.fetchApp`.

App APIs are custom business routes defined by the current ONES App itself.

- there is no fixed platform-wide app API contract
- design the request path, request parameters, and response shape from the current business context
- implement the frontend request code and backend route code together

Guidance:

- trace each `ONES.fetchApp` path into a backend route before trusting it
- if no route exists, treat it as a likely bug or incomplete implementation

## Frontend

Before choosing UI components, inspect the current project's ONES Design usage and follow the conventions already used in the workspace.

Do not rely on this document to enumerate all available components.

Instead, inspect the project-installed `@ones-design/core` module to discover which components are available in the current workspace.

Component guidance:

- prefer `@ones-design/table` for table scenarios
- prefer `@ones-design/core` for basic interaction components such as `Button`, `Modal`, and `Space`
