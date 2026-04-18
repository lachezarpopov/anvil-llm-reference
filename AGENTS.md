# AGENTS.md

This repository contains an Anvil app or Anvil app reference material. Before making changes to an Anvil application, use the portable reference docs in `anvil_reference/`.

## Required Anvil Reference Check

If `anvil_reference/README.md` exists, read it before making Anvil-specific changes. Then read the most relevant reference files for the task:

- App structure, `anvil.yaml`, services, Anvil CLI, local App Server: `anvil_reference/01_app_structure_and_runtime.md`
- Form YAML, component trees, event/data bindings, layout properties: `anvil_reference/02_form_template_yaml.md`
- Form lifecycle, component APIs, `self.item`, data binding writeback: `anvil_reference/03_forms_components_and_data_bindings.md`
- CSS, roles, generated DOM wrappers, ColumnPanel/FlowPanel/GridPanel quirks: `anvil_reference/04_layout_dom_css.md`
- Server modules, Data Tables, model classes, transactions, trust boundaries: `anvil_reference/05_server_data_tables_and_security.md`
- Custom components, dependencies, HTML templates, routing: `anvil_reference/06_custom_components_dependencies_and_routing.md`
- Agent workflow checklist: `anvil_reference/07_ai_agent_playbook.md`
- Portability, app-specific conventions, drift risks: `anvil_reference/08_scope_and_maintenance_notes.md`

If `anvil_reference/` is missing, proceed carefully from the app itself and official Anvil docs when current API details are needed.

## Local Source Of Truth

The target app always wins over generic reference guidance. Inspect the app before editing:

- `anvil.yaml`
- `client_code/`
- `server_code/`
- `theme/parameters.yaml`
- `theme/assets/theme.css`
- Relevant `form_template.yaml` files
- Existing navigation/routing patterns
- Existing custom component and role naming conventions

Do not impose role names, folder structure, CSS tokens, routing libraries, or data-access patterns from another app.

## Editing Anvil Forms And YAML

When changing a Form, read both its Python file and designer YAML:

- Package Form Python: `client_code/<FormName>/__init__.py`
- Package Form YAML: `client_code/<FormName>/form_template.yaml`

Rules:

- Preserve existing component names unless a rename is required.
- Do not create duplicate component names.
- Put component-owned settings in `properties`.
- Put parent/container layout settings in `layout_properties`.
- Preserve existing `grid_position` values unless intentionally changing layout.
- Add Python event handler methods before wiring them in YAML.
- Verify every YAML event handler name exists in the Form Python.
- Keep YAML diffs minimal and avoid whole-file reformatting.

## CSS And Theme Roles

Use existing roles and theme conventions first. Add new roles only when needed.

Remember:

- Anvil components render with wrapper DOM, not plain HTML.
- FlowPanel flex CSS usually targets `> .flow-panel-gutter`.
- FlowPanel child sizing usually targets `> .flow-panel-gutter > .flow-panel-item`.
- ColumnPanel children are wrapped in `.anvil-panel-section` and `.anvil-panel-col`.
- `.anvil-panel-col` may clip shadows/dropdowns with `overflow-x: hidden`.
- GridPanel rows and Bootstrap-style columns may need gutter overrides.
- Use role selectors such as `.anvil-role-my-role` instead of broad global selectors where possible.

If a selector does not work, inspect the actual rendered DOM for the target app/runtime.

## Server, Data, And Security

Treat client code as untrusted.

Keep these in server code or trusted model methods:

- Authorization checks.
- Business validation.
- Data Table writes unless the app explicitly uses safe client-writable views.
- Secrets and external API credentials.
- Cross-table workflows.
- Long-running jobs and background tasks.

Before exposing a callable or table view, verify:

- Who may call it.
- Whether the current user is checked server-side.
- Whether row/user ids supplied by the client are authorized.
- Whether returned rows expose linked rows or columns unexpectedly.
- Whether multi-row writes should be wrapped in a transaction.

## Data Bindings

Use data bindings for simple UI projection and simple writeback. Use explicit event handlers for validation, server saves, transformations, and side effects.

After mutating form state used by bindings, call `self.refresh_data_bindings()` once after state updates are complete.

Remember that setting `self.item` refreshes bindings, but mutating inside `self.item` does not automatically refresh the UI.

## Routing And Navigation

Do not assume a routing library. Search the app first for:

- `open_form`
- `content_panel`
- `routing`
- `navigate`
- `set_url_hash`

Follow the app's existing navigation style. If routing is present, use the existing import path and route registration pattern.

## Verification

For Anvil app changes:

- Run available project tests or checks first.
- If the Anvil CLI is available, run `anvil validate anvil.yaml` and validate any changed Form YAML.
- If practical, run the app with `anvil-app-server --app .` from the app root.
- Run `anvil watch` only when you intend to sync local files to an Anvil app and know the target app/account.
- Exercise the edited Form/page.
- Watch browser console and server logs.
- Check relevant responsive layouts for CSS changes.

For docs-only changes:

- Check links and filenames.
- Scan for non-portable references if the docs are intended to be copied into other repos.
