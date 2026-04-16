# Forms, Components, And Data Bindings

## Form Python Files

A Form's Python file usually imports its generated template class and subclasses it:

```python
from ._anvil_designer import UserProfileTemplate
from anvil import *
import anvil.server


class UserProfile(UserProfileTemplate):
    def __init__(self, user_id, **properties):
        self.user_id = user_id
        self.user = None
        self.init_components(**properties)
        self.load_user()
```

Common rules:

- Accept `**properties` and pass them to `self.init_components(**properties)`.
- Define state needed by data bindings before calling `refresh_data_bindings()`.
- Keep UI event handlers small.
- Put business rules, authorization, persistence, and external API calls in server modules or server-side model methods.

If you need to read constructor arguments before component creation, assign them before `init_components()`. If you need to access components, do it after `init_components()`.

## Form Lifecycle

Important Form/component events:

- `show`: Raised when the component is shown on screen.
- `hide`: Raised when removed from screen.
- `refreshing_data_bindings`: Raised when `refresh_data_bindings()` starts.

Use `show` for work that requires the component to be rendered, such as `scroll_into_view()`, layout measurement, or DOM class changes that depend on actual nodes.

## Component Access

Named YAML components become attributes on the Form:

```python
self.btn_save.text = "Save"
self.lbl_status.visible = False
```

If a component name duplicates a Form method or attribute, it can shadow that member. Prefer unique, descriptive component names.

Useful runtime methods:

```python
self.add_component(component, **layout_properties)
self.get_components()
self.clear()
self.raise_event("x-custom-event", value=123)
self.raise_event_on_children("x-refresh")
component.remove_from_parent()
component.scroll_into_view(smooth=True, align="center")
```

`scroll_into_view()` supports `align="start"`, `"center"`, `"end"`, and `"nearest"`.

## Event Handlers

Typical handler:

```python
def btn_save_click(self, **event_args):
    sender = event_args["sender"]
    ...
```

Event handlers receive `sender` and `event_name`, plus event-specific values.

Button click includes:

```python
event_args["keys"]  # dict with shift, alt, ctrl, meta
```

Mouse events include coordinates and button data. TextBox has `change`, `pressed_enter`, `focus`, and `lost_focus`.

Use custom events with an `x-` prefix:

```python
self.raise_event("x-user-saved", user=self.user)
self.add_event_handler("x-user-saved", self.handle_user_saved)
```

For a component with only one handler, `set_event_handler()` replaces existing handlers. `add_event_handler()` appends and can register the same function more than once.

## Data Binding Mental Model

Anvil data bindings are generated Python functions. For a binding like:

```yaml
data_bindings:
- property: text
  code: self.item["name"]
  writeback: true
```

Anvil effectively creates:

```python
def update_val(self, _anvil_component):
    _anvil_component.text = self.item["name"]

def clear_val(self, _anvil_component):
    _anvil_component.text = None

def save_val(self, _anvil_component):
    self.item["name"] = _anvil_component.text
```

`refresh_data_bindings()` runs the update functions. If a `KeyError` occurs, Anvil clears the target property to `None`.

## `self.item`

Every Form has an `item` property. It is commonly used by:

- RepeatingPanel item templates.
- DataRowPanel templates.
- Forms that edit a dict, row, or model object.

Setting `self.item` refreshes data bindings:

```python
self.item = user_row
```

Mutating the current item does not automatically refresh:

```python
self.item["name"] = "Ada"
self.refresh_data_bindings()
```

For item template Forms, keep `self.item` as the row/dict/model being displayed and avoid copying values into duplicate local state unless the UI is intentionally editing a draft.

## Writeback

Writeback happens when a component raises its internal `x-anvil-write-back-<property>` event.

For TextBox:

- `text` writeback occurs before `lost_focus`.
- `text` writeback occurs before `pressed_enter`.
- If `type: number`, `text` returns a number or `None`, not a string.

For TextArea:

- `text` writeback occurs before `lost_focus`.

Writeback must assign to an lvalue:

```yaml
# Good
code: self.item["email"]
writeback: true

# Bad
code: self.item["first"] + " " + self.item["last"]
writeback: true
```

Use explicit event handlers for nontrivial save behavior:

```python
def btn_save_click(self, **event_args):
    self.lbl_status.text = "Saving..."
    self.lbl_status.visible = True
    anvil.server.call("save_user", self.item)
    self.lbl_status.text = "Saved"
```

## Refreshing Data Bindings

Call `self.refresh_data_bindings()` after:

- Updating form-level values used by data bindings.
- Mutating `self.item` in place.
- Receiving new data from a server call.
- Toggling state used by bindings such as `self.can_save`.

Do not call it in a tight loop for every field. Update state, then refresh once.

The `refreshing_data_bindings` event is useful for computing derived UI state immediately before bindings run:

```python
def form_refreshing_data_bindings(self, **event_args):
    self.can_save = bool(self.item.get("email"))
```

Avoid server calls from `refreshing_data_bindings`; that event can run often.

## Dynamic Components

Create and add components in Python:

```python
from anvil import Button, Label

button = Button(text="Archive", role="secondary-action")
button.add_event_handler("click", self.archive_click)
self.actions_panel.add_component(button)
```

Pass layout properties according to the parent:

```python
self.grid.add_component(
    Button(text="Left"),
    row="A",
    col_xs=0,
    width_xs=6,
)

self.flow.add_component(TextBox(), width=240, expand=True)

self.html_template.add_component(Label(text="Hello"), slot="default")
```

Components can have only one parent. Call `remove_from_parent()` before moving a component to a new container.

## RepeatingPanel

RepeatingPanel creates one instance of its `item_template` Form per item:

```python
self.repeating_panel_users.item_template = UserRow
self.repeating_panel_users.items = users
```

Each template instance receives:

```python
self.item    # the corresponding item
self.parent  # the RepeatingPanel
```

Use template Forms for row UI and parent events for list-level behavior:

```python
# In item template
def btn_delete_click(self, **event_args):
    self.parent.raise_event("x-delete-user", user=self.item)

# In parent form
self.repeating_panel_users.add_event_handler("x-delete-user", self.delete_user)
```

After changing the list, set `items` again or update the underlying iterable and refresh intentionally.

## DataGrid And DataRowPanel

DataGrid columns are dictionaries:

```python
self.data_grid_users.columns = [
    {"id": "name", "title": "Name", "data_key": "name", "expand": True},
    {"id": "email", "title": "Email", "data_key": "email", "width": 240},
]
```

DataRowPanel with `auto_display_data=True` reads values from `self.item` using the column's `data_key`, or `id` when no `data_key` is set.

If you manually add a component to a DataRowPanel column, it replaces the auto-generated value for that column:

```python
row.add_component(TextBox(text=user["name"]), column="name")
```

DataGrid pagination uses `rows_per_page`. Setting it to `0` or `None` disables pagination according to the component docs.

## DOM Nodes

HTML templates expose named DOM nodes:

```html
<div anvil-name="dialog_body"></div>
```

Python:

```python
self.dom_nodes["dialog_body"].classList.add("is-loading")
self.dom_nodes["dialog_body"].classList.remove("is-loading")
```

Use DOM class toggles when UI state is already known in Python, such as loading, expanded, selected, or modal-open state. Use CSS media queries for viewport-responsive state.

Do not rely on DOM selectors to infer business state when Python already knows it.

## Form Responsibility Pattern

Good Form responsibilities:

- Render UI and apply visual state.
- Collect user input.
- Wire component events.
- Call server functions.
- Trigger navigation.
- Show validation messages.
- Refresh data bindings.

Avoid in Forms:

- Direct database writes unless the app intentionally exposes client-writable views.
- Authorization decisions.
- Secret handling.
- Long-running jobs.
- Complex business workflows.
- External API calls with credentials.

Typical split:

```python
# client_code/UserEditor/__init__.py
def btn_save_click(self, **event_args):
    try:
        result = anvil.server.call("save_user_profile", self.item)
        self.item = result
        self.lbl_status.text = "Saved"
    except ValueError as e:
        self.lbl_status.text = str(e)
```

```python
# server_code/Users.py
@anvil.server.callable(require_user=True)
def save_user_profile(data):
    user = anvil.users.get_user()
    validate_profile(data)
    return update_profile_for_user(user, data)
```

