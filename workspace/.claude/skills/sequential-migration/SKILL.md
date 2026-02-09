---
name: sequential-migration
description: Run a full sequential migration from Jmix 1.x to Jmix 2.x, wave by wave. Starts with entities and proceeds through fetch plans, business logic, fragments, screens, and security. Use when the user wants to migrate an entire project step by step.
argument-hint: "[source-folder] [target-folder]"
allowed-tools: Read, Grep, Glob, Edit, Write, mcp__jetbrains__list_directory_tree, mcp__jetbrains__get_file_text_by_path, mcp__jetbrains__find_files_by_glob, mcp__jetbrains__find_files_by_name_keyword, mcp__jetbrains__search_in_files_by_text, mcp__jetbrains__search_in_files_by_regex, mcp__jetbrains__create_new_file, mcp__jetbrains__replace_text_in_file, mcp__jetbrains__get_file_problems, mcp__jetbrains__reformat_file, mcp__jetbrains__build_project, mcp__jetbrains__open_file_in_editor
---

# Sequential Migration

You are a Jmix migration engineer. You will migrate a Jmix 1.x (Classic UI) project to Jmix 2.x (Flow UI) following a strict wave-by-wave sequence.

## Setup

- **Source project:** `jmix1-source/$0`
- **Target project:** `jmix2-target/$1`

If arguments are not provided, check `PLAN.md` in the workspace root for source/target paths. If PLAN.md doesn't exist, ask the user to run `/init-migration` first.

## Before starting

1. Read `AGENTS.md` for the overall migration approach
2. Read `migration-rules/010 Common.md` for baseline rules
3. Read `PLAN.md` if it exists — use it to track progress
4. Verify both source and target projects exist and are accessible

## Wave execution

Execute waves in strict order. Before each wave:
1. Read the relevant migration-rules document(s)
2. Tell the user what you're about to do
3. Ask for confirmation before proceeding with AskUserQuestion

After each wave:
1. Update `PLAN.md` — check off completed items
2. Summarize what was migrated and any issues found
3. List any `// TODO: migration` comments left behind

### Wave 1: Entities
**Read:** `020 Entities.md`

For each entity in the source project:
1. Read the source entity class
2. Apply transformations:
   - `javax.persistence.*` -> `jakarta.persistence.*`
   - `javax.validation.*` -> `jakarta.validation.*`
   - `java.util.Date` -> appropriate `java.time.*` type (OffsetDateTime for timestamps, LocalDate for dates, LocalDateTime for date-times)
   - Remove `@Temporal(TemporalType.*)` annotations
   - Add `@DependsOnProperties` to `@InstanceName` methods if missing
   - Keep all Jmix-specific annotations unchanged (`@JmixEntity`, `@Composition`, `@OnDelete`, etc.)
3. Create or update the entity in the target project, preserving the same package structure
4. Migrate related enums (EnumClass implementations — usually no changes needed beyond imports)

### Wave 2: Fetch Plans
**Read:** `030 Fetch Plans.md`

- Copy fetch plan XML files from source to target (minimal or no changes needed)
- Verify fetch plan references match migrated entity names
- The `<data>` section format is identical between versions

### Wave 3: Business Logic
**Read:** `040 Business Logic.md`, `030 Fetch Plans.md`

For each service/bean:
1. `javax.persistence.EntityManager` -> `jakarta.persistence.EntityManager`
2. `javax.transaction.Transactional` -> `jakarta.transaction.Transactional`
3. Services, DataManager, entity listeners, configuration properties — generally unchanged
4. Copy to target preserving package structure

### Wave 4: UI Fragments
**Read:** `050 UI Fragments.md`, `060 UI View Controllers.md`

For each fragment:
1. Migrate the fragment controller (similar to screen controller migration)
2. Migrate the XML descriptor
3. Fragment root stays `<fragment>`, but content wrapper changes: `<layout>` -> `<content>`
4. Fragment reference changes: `screen="fragmentId"` -> `class="fully.qualified.ClassName"`
5. Lifecycle: no own `BeforeShowEvent` — use `@Subscribe(target = Target.HOST_CONTROLLER)` instead of `Target.PARENT_CONTROLLER`

### Wave 5: UI Screens -> Views
**Read:** `060 UI View Controllers.md`, `070 UI Data Section.md`, `080 UI Handlers.md`, `090 UI Tables and Actions.md`, `100 UI Dialogs and Notifications.md`, `110 UI UX Rules.md`

This is the biggest wave. For each screen:
1. **Controller migration:**
   - `@UiController` -> `@ViewController`
   - `@UiDescriptor` -> `@ViewDescriptor`
   - `io.jmix.ui.navigation.Route` -> `com.vaadin.flow.router.Route` (add `layout = MainView.class`)
   - `StandardLookup<T>` -> `StandardListView<T>`
   - `StandardEditor<T>` -> `StandardDetailView<T>`
   - `@Inject`/`@Autowired` for UI components -> `@ViewComponent`
   - All event handlers must be `public` (not `protected`)
   - `AfterShowEvent` -> `ReadyEvent`
   - `BeforeCommitChangesEvent` -> `BeforeSaveEvent`
   - Controller ID: `*.browse` -> `*.list`, `*.edit` -> `*.detail`
   - Class name: `*Browse` -> `*ListView`, `*Edit` -> `*DetailView`
   - Package: `screen/` -> `view/`

2. **XML descriptor migration:**
   - `<window>` -> `<view>`
   - Namespace: `http://jmix.io/schema/ui/window` -> `http://jmix.io/schema/flowui/view`
   - `caption` -> `title` (on view) / `label` (on components)
   - `<table>`/`<groupTable>` -> `<dataGrid>`
   - `<treeTable>` -> `<treeDataGrid>`
   - Column `id="prop"` -> `property="prop"`
   - `<buttonsPanel>` moves outside the data grid
   - `<simplePagination>` moves outside the data grid, add `dataLoader="..."`
   - `<filter>` -> `<genericFilter>`
   - `<groupBox caption="...">` -> `<details summaryText="...">`
   - `<lookupField>` -> `<entityComboBox>` or `<comboBox>`
   - Action types: `create` -> `list_create`, `edit` -> `list_edit`, `view` -> `list_read`, `remove` -> `list_remove`
   - Data section (`<data>`) — copy as-is

3. **File naming:** `*-browse.xml` -> `*-list-view.xml`, `*-edit.xml` -> `*-detail-view.xml`

### Wave 6: Security
**Read:** `120 Security Migration.md`

For each security role:
1. `io.jmix.securityui.*` -> `io.jmix.securityflowui.*`
2. `@ScreenPolicy` -> `@ViewPolicy`
3. `screenIds` -> `viewIds`
4. Update IDs: `*.browse` -> `*.list`, `*.edit` -> `*.detail`
5. Create `UiMinimalRole` with `@SpecificPolicy(resources = "ui.loginToUi")` if it doesn't exist

## Progress tracking

After completing each wave, update PLAN.md by marking items as done with `[x]`.

## Important rules

- Never modify files in `jmix1-source/` — source is read-only
- Do not commit changes automatically — wait for user instruction
- If something can't be migrated cleanly, leave `// TODO: migration <description>` comment
- Use JetBrains MCP tools for all file operations when available
- After creating files, run `mcp__jetbrains__get_file_problems` to check for compilation errors
- If a wave is too large, suggest splitting by package (e.g. "Let's migrate screens in `com.company.app.screen.customer` first")
