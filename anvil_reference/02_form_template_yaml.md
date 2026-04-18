# Form Template YAML

## Purpose

`form_template.yaml` is the visual designer's serialized form definition. It stores:

- The root container or layout.
- Component hierarchy.
- Component properties.
- Parent-specific layout properties.
- Event handler mappings.
- Data bindings.
- Custom component and slot metadata.

The form's behavior lives in `__init__.py`; the designer structure lives in YAML.

## Package Form Files

Modern package Form:

```text
client_code/MyForm/
  __init__.py
  form_template.yaml
```

Older module Form:

```text
client_code/MyForm.py
client_code/MyForm.yaml
```

## Classic Form Shape

A classic Form inherits from a root container such as `ColumnPanel` or `HtmlTemplate`.

```yaml
is_package: true
container:
  type: ColumnPanel
  properties:
    col_spacing: none
  event_bindings:
    show: form_show
components:
- name: lbl_title
  type: Label
  properties:
    text: Hello
    role: headline
  layout_properties:
    grid_position: ABCDEF,GHIJKL
```

Runtime schema fields include:

- `class_name`: Form class name. Some exported files omit this when inferable.
- `is_package`: Whether the Form is stored as a package.
- `container`: Root classic container.
- `components`: Top-level components added to the root container.
- `custom_component`: Marks a Form as a configured custom component.
- `custom_component_container`: Exposes container behavior for a custom component.
- `properties`: Custom component property definitions.
- `events`: Custom component event definitions.
- `slots`: Slot definitions exposed by this Form.
- `item_type`: Optional table item type metadata.

## Layout-Based Form Shape

Forms can also use a layout rather than inherit directly from a root container:

```yaml
layout:
  type: form:Layouts.MainLayout
  properties: {}
  event_bindings:
    show: layout_show
  form_event_bindings:
    show: form_show
components_by_slot:
  default:
  - name: content_panel
    type: ColumnPanel
    properties: {}
    layout_properties:
      slot: default
```

Key differences:

- `layout` describes the layout component.
- `components_by_slot` groups child components by named slot.
- `form_event_bindings` attach handlers to the Form itself.
- `event_bindings` on `layout` attach handlers to the layout instance.

## Component Definition

Every component entry is a dict:

```yaml
- name: btn_save
  type: Button
  properties:
    text: Save
    role: primary-action
    enabled: true
  layout_properties:
    grid_position: ROW123,COL456
  event_bindings:
    click: btn_save_click
  data_bindings:
  - property: enabled
    code: self.can_save
    writeback: false
  components:
  - name: nested_label
    type: Label
    properties:
      text: Nested
```

Fields:

- `name`: Python attribute name on the Form, for example `self.btn_save`.
- `type`: Built-in component type such as `Button`, or a `form:...` custom component reference.
- `properties`: Properties passed to the component constructor.
- `layout_properties`: Keyword arguments passed to the parent container's `add_component`.
- `event_bindings`: Maps event names to methods on the Form.
- `data_bindings`: Designer data binding expressions.
- `components`: Child component tree for containers.

Component names must be unique within a Form. If a component name collides with a method or attribute, the component attribute can hide that method/attribute.

## Properties Versus Layout Properties

Use `properties` for properties owned by the component:

```yaml
properties:
  text: Save
  role: primary-action
  visible: true
  spacing_above: none
  spacing_below: none
```

Use `layout_properties` for values consumed by the parent container:

```yaml
layout_properties:
  grid_position: ABCDEF,GHIJKL
  full_width_row: true
  row_background: "theme:Gray 100"
```

Common parent layout properties:

| Parent | Layout properties |
| --- | --- |
| `ColumnPanel` | `grid_position`, `full_width_row`, `row_background` |
| `FlowPanel` | `width`, `expand` |
| `GridPanel` | `row`, `col_xs`, `width_xs`, `col_sm`, `width_sm`, `col_md`, `width_md`, `col_lg`, `width_lg` |
| `DataGrid` | `pinned`, `slot` |
| `DataRowPanel` | `column` |
| `HtmlTemplate` | `slot`, `index` |

Unknown layout properties are passed to the parent and may be ignored. Avoid moving component-owned properties into `layout_properties` unless the existing app already does so for a known reason.

## Event Bindings

YAML event binding:

```yaml
event_bindings:
  click: btn_save_click
```

Form method:

```python
def btn_save_click(self, **event_args):
    ...
```

At runtime, Anvil looks up `btn_save_click` on the Form and registers it as the handler. If the method is missing or not callable, the runtime warns and the handler is not attached.

Decorated event handlers in Python can coexist with YAML handlers. If a decorator handles the same event for the same component, the decorator wins for that event.

All event callbacks receive `sender` and `event_name` in `event_args`, plus event-specific parameters such as `keys`, `x`, `y`, or `file`.

Custom events should use the `x-` prefix unless they are declared as custom component events.

## Data Bindings

YAML data binding:

```yaml
data_bindings:
- property: text
  code: self.item["name"]
  writeback: true
```

Runtime behavior:

- On `refresh_data_bindings()`, Anvil evaluates `code` and assigns the result to the component property.
- If `writeback: true`, Anvil also generates assignment code from the component property back to the expression.
- Writeback expressions must be assignable. `self.item["name"]` is assignable; `self.first_name + " " + self.last_name` is not.
- `refresh_data_bindings()` updates UI from data. It does not force all writebacks from UI to data.
- Setting a Form's `self.item` triggers `refresh_data_bindings()`. Mutating inside `self.item`, such as `self.item["name"] = "Ada"`, does not automatically trigger a refresh; call `self.refresh_data_bindings()` afterward.

Keep data binding expressions simple. Put expensive lookups, server calls, and side effects in Python methods or event handlers.

## Custom Component References

Built-in components use simple names:

```yaml
type: Button
type: ColumnPanel
type: RepeatingPanel
```

Forms/custom components use `form:` references:

```yaml
type: form:components.UserBadge
type: form:DEPENDENCY_ID:widgets.Tabulator
```

Exact formats vary by app and dependency. Copy a nearby existing reference when adding similar components.

## Validating YAML

If the Anvil CLI is installed, validate edited app or Form YAML before running the app:

```bash
anvil validate client_code/MyForm/form_template.yaml
```

`anvil validate <file>` catches schema problems, but it does not prove that referenced Python handler methods exist or that the UI behaves correctly at runtime.

## HTML Template Slots

HTML templates declare slots in theme HTML:

```html
<div anvil-slot="default"></div>
```

YAML places components into slots:

```yaml
components:
- name: content_panel
  type: ColumnPanel
  properties: {}
  layout_properties:
    slot: default
```

HTML template helpers:

- `anvil-slot="name"`: Named insertion point.
- `anvil-slot-repeat="name"`: Repeat slot template for multiple children.
- `anvil-name="name"`: Exposes DOM node via `self.dom_nodes["name"]`.
- `anvil-hide-if-slot-empty="name"`: Hide element when slot is empty.
- `anvil-if-slot-empty="name"`: Show element when slot is empty.

## Common Component YAML Patterns

ColumnPanel:

```yaml
name: pnl_stack
type: ColumnPanel
properties:
  col_spacing: none
  role: vertical-stack
```

FlowPanel:

```yaml
name: pnl_actions
type: FlowPanel
properties:
  align: right
  gap: none
  vertical_align: middle
  role: action-row
```

GridPanel:

```yaml
name: pnl_grid
type: GridPanel
properties:
  role: compact-grid
components:
- name: btn_left
  type: Button
  properties:
    text: Left
  layout_properties:
    row: A
    col_xs: 0
    width_xs: 6
```

RepeatingPanel:

```yaml
name: repeating_panel_users
type: RepeatingPanel
properties:
  item_template: Users.UserRow
  items: []
```

DataGrid:

```yaml
name: data_grid_users
type: DataGrid
properties:
  auto_header: true
  rows_per_page: 20
  show_page_controls: true
  columns:
  - id: name
    title: Name
    data_key: name
    expand: true
  - id: email
    title: Email
    data_key: email
    width: 240
```

DataRowPanel:

```yaml
name: row_panel
type: DataRowPanel
properties:
  auto_display_data: true
  item: {}
```

Timer:

```yaml
name: timer_poll
type: Timer
properties:
  interval: 0
event_bindings:
  tick: timer_poll_tick
```

`interval: 0` disables the timer.

## Safe Editing Rules

When editing YAML by hand or by agent:

- Parse and emit YAML with a YAML parser when possible.
- Preserve component order unless intentionally changing layout order.
- Preserve existing component names and `grid_position` values.
- Add event handlers in Python before wiring them in YAML.
- Keep `properties` and `layout_properties` separate.
- Avoid broad normalization that rewrites the whole file.
- Do not duplicate component `name` values.
- Use role arrays for composed roles:

```yaml
properties:
  role: [primary-action, loading]
```

Before finalizing, check that every YAML handler name exists in the Form Python and that every referenced custom component or dependency import exists.
