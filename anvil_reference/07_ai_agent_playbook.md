# AI Agent Playbook

## First Pass Inspection

Before editing an Anvil app, inspect:

```bash
find . -maxdepth 3 -name anvil.yaml -o -path './client_code/*' -o -path './server_code/*'
rg -n "startup|startup_form|dependencies|runtime_options|services|db_schema" anvil.yaml
rg -n "open_form|routing|navigate|set_url_hash|launch" client_code server_code
rg -n "@anvil.server.callable|@anvil.server.background_task|http_endpoint|in_transaction" server_code
```

Then identify:

- Startup Form or startup Module.
- Navigation/routing mechanism.
- Theme and role conventions.
- Whether Forms are packages or older module Forms.
- Data Table schema and client permissions.
- Existing server/service layer patterns.
- Custom components and dependencies.

If the Anvil CLI is installed, use it for lightweight local checks, but do not start syncing until the target app/account is known:

```bash
anvil version
anvil validate anvil.yaml
```

## Making A UI Change

Checklist:

- Find the Form package under `client_code/`.
- Read both `__init__.py` and `form_template.yaml`.
- Search for the component name across the app before renaming it.
- Preserve existing component names unless a rename is necessary.
- Add new components to YAML with unique names.
- Add event handlers in Python with `**event_args`.
- Put component-owned settings in `properties`.
- Put parent layout settings in `layout_properties`.
- Use existing roles and theme tokens before creating new roles.
- Add CSS to `theme/assets/theme.css` using role selectors.
- Avoid app-wide selectors such as `.btn` unless the app already uses them intentionally.

When adding components dynamically, use the parent container's `add_component()` API rather than manually manipulating DOM.

## Making A Server/Data Change

Checklist:

- Keep secrets and external API credentials server-side.
- Add or update a server function in `server_code/`.
- Use `@anvil.server.callable(require_user=True)` for user-specific operations.
- Recheck object-level permissions in the callable.
- Prefer `client: none` Data Table permissions unless app design requires otherwise.
- Wrap multi-row updates in `@anvil.tables.in_transaction`.
- Use `q.fetch_only(...)` for large linked data where only a few columns are needed.
- Return only the rows/data the client should see.

Never trust:

- IDs supplied by client code.
- Hidden or disabled UI controls.
- Client-side validation.
- The current user stored in client variables.

## Adding A New Form

Create:

```text
client_code/NewForm/
  __init__.py
  form_template.yaml
```

Python skeleton:

```python
from ._anvil_designer import NewFormTemplate
from anvil import *
import anvil.server


class NewForm(NewFormTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
```

YAML skeleton:

```yaml
is_package: true
container:
  type: ColumnPanel
  properties:
    col_spacing: none
components: []
```

If the app uses layouts or routing, follow an existing similar page instead of this skeleton.

## Editing YAML Safely

Do:

- Use a YAML parser for structured edits when practical.
- Keep two-space indentation.
- Preserve ordering unless layout should change.
- Preserve `grid_position` values for existing components.
- Copy nearby component patterns.
- Verify handler method names exist.
- Verify custom component references exist.
- Run `anvil validate anvil.yaml` or `anvil validate client_code/<FormName>/form_template.yaml` when the Anvil CLI is available.

Avoid:

- Reformatting the entire YAML file.
- Moving `layout_properties` into `properties` or vice versa without checking ownership.
- Adding duplicate names.
- Deleting designer metadata you do not understand.
- Creating huge nested component trees when a reusable Form/custom component is clearer.

## CSS Workflow

1. Find existing roles in YAML and `theme/assets/theme.css`.
2. Reuse an existing role if it matches the semantic purpose.
3. If adding a role, make it semantic and local to the app.
4. For layout fixes, use composable utility roles.
5. Target actual Anvil wrappers.
6. Test responsive breakpoints.

Useful selectors:

```css
/* ColumnPanel row gap */
.anvil-role-my-stack > .anvil-panel-section + .anvil-panel-section {}

/* FlowPanel flex container */
.anvil-role-my-row > .flow-panel-gutter {}

/* FlowPanel child wrapper */
.anvil-role-my-row > .flow-panel-gutter > .flow-panel-item {}

/* Button actual button */
.anvil-role-my-button > button.btn {}

/* DataGrid column */
.anvil-data-row-panel .data-row-col[data-grid-col-id="name"] {}
```

If a selector does nothing, inspect the rendered DOM. Runtime prefixing and wrapper depth vary.

## Data Binding Workflow

Use data bindings for simple UI projection:

```yaml
data_bindings:
- property: text
  code: self.item["name"]
```

Use writeback for simple assignments:

```yaml
data_bindings:
- property: text
  code: self.item["name"]
  writeback: true
```

Use event handlers for:

- Validation.
- Server saves.
- Side effects.
- Cross-field updates.
- Transformations that are not assignable.

After mutating state:

```python
self.can_save = self.compute_can_save()
self.refresh_data_bindings()
```

## Navigation Workflow

Find the current pattern first:

```bash
rg -n "open_form|content_panel|routing|navigate|set_url_hash" client_code
```

If simple `open_form`:

```python
open_form(Home())
```

If content panel:

```python
self.content_panel.clear()
self.content_panel.add_component(Details(item=item))
```

If routing:

- Use the existing routing import.
- Add route registration where the app already registers routes.
- Navigate with the existing helper.
- Pass URL-safe ids or query values.
- Rehydrate and authorize on the server.

## Verification

For docs-only changes:

```bash
find anvil_reference -type f -maxdepth 1 -print
```

For app changes:

- Run any project-specific tests or scripts first.
- Validate changed YAML with the Anvil CLI when available.
- If no tests exist, start the App Server locally:

```bash
anvil-app-server --app .
```

- If the intended workflow is hosted-editor sync rather than local App Server, run `anvil watch` only after verifying the Anvil app id, branch, account, and server URL. Use `anvil watch --staged-only` when you want Git staging to control what syncs.
- Open the relevant page.
- Exercise the edited event handlers.
- Watch browser console and server logs.
- Check mobile/desktop layout for CSS changes.

If dependency installation or network access is required and fails under sandboxing, ask for permission rather than working around it.

## Common Failure Modes

Missing handler:

- YAML has `event_bindings: {click: btn_click}` but Python lacks `btn_click`.

Wrong property location:

- Parent layout properties are placed under `properties` or component properties under `layout_properties`.

FlowPanel CSS no-op:

- CSS targets `.anvil-role-row` instead of `> .flow-panel-gutter`.

Clipped shadows/dropdowns:

- `.anvil-panel-col` has `overflow-x: hidden`; use an overflow utility role.

Unexpected DataGrid values:

- DataRowPanel `auto_display_data` uses column `data_key`; check column ids and item shape.

Client security bug:

- A server function trusts a client-provided user id or row id without checking ownership.

Data binding writeback error:

- Binding expression is not assignable.

Search performance issue:

- Code converts a huge search iterator to a list or follows linked rows without `fetch_only`.

Model class confusion:

- Model classes subclass `app_tables.<table>.Row`, not a generic framework `Model` base in the current runtime.

## Final Response Checklist For Agents

When reporting back:

- State what files changed.
- Mention any version-specific assumptions that affected the result.
- Say what verification was run.
- Say what was not run and why.
- Keep app-specific preferences separate from general framework facts.
