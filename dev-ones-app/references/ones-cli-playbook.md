# ONES CLI Playbook

Use this file when you need command-level guidance for creating, developing, building, or installing a ONES App.

## Install Or Upgrade

If the environment does not have a usable `ones` command, or `ones` / `ones app` reports errors, install or upgrade the CLI with:

```bash
curl -fsSL "https://open.ones.cn/quickstart.sh" | bash
```

## Core Commands

### `ones create`

Use `ones create` to create a new ONES App project.

### `ones specs fetch`

Use:

```bash
ones specs fetch --dir "/path/to/project"
```

Rules:

- replace `"/path/to/project"` with the real project path
- keep the surrounding double quotes

### `ones dev`

Use `ones dev` only when local development, debugging, or troubleshooting requires it.

### `ones build`

Use `ones build` as the final packaging verification command.

Success means:

- the command succeeds
- an `.opkx` file is generated

Do not treat plain `npm run build` as the final packaging verification.

### `ones login`

Use `ones login` when remote CLI actions require authentication.

### `ones app install`

Use `ones app install` when the user is still in a development workflow and wants to install or update the app directly from the project.

If the user wants to install after running `ones build`, guide them to upload the generated `.opkx` file in the ONES app management page.
