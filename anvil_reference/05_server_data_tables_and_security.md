# Server, Data Tables, And Security

## Server Modules

Server modules live under `server_code/` and run in trusted Python. They should own:

- Authorization and permission checks.
- Business validation.
- Data Table reads/writes.
- External API calls and secrets.
- Background tasks.
- PDF/email/integration workflows.

Expose functions to client code with `@anvil.server.callable`:

```python
import anvil.server


@anvil.server.callable
def get_profile(user_id):
    return load_profile_for_current_user(user_id)
```

Client call:

```python
profile = anvil.server.call("get_profile", self.user_id)
```

You can register with an explicit name:

```python
@anvil.server.callable("users.get_profile")
def get_profile(user_id):
    ...
```

`@anvil.server.callable(require_user=True)` requires a logged-in user. `require_user` can also be a function. Still perform object-level authorization inside the server function.

## Background Tasks

Use background tasks for work that should not block a client request:

```python
@anvil.server.background_task("jobs.rebuild_index")
def rebuild_index(project_id):
    ...
```

Launch from client or server:

```python
task = anvil.server.launch_background_task("jobs.rebuild_index", project_id)
task_id = task.get_id()
```

Retrieve/list:

```python
task = anvil.server.get_background_task(task_id)
tasks = anvil.server.list_background_tasks()
```

Scheduled tasks are configured in `anvil.yaml` and refer to background task names.

## Call Context And Session

Useful server globals:

```python
anvil.server.context
anvil.server.session
```

`anvil.server.context` describes the current execution environment, caller, and client. `remote_caller.is_trusted` tells whether the caller is trusted server-side code.

`anvil.server.session` is a per-session dictionary. Store small, non-secret session state only. Do not use it as durable storage.

## Portable Classes

Objects crossing client/server boundaries must be serializable. Custom classes can be registered:

```python
@anvil.server.portable_class
class PriceQuote:
    def __init__(self, total, currency):
        self.total = total
        self.currency = currency
```

You can provide a custom name:

```python
@anvil.server.portable_class("myapp.PriceQuote")
class PriceQuote:
    ...
```

Registered portable classes can be sent through `anvil.server.call`, returned from server functions, and nested inside serializable structures.

## Data Tables Basics

Import:

```python
from anvil.tables import app_tables
import anvil.tables as tables
import anvil.tables.query as q
```

Add:

```python
row = app_tables.people.add_row(name="Ada", age=36)
```

Get one:

```python
row = app_tables.people.get(email="ada@example.com")
```

Search:

```python
for row in app_tables.people.search(active=True):
    print(row["email"])
```

Query operators:

```python
app_tables.people.search(age=q.greater_than_or_equal_to(18))
app_tables.people.search(name=q.ilike("%ada%"))
app_tables.people.search(q.any_of(active=True, admin=True))
app_tables.people.search(tables.order_by("created", ascending=False))
```

Useful operators:

- `q.like(pattern)`
- `q.ilike(pattern)`
- `q.greater_than(value)`
- `q.less_than(value)`
- `q.greater_than_or_equal_to(value)`
- `q.less_than_or_equal_to(value)`
- `q.between(min, max, min_inclusive=True, max_inclusive=False)`
- `q.full_text_match(query, raw=False)`
- `q.any_of(...)`
- `q.all_of(...)`
- `q.none_of(...)`
- `q.fetch_only(...)`
- `q.only_cols(...)`
- `q.page_size(rows)`

Search results are lazy iterables, not lists. Use `list(search_result)` when you intentionally need all rows.

Slice search results:

```python
latest_ten = app_tables.people.search(
    tables.order_by("created", ascending=False)
)[:10]
```

## Rows

Rows behave like dict-like objects:

```python
row["email"] = "ada@example.com"
row.update(active=True)
row.delete()
row.keys()
row.items()
row.values()
row.get("missing", default_value)
```

Important runtime behavior:

- Row column keys are strings.
- Rows cannot have arbitrary local state through `row.some_attr = ...` unless using a model class with descriptor handling.
- Deleted rows raise row-deleted errors when accessed.
- Draft rows and rows with buffered changes cannot be serialized until saved or reset.
- Media columns may be uncached and fetched lazily.

`row.get_id()` exists for compatibility, but do not treat it as a stable public schema substitute for table ids and row ids unless the app already does.

## Table Views And Client Access

A table can expose restricted views:

```python
@anvil.server.callable(require_user=True)
def get_my_messages_view():
    user = anvil.users.get_user()
    return app_tables.messages.client_readable(owner=user)
```

View methods:

- `client_readable(*restrictions, **fixed_columns)`
- `client_writable(*restrictions, **fixed_columns)`
- `client_writable_cascade(*restrictions, **fixed_columns)`
- `restrict_columns(col_spec)`

Use writable views carefully. A writable view is still an API surface exposed to untrusted client code.

Default safe posture:

- `client: none` in `anvil.yaml`.
- Client calls server functions.
- Server returns only rows/data the user is authorized to see.

## Transactions

Transactions are server-side only. Decorator form:

```python
@anvil.server.callable
@anvil.tables.in_transaction
def transfer_points(from_user, to_user, amount):
    ...
```

Python applies decorators bottom-up, so the transaction wrapper is applied before the function is made callable.

`@anvil.tables.in_transaction` automatically retries conflicts several times before raising.

Context manager form:

```python
with anvil.tables.Transaction() as txn:
    row["balance"] -= amount
```

Context manager conflicts raise `tables.TransactionConflict`; retry explicitly if appropriate.

## Model Classes

Modern accelerated Data Tables support Model Classes by subclassing a table's Row class:

```python
from datetime import date
from anvil.tables import app_tables


class Person(app_tables.people.Row):
    @property
    def age(self):
        born = self["date_of_birth"]
        return (date.today() - born).days // 365

    def display_name(self):
        return self["name"].strip()
```

Once registered, Anvil returns `Person` instances from:

- `app_tables.people.get(...)`
- `app_tables.people.search(...)`
- Linked row columns pointing to `people`

Optional subclass features available in modern Data Tables:

```python
class Person(app_tables.people.Row, attrs=True, buffered=True):
    ...
```

- `attrs=True`: Adds `row.column_name` style access by mapping attributes to columns when possible.
- `buffered=True`: Changes are buffered by default until `save()` or `reset()`.
- `client_writable=True`: Allows client-side creates/updates/deletes through model-mediated server calls.
- `client_updatable=True`, `client_creatable=True`, `client_deletable=True`: Control client-side model write permissions more narrowly.

These flags are powerful. Use them only when the model enforces authorization and validation server-side.

Draft rows:

```python
person = app_tables.people.Row(name="Ada")
person.save()
```

Buffered changes:

```python
with person.buffer_changes():
    person["name"] = "Ada Lovelace"
    person["active"] = True
    person.save()
```

Or:

```python
person.buffer_changes(True)
person["name"] = "Ada"
person.save()   # persist buffered changes
person.reset()  # discard buffered changes
```

Server-only model methods:

```python
import anvil.server


class Person(app_tables.people.Row):
    @anvil.server.server_method
    def deactivate(self):
        self["active"] = False
```

`server_method` must be used as the top-level decorator on a method inside a portable class/model class.

Runtime restrictions:

- Model classes cannot customize `__init__`, `__new__`, `__serialize__`, `__deserialize__`, or `__new_deserialized__`.
- Model classes are portable by default.
- Misspelled subclass options such as `client_writeable` raise helpful errors.

## Recommended Data Pattern

For simple apps:

```python
@anvil.server.callable(require_user=True)
def list_projects():
    user = anvil.users.get_user()
    return list(app_tables.projects.search(owner=user))
```

For larger apps:

- Keep UI state in Forms.
- Put table-specific behavior in Model Classes.
- Put cross-table workflows in server service modules.
- Wrap multi-row updates in transactions.
- Return rows/models only when client table permissions and data exposure are intentional.

## Security Checklist

Before exposing any callable or table view:

- Who can call this function?
- What user/object authorization is checked?
- Can the client pass a different row/user id?
- Are all server-side assumptions validated?
- Are secrets only read server-side?
- Are Data Table permissions no broader than necessary?
- Could a returned row expose linked rows or columns unexpectedly?
- Does a background task run with enough context to recheck permissions?
- Is user-provided HTML sanitized or avoided?

Client code can improve UX, but it cannot enforce security.
