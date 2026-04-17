# Anvil Reference Set

This directory is an app-neutral, portable reference set for AI agents and developers working on Anvil apps. It is designed to be copied directly into an Anvil app repository and used without companion materials or prior framework rediscovery.

Use these documents as a starting map, not as a substitute for the app in front of you. Existing app code, `anvil.yaml`, `theme/assets/theme.css`, and the actual form YAML are always the local source of truth.

## Reading Order

1. [App Structure And Runtime](01_app_structure_and_runtime.md)  
   File layout, `anvil.yaml`, app startup, local App Server behavior, dependencies, services, and local development.

2. [Form Template YAML](02_form_template_yaml.md)  
   The `form_template.yaml` schema, component trees, event/data bindings, layout properties, slots, and safe editing rules.

3. [Forms, Components, And Data Bindings](03_forms_components_and_data_bindings.md)  
   Form lifecycle, event handlers, `self.item`, `refresh_data_bindings()`, writeback, dynamic components, and DOM access.

4. [Layout, DOM, And CSS](04_layout_dom_css.md)  
   Runtime DOM wrappers, ColumnPanel, FlowPanel, GridPanel, common component quirks, theme roles, and reusable CSS patterns.

5. [Server, Data Tables, And Security](05_server_data_tables_and_security.md)  
   Server modules, callable functions, background tasks, Data Tables, row/model classes, transactions, and client trust boundaries.

6. [Custom Components, Dependencies, And Routing](06_custom_components_dependencies_and_routing.md)  
   Custom components, custom properties/events, HTML templates, slots, app dependencies, and optional routing patterns.

7. [AI Agent Playbook](07_ai_agent_playbook.md)  
   A task checklist for inspecting, changing, and verifying Anvil apps without rediscovering framework quirks.

8. [Scope And Maintenance Notes](08_scope_and_maintenance_notes.md)  
   Assumptions, version-drift risks, and what should be treated as app-specific rather than framework-wide.

## Optional Live References

These docs are intended to be usable offline. When internet access is available and a fact may have changed, check the official Anvil docs:

- [Forms as Components](https://anvil.works/docs/ui/components/forms)
- [Server Modules](https://anvil.works/docs/server/server-modules)
- [Using Data Tables from Python](https://anvil.works/docs/data-tables/data-tables-in-code)
- [Custom Components](https://anvil.works/docs/client/customisation/custom-components)
- [Component Libraries](https://anvil.works/docs/components)
- [URL Routing](https://anvil.works/docs/client/navigation/routing)

For runtime implementation details that are not covered by public docs, the open-source Anvil runtime is available at:

- [anvil-works/anvil-runtime](https://github.com/anvil-works/anvil-runtime)

## Design Goal

The goal is to give an AI agent enough framework context to:

- Add or modify Anvil Forms without corrupting YAML.
- Understand why generated DOM/CSS behaves differently from normal hand-written HTML.
- Keep business rules and security checks in trusted code.
- Use Data Tables and Model Classes safely.
- Separate reusable Anvil facts from one app's visual system or coding preferences.

## Update Policy

When Anvil behavior changes, update the relevant reference file and add a note in `08_scope_and_maintenance_notes.md`. Component DOM, CSS class prefixes, routing dependencies, and Data Tables features are the most likely areas to drift.
