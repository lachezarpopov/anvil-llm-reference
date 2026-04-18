# Scope And Maintenance Notes

## Portability Goal

This reference set is meant to be copied into any Anvil app repository and used on its own. It should not require:

- A separate Anvil runtime checkout.
- Separate framework notes.
- Prior app-specific prompt documents.
- Knowledge of how this reference set was originally assembled.

When these docs mention verification, they mean verification against the target app itself, its loaded CSS/DOM, its `anvil.yaml`, and official Anvil documentation if internet access is available.

## Baseline Assumptions

These docs focus on modern Anvil apps stored as local files with:

- `anvil.yaml` at the app root.
- Client Forms under `client_code/`.
- Server modules under `server_code/`.
- Theme assets under `theme/assets/`.
- Package Forms using `__init__.py` plus `form_template.yaml`.
- Standard/classic Anvil components.
- Data Tables accessed via `from anvil.tables import app_tables`.

Older module-style Forms and layout-based Forms are covered where they affect editing decisions.

## App-Specific Things To Discover Locally

Always inspect the target app for:

- Navigation style: `open_form`, content-panel swapping, routing dependency, or custom router.
- Theme roles and naming conventions.
- CSS tokens and spacing scale.
- Whether the app uses classic components, Material 3, or custom components heavily.
- Data Table permissions and whether client code is allowed to access tables.
- Server module organization and callable naming style.
- Custom component reference format in `form_template.yaml`.
- Dependency app ids and local dependency mapping.

Do not impose another app's role names, colors, folder structure, or routing library on the target app.

## General Guidance Versus App Standards

General Anvil guidance:

- Forms are UI components and should stay thin.
- Server modules are trusted and should enforce business/security rules.
- `form_template.yaml` is the designer's component tree and should be edited conservatively.
- Data bindings are good for simple UI projection.
- Component roles compose through role arrays.
- Anvil's generated DOM wrappers matter for CSS.
- Client code is untrusted.

App standards:

- Every button must use a role.
- Specific role names such as `primary-button`.
- Specific design tokens, colors, icon sets, or typography scales.
- A particular `client_code/pages/components/services` folder structure.
- A particular routing helper or dependency.
- A preference for direct Data Table views versus server DTOs.

Follow app standards when present. Do not treat them as framework facts.

## Known Drift Areas

These areas can change across Anvil versions, themes, or hosting/runtime environments:

- Component DOM wrapper structure.
- CSS class prefixes such as `.flow-panel` versus `.anvil-flow-panel`.
- Default theme spacing values.
- Material 3 component APIs and DOM.
- Data Tables model-class features.
- Routing dependency APIs.
- Anvil CLI beta behavior, command names, and sync semantics.
- Local App Server command-line flags.
- Hosted-only service behavior.

When a behavior matters to a task, verify it in the target app by inspecting the files, running the app, or checking official docs.

## Specific Assumptions To Recheck

Spacing:

- These docs describe classic spacing values commonly seen in current Anvil apps, including `small` as 5px.
- If the app's loaded CSS differs, follow the app.

Model classes:

- These docs describe modern Data Tables model classes as subclasses of `app_tables.<table>.Row`.
- If an app has an older hand-rolled model layer, follow that app's model layer.

Routing:

- Routing is optional and dependency-specific.
- Inspect imports and `anvil.yaml` before adding routes.

DOM selectors:

- Role selectors are usually more stable than framework wrapper selectors.
- Wrapper selectors are still necessary for layout fixes, but should be checked in the browser.

## Official Docs

This folder is intended to be sufficient for common agent work. For up-to-date public API details, use official Anvil documentation when available:

- Forms: `https://anvil.works/docs/ui/components/forms`
- Server modules: `https://anvil.works/docs/server/server-modules`
- Data Tables in code: `https://anvil.works/docs/data-tables/data-tables-in-code`
- Custom components: `https://anvil.works/docs/client/customisation/custom-components`
- Component libraries: `https://anvil.works/docs/components`
- URL routing: `https://anvil.works/docs/client/navigation/routing`
- Anvil CLI: `https://anvil.works/docs/using-another-ide/quickstart`

## Runtime Source Code

When public docs do not cover a behavior and internet access is available, the open-source Anvil runtime can be used as a secondary reference:

- `https://github.com/anvil-works/anvil-runtime`

Treat the target app and official docs as the first sources of truth. Use the runtime source for implementation details, drift checks, and generated DOM/runtime behavior that needs confirmation.

## Updating This Reference Set

When maintaining these docs:

- Keep them self-contained.
- Do not add references to non-portable materials.
- Prefer target-app evidence for app-specific decisions.
- Prefer official Anvil docs for current public API behavior.
- Preserve the distinction between Anvil facts and app conventions.
- Add dated notes for behaviors likely to drift.
