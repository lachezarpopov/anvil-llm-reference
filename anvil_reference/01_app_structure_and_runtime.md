# App Structure And Runtime

## What Anvil Is

Anvil apps are full-stack Python web apps:

- Client-side Python runs in the browser.
- Server modules run in trusted server-side Python.
- Forms define UI and are represented by Python code plus designer YAML.
- Built-in services such as Data Tables, Users, Email, and integrations are configured in `anvil.yaml`.
- The Anvil CLI can check out apps locally, validate YAML, and sync local edits with the Anvil Editor.
- The open-source Anvil App Server can serve an app from the local filesystem.

## Standard App Directory

A modern Anvil app stored as local files is a Python package. A minimal app usually looks like this:

```text
MyApp/
  __init__.py
  anvil.yaml
  client_code/
    Form1/
      __init__.py
      form_template.yaml
    Module1.py
  server_code/
    ServerModule1.py
  theme/
    parameters.yaml
    templates.yaml
    assets/
      standard-page.html
      theme.css
```

Important paths:

- `client_code/`: Browser-side Forms and Modules.
- `client_code/<FormName>/__init__.py`: The form's Python code, equivalent to Code View in the Editor.
- `client_code/<FormName>/form_template.yaml`: The visual designer's component tree and form metadata.
- `server_code/`: Trusted server-side Python modules.
- `theme/parameters.yaml`: Theme roles, color scheme, and theme parameters.
- `theme/templates.yaml`: Editor-facing template metadata.
- `theme/assets/`: CSS, HTML templates, images, icons, and static assets.
- `anvil.yaml`: App-level configuration.

Older apps may store forms as module files:

```text
client_code/
  Form1.py
  Form1.yaml
```

Package Forms are preferred for new work.

## `anvil.yaml` Responsibilities

`anvil.yaml` configures the app as a whole. Common keys include:

- `package_name`: Top-level Python package name for absolute imports.
- `name`: Human-readable app name.
- `runtime_options`: Client runtime settings. Modern apps use `version: 2` and `client_version: '3'`.
- `startup`: Startup Form or Module, for example `{type: form, module: Form1}`.
- `startup_form`: Older equivalent to `startup: {type: form, module: ...}`.
- `metadata`: Browser title, description, favicon/social logo.
- `allow_embedding`: Whether the app may be embedded in another page.
- `native_deps`: Raw HTML inserted into the page `<head>`, usually for external JS/CSS libraries.
- `dependencies`: Other Anvil apps used for code, Forms, custom components, or shared services.
- `services`: Enabled services and client/server configuration.
- `db_schema`: Data Table schema.
- `scheduled_tasks`: Background tasks that run on a schedule.

Example startup:

```yaml
startup:
  type: form
  module: Main
```

Example runtime options:

```yaml
runtime_options:
  version: 2
  client_version: '3'
  server_persist: true
```

`server_persist` enables persistent server modules where supported by the hosting/runtime environment. Do not rely on global state for correctness; use it as a performance optimization only.

## Services

Anvil services are listed under `services`. A Data Tables entry commonly looks like:

```yaml
services:
- source: /runtime/services/tables.yml
  client_config: {}
  server_config:
    auto_create_missing_columns: false
```

Keep `auto_create_missing_columns: false` unless an existing app intentionally relies on auto-created columns. Auto-creating columns can hide typos such as `Name` versus `name`.

Common service sources include:

- `/runtime/services/tables.yml`
- `/runtime/services/anvil/users.yml`
- `/runtime/services/anvil/email.yml`
- `/runtime/services/anvil/secrets.yml`
- `/runtime/services/google.yml`
- `/runtime/services/facebook.yml`
- `/runtime/services/anvil/microsoft.yml`
- `/runtime/services/stripe.yml`

Hosted Anvil exports may contain encrypted secrets or hosted-only config. The local App Server ignores values it cannot decrypt and expects local values through command-line or config-file options.

## Data Table Schema In `anvil.yaml`

`db_schema` contains table definitions. Each table entry includes:

- `name`: Display name.
- `python_name`: Attribute name under `app_tables`.
- `id`: Stable table id.
- `columns`: Map of column ids to column specs.
- `access`: Client/server permissions and table id.

Column types include:

- `string`
- `number`
- `bool`
- `date`
- `datetime`
- `simpleObject`
- `media`
- `liveObject`
- `liveObjectArray`

Linked row columns use:

```yaml
type: liveObject
backend: anvil.tables.Row
table_id: 1
```

Data Table access levels:

- `none`: Client cannot search/update/delete; rows returned from server can be read.
- `search`: Client can `search()` and `get()`, but cannot add/update/delete.
- `full`: Client can perform all table operations.

Prefer `client: none` by default and expose narrow server functions or restricted views when the client needs data.

## Local Development With Anvil CLI

The Anvil CLI is for editing Anvil apps in a local IDE while syncing with the Anvil Editor. It is different from the local App Server:

- Use the Anvil CLI when the goal is to check out a hosted Anvil app, validate local YAML files, or sync local file edits back to Anvil.
- Use the local App Server when the goal is to run an app locally from files.

The CLI is currently beta, so check the official CLI docs or `anvil help` when command behavior matters.

Common setup:

```bash
anvil configure
anvil login
```

`anvil configure` performs guided setup, including default server URL, preferred editor, verbose logging, and login. The CLI stores local configuration and authentication tokens on the user's machine; do not copy those config files into an app repo.

Checkout examples:

```bash
anvil checkout
anvil checkout APP_ID MyApp
anvil checkout https://anvil.works/git/APP_ID.git MyApp
```

`anvil checkout` can also accept an Editor URL or run interactively. Useful checkout options include:

- `--branch BRANCH`: Check out a specific branch.
- `--depth N`: Create a shallow clone.
- `--single-branch`: Clone only one branch.
- `--origin NAME`: Use a custom Git remote name.
- `--open`: Open the destination after checkout.
- `--url ANVIL_URL` and `--user USERNAME`: Specify installation/account when inference is ambiguous.

Sync examples:

```bash
cd MyApp
anvil watch
anvil watch --appid APP_ID
anvil watch --staged-only
```

`anvil watch` watches local files and syncs them with the selected Anvil app. Before running it, verify the target app, branch, account, and server URL. `--staged-only` is useful when you want to sync only files staged with Git.

Useful watch options:

- `--appid APP_ID`: Specify the app id directly.
- `--first`: Auto-select the first detected app id.
- `--staged-only`: Sync only staged files.
- `--auto`: Restart on branch changes and sync when behind.
- `--open`: Open the watched path in the preferred editor.
- `--url ANVIL_URL` and `--user USERNAME`: Specify installation/account.

Validation examples:

```bash
anvil validate anvil.yaml
anvil validate client_code/Form1/form_template.yaml
```

`anvil validate <file>` validates YAML based on the path, currently for `anvil.yaml` and `client_code/**/*.yaml`. Use it after editing app YAML or Form template YAML. Validation catches schema problems, but it does not replace running the app and exercising the edited UI.

Other useful commands:

- `anvil config`: Manage CLI configuration.
- `anvil version`: Show version information.
- `anvil update`: Update the CLI.
- `anvil logout`: Log out from Anvil.
- `anvil --json ...`: Emit NDJSON for scripts or agent tooling where supported.
- `anvil --verbose ...`: Show more detailed output.

## Local Development With App Server

Install and run:

```bash
pip install anvil-app-server
anvil-app-server --app MyApp
```

Default local URL:

```text
http://localhost:3030/
```

Local edits normally require only a browser refresh, not a server restart.

Useful App Server options:

- `--app DIRECTORY`: App directory. Defaults to the current directory.
- `--data-dir DIRECTORY`: Local data storage. Defaults to `.anvil-data`.
- `--port PORT`: HTTP port.
- `--origin URL`: Public origin for generated URLs and optional HTTPS behavior.
- `--config-file FILENAME`: YAML config file for server options.
- `--dep-id ID=PACKAGE`: Map a dependency app id to a local adjacent directory/package.
- `--secret NAME=VALUE`: Supply app secrets locally.
- `--encryption-key NAME=VALUE`: Supply secrets-service encryption keys.
- `--uplink-key KEY`: Allow privileged Uplink connections.
- `--client-uplink-key KEY`: Allow unprivileged Uplink connections.
- `--shell`: Start an interactive Python shell via Uplink.
- `--auto-migrate`: Apply Data Table schema migrations automatically. Treat as potentially destructive.
- `--ignore-invalid-schema`: Run despite schema mismatch. Expect possible runtime errors.

If multiple local apps use the built-in database, give each a separate `--data-dir`.

## Dependencies

App dependencies are declared with app ids:

```yaml
dependencies:
- app_id: ZBDT7UM6GVGR7W4D
  version:
    branch: master
```

For the local App Server, dependency apps should usually be checked out next to the main app directory, then mapped with `--dep-id` if needed:

```bash
anvil-app-server --app MainApp --dep-id ZBDT7UM6GVGR7W4D=SharedComponents
```

In YAML, custom Forms/components from dependencies can appear as `type: form:...`. The exact suffix depends on how the dependency is referenced, so copy the existing app's format rather than inventing one.

## Startup Patterns

Startup can be a Form:

```yaml
startup:
  type: form
  module: Home
```

Or a Module:

```yaml
startup:
  type: module
  module: Startup
```

A startup module is useful when the app must perform routing setup, authentication checks, feature detection, or other decisions before calling `open_form(...)`.

## Client And Server Trust Boundary

Client code is visible and modifiable by users. Treat it as untrusted:

- Do not put secrets in client code or theme assets.
- Do not trust client-side validation for business rules.
- Do not rely on disabled/hidden UI for authorization.
- Do security checks in server modules or server-side model methods.
- Use table permissions and restricted table views deliberately.

Forms can validate for user experience, but server code must validate for correctness.

## Common App Structure Choices

The framework does not require a strict directory pattern beyond Anvil's app structure. In larger apps, useful conventions include:

```text
client_code/
  pages/
  components/
  layouts/
  services/
  state/
  routes/
server_code/
  services/
  repositories/
  jobs/
  api/
```

Treat this as an architectural preference, not an Anvil requirement. When contributing to an existing app, follow its current module layout.
