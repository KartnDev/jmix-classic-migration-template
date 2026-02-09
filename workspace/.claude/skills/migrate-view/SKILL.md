---
name: migrate-view
description: Migrate a single Jmix 1.x Classic UI screen to a Jmix 2.x Flow UI view. Converts XML descriptors, controllers, annotations, and component mappings. Use when migrating a specific screen or set of screens.
argument-hint: "[screen-id-or-class-name]"
allowed-tools: Read, Grep, Glob, Edit, Write, mcp__jetbrains__get_file_text_by_path, mcp__jetbrains__find_files_by_name_keyword, mcp__jetbrains__find_files_by_glob, mcp__jetbrains__search_in_files_by_text, mcp__jetbrains__search_in_files_by_regex, mcp__jetbrains__create_new_file, mcp__jetbrains__replace_text_in_file, mcp__jetbrains__get_file_problems, mcp__jetbrains__reformat_file, mcp__jetbrains__open_file_in_editor
---

# Migrate View

You are a Jmix UI migration specialist. You convert a Jmix 1.x Classic UI screen into a Jmix 2.x Flow UI view.

## Before starting

Read these migration rules:
- `migration-rules/010 Common.md`
- `migration-rules/060 UI View Controllers.md`
- `migration-rules/070 UI Data Section.md`
- `migration-rules/080 UI Handlers.md`
- `migration-rules/090 UI Tables and Actions.md`
- `migration-rules/100 UI Dialogs and Notifications.md`
- `migration-rules/110 UI UX Rules.md`

Identify source and target projects from `PLAN.md` or ask the user.

## Input

`$ARGUMENTS` can be:
- A screen ID (e.g. `petclinic_Owner.browse`)
- A controller class name (e.g. `OwnerBrowse`)
- A package (e.g. `screen.owner`) to migrate all screens in that package

## Migration steps

### 1. Locate the source screen
Find both:
- The Java controller (annotated with `@UiController`)
- The XML descriptor (referenced in `@UiDescriptor`)

### 2. Migrate the controller

**Class-level changes:**

| Jmix 1.x | Jmix 2.x |
|---|---|
| `@UiController("Entity.browse")` | `@ViewController("Entity.list")` |
| `@UiController("Entity.edit")` | `@ViewController("Entity.detail")` |
| `@UiDescriptor("entity-browse.xml")` | `@ViewDescriptor("entity-list-view.xml")` |
| `@LookupComponent("entityTable")` | `@LookupComponent("entityDataGrid")` |
| `io.jmix.ui.navigation.Route` | `com.vaadin.flow.router.Route` |
| `@Route("path")` | `@Route(value = "path", layout = MainView.class)` |
| `extends StandardLookup<T>` | `extends StandardListView<T>` |
| `extends StandardEditor<T>` | `extends StandardDetailView<T>` |
| `extends Screen` | `extends StandardView` |
| `OwnerBrowse` | `OwnerListView` |
| `OwnerEdit` | `OwnerDetailView` |
| package `screen.*` | package `view.*` |

**Injection changes:**
- `@Inject` or `@Autowired` for UI components -> `@ViewComponent`
- `@Autowired` for Spring beans -> stays `@Autowired`
- `@Autowired MessageBundle` -> `@ViewComponent MessageBundle`

**Event handler changes:**
- All handlers must be `public` (not `protected`)
- `AfterShowEvent` -> `ReadyEvent`
- `BeforeCommitChangesEvent` -> `BeforeSaveEvent`
- `AfterCommitChangesEvent` -> `AfterSaveEvent`

**Add to list views:**
- `@DialogMode(width = "50em")` (or appropriate width)

### 3. Migrate the XML descriptor

**Root element:**
```xml
<!-- Jmix 1.x -->
<window xmlns="http://jmix.io/schema/ui/window"
        caption="msg://..." focusComponent="...">

<!-- Jmix 2.x -->
<view xmlns="http://jmix.io/schema/flowui/view"
      title="msg://..." focusComponent="...">
```

**Data section:** Copy as-is (identical format).

**Facets:**
- `<screenSettings id="settingsFacet" auto="true"/>` -> `<settings auto="true"/>`
- Add `<urlQueryParameters>` with `<genericFilter>` and `<pagination>` references

**Layout and components:**

| Jmix 1.x | Jmix 2.x |
|---|---|
| `<filter>` | `<genericFilter>` |
| `<table>` / `<groupTable>` | `<dataGrid>` |
| `<treeTable>` | `<treeDataGrid>` |
| Column `id="prop"` | Column `property="prop"` |
| `multiselect="true"` | `selectionMode="MULTI"` |
| `<buttonsPanel>` inside table | `<hbox classNames="buttons-panel">` outside dataGrid |
| `<simplePagination/>` inside table | `<simplePagination dataLoader="..."/>` outside, in buttonsPanel endSlot |
| `<groupBox caption="...">` | `<details summaryText="...">` |
| `<form columns="2">` | `<formLayout>` |
| `<lookupField>` | `<entityComboBox>` or `<comboBox>` |
| `<lookupPickerField>` | `<entityPicker>` |
| `<scrollBox>` | `<scroller>` |
| `<checkBox>` | `<checkbox>` |
| `<label value="...">` | `<span text="...">` |
| `caption="..."` | `label="..."` |
| `stylename="danger"` | `themeNames="error"` |

**Action types:**
- `type="create"` -> `type="list_create"`
- `type="edit"` -> `type="list_edit"`
- `type="view"` -> `type="list_read"`
- `type="remove"` -> `type="list_remove"`
- `type="excelExport"` -> `type="grdexp_excelExport"`

**File naming:**
- `owner-browse.xml` -> `owner-list-view.xml`
- `owner-edit.xml` -> `owner-detail-view.xml`

### 4. Migrate message bundle

If there are screen-specific messages:
- `ownerBrowse.caption` -> `ownerListView.title`
- `ownerEdit.caption` -> `ownerDetailView.title`

### 5. Create files in target project

Place files under `jmix2-target/` in `view/` package (not `screen/`).

### 6. Validate

- Run `mcp__jetbrains__get_file_problems` on created Java and XML files
- Fix compilation errors
- Add `// TODO: migration <description>` for unresolvable issues
- Components without direct equivalent (e.g. `relatedEntities`, `maskedField`, `groupTable` without add-on): leave TODO

### 7. Update PLAN.md

Mark the screen as done in PLAN.md.

## Important

- Never modify files in `jmix1-source/`
- Do not commit changes
- `<data>` section is the same in both versions — copy without modifications
- `groupTable` has no built-in equivalent — use `dataGrid` and note the loss of grouping, or suggest the Grouping Data Grid add-on (Jmix 2.7+)
- If a screen references fragments, ensure fragments are migrated first
