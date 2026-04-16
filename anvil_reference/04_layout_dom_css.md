# Layout, DOM, And CSS

## Why Anvil CSS Feels Different

Classic Anvil components are not rendered as a single obvious HTML element. The runtime wraps components in layout containers, uses Bootstrap-derived classes in classic themes, applies default spacing classes, and sometimes writes inline styles.

Effective Anvil CSS usually means:

- Style by theme roles, not by broad component type.
- Target the real wrapper that controls layout.
- Reset Anvil spacing when you want full CSS control.
- Use `!important` only for inline styles or unavoidable framework rules.
- Inspect the current app/runtime DOM when a selector does not work.

## Runtime Version Prefixes

Anvil apps may use legacy or newer class prefixes. You may see either:

```css
.column-panel
.flow-panel
.grid-panel
.col-padding
.flow-panel-gutter
```

or:

```css
.anvil-column-panel
.anvil-flow-panel
.anvil-grid-panel
.anvil-col-padding
.anvil-flow-panel-gutter
```

Many structural classes remain unprefixed, such as `.anvil-panel-section` and `.anvil-panel-col`. Role classes are always of the form:

```css
.anvil-role-my-role
```

For reusable app CSS, prefer role selectors and test against the actual rendered DOM.

## Theme Roles

The `role` property can be:

```python
self.button.role = "primary-action"
self.button.role = ["primary-action", "danger"]
self.button.role = None
```

YAML:

```yaml
properties:
  role: [button, danger]
```

Runtime behavior:

- Each role becomes `anvil-role-<role>`.
- Role names are sanitized to letters, numbers, underscore, and hyphen.
- Multiple roles are applied to the same outer component element.

Composable roles are a good general pattern:

```css
.anvil-role-button > button {
  border-radius: 6px;
}

.anvil-role-danger > button {
  color: var(--danger-fg);
}

.anvil-role-loading > button {
  opacity: 0.6;
}
```

Avoid app-specific role catalogs in shared docs. Each app should define its own semantic roles in `theme/parameters.yaml` and CSS in `theme/assets/theme.css`.

## Default Spacing

Classic component spacing classes:

```css
.anvil-spacing-above-none   { margin-top: 0px; }
.anvil-spacing-above-small  { margin-top: 5px; }
.anvil-spacing-above-medium { margin-top: 20px; }
.anvil-spacing-above-large  { margin-top: 40px; }

.anvil-spacing-below-none   { margin-bottom: 0px; }
.anvil-spacing-below-small  { margin-bottom: 5px; }
.anvil-spacing-below-medium { margin-bottom: 20px; }
.anvil-spacing-below-large  { margin-bottom: 40px; }
```

Current classic Anvil styling commonly uses `small` as 5px. If an app's loaded CSS differs, follow the app's actual CSS.

For predictable CSS-controlled layouts, set component spacing to `none`:

```yaml
properties:
  spacing_above: none
  spacing_below: none
```

Then apply gaps on the relevant wrapper.

## ColumnPanel

ColumnPanel is the default classic layout container. Runtime DOM:

```text
.anvil-role-my-panel.column-panel
  .anvil-panel-section
    .anvil-panel-section-container
      .anvil-panel-section-gutter
        .anvil-panel-row
          .anvil-panel-col
            .col-padding.col-padding-none
              [child component]
```

Newer prefixed runtime variants use `.anvil-column-panel` and `.anvil-col-padding`, but the panel-section classes are still recognizable.

Important behavior:

- Each top-level row is in its own `.anvil-panel-section`.
- `.anvil-panel-section-container` gets responsive max widths.
- `full_width_row: true` removes the max width for that row.
- `.anvil-panel-section-gutter` uses negative margins to balance column padding.
- `.anvil-panel-row` is `display: flex`.
- `.anvil-panel-col` has `overflow-x: hidden` and `pointer-events: none`.
- Direct children inside `.anvil-panel-col` restore `pointer-events: auto`.

Column spacing:

| `col_spacing` | Inner padding |
| --- | --- |
| `none` | `0 1px`, plus margin compensation |
| `tiny` | `0 5px` |
| `small` | `0 10px` |
| `medium` | `0 15px` |
| `large` | `0 20px` |
| `huge` | `0 30px` |

Use `col_spacing: none` when CSS should own spacing:

```yaml
name: pnl_stack
type: ColumnPanel
properties:
  col_spacing: none
  role: [profile-stack, reset-spacing]
```

Adjacent-row gap pattern:

```css
.anvil-role-profile-stack > .anvil-panel-section + .anvil-panel-section {
  margin-top: var(--gap-section);
}
```

Overflow workaround:

```css
.anvil-role-shadow-escape .anvil-panel-col {
  overflow: visible;
}
```

Use this for shadows, popovers, menus, or anything that gets clipped by `.anvil-panel-col`.

## FlowPanel

FlowPanel is a horizontal wrapping flex container. Runtime DOM:

```text
.anvil-role-my-row.flow-panel.flow-spacing-none.vertical-align-middle
  .flow-panel-gutter
    .flow-panel-item
      [child component]
```

Important behavior:

- The outer role element is not the flex container.
- The actual flex container is `> .flow-panel-gutter`.
- Each child is wrapped in `> .flow-panel-gutter > .flow-panel-item`.
- `align` is set as inline `justify-content` on the gutter.
- `vertical_align` adds a class that sets `align-items`.
- Child wrappers default to `flex: 0 1 auto`.
- FlowPanels wrap by default.

Standard YAML for CSS-controlled gaps:

```yaml
name: pnl_actions
type: FlowPanel
properties:
  align: right
  gap: none
  vertical_align: middle
  role: action-row
```

CSS gap pattern:

```css
.anvil-role-action-row > .flow-panel-gutter {
  gap: var(--gap-element);
}
```

Stretch first child:

```css
.anvil-role-stretch-first > .flow-panel-gutter > .flow-panel-item:first-child {
  flex: 1;
  min-width: 0;
}
```

Non-wrapping row:

```css
.anvil-role-no-wrap > .flow-panel-gutter {
  flex-wrap: nowrap;
}
```

Hiding a child component may leave its `.flow-panel-item` wrapper in the layout. If browser support is acceptable, hide wrappers with `:has()`:

```css
.flow-panel-item:has(> .anvil-visible-false),
.flow-panel-item:has(> .visible-false) {
  display: none;
}
```

Check the actual visibility class in the app runtime.

## GridPanel

GridPanel is a Bootstrap-style 12-column grid. Children are positioned with row/column layout properties:

```python
self.grid.add_component(
    Button(text="Submit"),
    row="A",
    col_xs=0,
    width_xs=6,
)
```

Runtime DOM:

```text
.grid-panel
  .row style="margin-bottom: ..."
    .col-xs-6.col-sm-6.col-md-6.col-lg-6
      [child component]
```

Important behavior:

- Components added to the same row in code must be added left-to-right.
- Rows get inline `margin-bottom` from `row_spacing`, so CSS overrides may need `!important`.
- Bootstrap columns have default horizontal padding.

Compact grid pattern:

```css
.anvil-role-compact-grid {
  overflow: visible;
}

.anvil-role-compact-grid > .row {
  margin-left: calc(var(--gap-element) / -2);
  margin-right: calc(var(--gap-element) / -2);
  margin-bottom: var(--gap-element) !important;
}

.anvil-role-compact-grid > .row:last-child {
  margin-bottom: 0 !important;
}

.anvil-role-compact-grid > .row > [class*="col-"] {
  padding-left: calc(var(--gap-element) / 2);
  padding-right: calc(var(--gap-element) / 2);
  overflow: visible;
}
```

## Button

Runtime DOM:

```text
.anvil-role-my-button.anvil-button.anvil-inlinable
  button.btn.btn-default.to-disable style="max-width:100%; overflow:hidden; ..."
    i.anvil-component-icon.left...
    span.button-text
    i.anvil-component-icon.right...
```

Important behavior:

- The outer component receives role classes.
- The clickable element is the nested `<button>`.
- The button has inline `max-width:100%`, `text-overflow:ellipsis`, and `overflow:hidden`.
- Both left and right icon elements can exist. Target `.left` or `.right`, not only `.anvil-component-icon`.
- `align: full` makes the nested button width `100%` and removes `anvil-inlinable`.
- Disabled state is applied to the outer element and `.to-disable` child.

CSS:

```css
.anvil-role-primary-action > button.btn {
  border-radius: 6px;
  font-weight: 600;
}

.anvil-role-icon-action > button.btn .anvil-component-icon.left {
  margin: 0;
}
```

Icons from the `icon` property render as image elements, so SVG `stroke` or `fill` cannot be recolored with `color`. Use the correct SVG asset, CSS filters for simple color transforms, or inline HTML/SVG when true recoloring is required.

## TextBox And TextArea

TextBox DOM:

```text
input.form-control.to-disable.anvil-text-box.anvil-role-my-input
```

TextArea DOM:

```text
textarea.form-control.to-disable.anvil-text-area.anvil-role-my-area
```

Important behavior:

- TextBox `type` can be `text`, `number`, `email`, `tel`, or `url`.
- TextBox with `type: number` returns a number or `None`.
- `hide_text: true` changes the input type to password.
- TextArea `auto_expand` grows to content at runtime, but not in the designer.
- Both components raise `change`; TextBox also raises `pressed_enter`.
- Writeback occurs before `lost_focus`; TextBox also writes back before `pressed_enter`.

Inputs often need explicit margin resets when embedded in FlowPanels:

```css
input.anvil-role-inline-field,
textarea.anvil-role-inline-field {
  margin: 0 !important;
}
```

## Image

Image DOM:

```text
.anvil-image style="min-height:20px; overflow:hidden; ..."
  img style="max-width:100%; max-height:100%; ..."
```

Display modes:

| `display_mode` | Runtime behavior |
| --- | --- |
| `shrink_to_fit` | `width:100%; height:100%; object-fit:contain` |
| `zoom_to_fill` | `width:100%; height:100%; object-fit:cover` |
| `fill_width` | `width:100%; height:auto` |
| `original_size` | Browser natural size, constrained by max width/height |

`height` defaults to 200 and is ignored for self-sizing modes (`fill_width`, `original_size`).

If a role needs icon-sized images, override the nested image with care:

```css
.anvil-role-inline-icon img {
  width: 16px !important;
  height: auto !important;
}
```

## Link

Link extends ColumnPanel, so it can contain child components. Runtime DOM starts with an `<a>` that also behaves as a container:

```text
a.anvil-inlinable.anvil-container.column-panel
  i.anvil-component-icon.left...
  .link-text
  i.anvil-component-icon.right...
```

For underline/text styling, target `.link-text` when needed:

```css
.anvil-role-subtle-link .link-text {
  text-decoration: underline;
}
```

If `url` is set, the link opens that URL or Media object. If `url` is empty, it still raises `click`.

## RepeatingPanel

RepeatingPanel DOM:

```text
.repeating-panel
  .hide-while-paginating
    [item template instances]
```

It is mainly behavior-driven. Style the item template Form rather than the RepeatingPanel unless you are controlling list-level spacing.

## DataGrid And DataRowPanel

DataGrid DOM:

```text
.anvil-data-grid.anvil-paginator
  .data-grid-child-panel
  .data-grid-footer-panel
    .footer-slot
    .pagination-buttons
```

DataRowPanel DOM:

```text
.anvil-data-row-panel
  .data-row-col[data-grid-col-id="..."]
```

Column specs can set `width` in pixels and `expand` to control flex growth. DataGrid injects per-grid CSS rules for column widths using `data-grid-id` and `data-grid-col-id`.

## Timer

Timer is invisible. It has no visible DOM and raises `tick` repeatedly while on page. `interval` is seconds; `0` disables ticks.

## Utility Roles Worth Reusing Across Apps

These are app-neutral patterns. Names are suggestions; adapt to the app's role naming convention.

```css
.anvil-role-reset-spacing .anvil-component {
  margin: 0;
}

.anvil-role-shadow-escape .anvil-panel-col {
  overflow: visible;
}

.anvil-role-no-wrap > .flow-panel-gutter {
  flex-wrap: nowrap;
}

.anvil-role-stretch-first > .flow-panel-gutter > .flow-panel-item:first-child {
  flex: 1;
  min-width: 0;
}
```

Use semantic roles for product styling and utility roles for framework workarounds:

```yaml
properties:
  role: [search-row, stretch-first, no-wrap]
```

## When `!important` Is Justified

Prefer specificity or better selectors first. `!important` is reasonable when overriding:

- Inline styles emitted by the runtime, such as GridPanel row margin.
- Inline `height` or object sizing on image children.
- Input margins from framework/theme CSS with higher specificity.

Document why:

```css
/* Runtime writes row margin-bottom inline from GridPanel row_spacing. */
.anvil-role-compact-grid > .row {
  margin-bottom: var(--gap-element) !important;
}
```
