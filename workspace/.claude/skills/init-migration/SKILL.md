---
name: init-migration
description: Initialize migration from Jmix 1.x Classic UI to Jmix 2.x Flow UI. Analyzes source project structure, identifies modules, checks for target project, and builds a migration plan. Use when starting a new migration or when the user says "init migration", "start migration", "analyze project for migration".
argument-hint: "[source-project-folder]"
allowed-tools: Read, Grep, Glob, mcp__jetbrains__list_directory_tree, mcp__jetbrains__get_file_text_by_path, mcp__jetbrains__find_files_by_glob, mcp__jetbrains__find_files_by_name_keyword, mcp__jetbrains__search_in_files_by_text, mcp__jetbrains__create_new_file
---

# Init Migration

You are a Jmix migration analyst. Your job is to analyze a Jmix 1.x (Classic UI) source project and produce a structured migration plan.

## Step 1: Read the migration guidelines

Read `AGENTS.md` in the workspace root to understand the migration approach.
Read `migration-rules/010 Common.md` for baseline requirements.

## Step 2: Discover source projects

Scan `jmix1-source/` directory. If `$ARGUMENTS` is provided, look specifically for the `jmix1-source/$ARGUMENTS` project.

For each source project found:
1. Identify the project name, base package, and Jmix version from `build.gradle`
2. Count and list all entity classes (`@JmixEntity` or `@Entity`)
3. Count and list all screens (classes extending `StandardLookup`, `StandardEditor`, `Screen`, or annotated with `@UiController`)
4. Count and list all services (`@Service`, `@Component` beans in service packages)
5. Count and list all security roles (`@ResourceRole`, `@RowLevelRole`)
6. Count and list all UI fragments
7. Identify fetch plans (XML or annotated)
8. Check for add-ons: maps, BPM, reports, charts, etc.

## Step 3: Check for multiple modules

If there are multiple source projects in `jmix1-source/`:
- List all of them with their statistics
- Ask the user which module to migrate first using AskUserQuestion
- Record the choice in the plan

## Step 4: Check for target project

For the selected source project, check if a corresponding target exists in `jmix2-target/`:
- Look for a project with matching or similar name
- If found, verify it's a valid Jmix 2.x project (check `build.gradle` for `io.jmix.flowui`)
- If NOT found:
  - Warn the user that a target Jmix 2.x project is required
  - Recommend: "Please generate a new Jmix 2.x project using Jmix Studio with the same base package (`<detected.base.package>`) and place it in `jmix2-target/<project-name>`"
  - Stop and wait for the user to create the target project

## Step 5: Generate PLAN.md

Create `PLAN.md` in the workspace root with the following structure:

```markdown
# Migration Plan: <project-name>

## Source Project
- **Location:** jmix1-source/<name>
- **Base package:** <package>
- **Jmix version:** <version>

## Target Project
- **Location:** jmix2-target/<name>
- **Base package:** <package>
- **Jmix version:** <version>

## Project Statistics
| Category | Count | Details |
|----------|-------|---------|
| Entities | N | list... |
| Screens | N | list... |
| Services | N | list... |
| Security Roles | N | list... |
| Fragments | N | list... |
| Fetch Plans | N | list... |

## Add-ons Detected
- [ ] Maps
- [ ] BPM
- [ ] Reports
- [ ] Charts
- [ ] Other: ...

## Migration Waves

### Wave 1: Entities
- [ ] <EntityName1> — <brief description>
- [ ] <EntityName2> — <brief description>
...

### Wave 2: Fetch Plans
- [ ] <fetch-plan files or definitions>
...

### Wave 3: Business Logic
- [ ] <ServiceName1>
- [ ] <ServiceName2>
...

### Wave 4: UI Fragments
- [ ] <FragmentName1>
...

### Wave 5: UI Screens -> Views
- [ ] <ScreenName1> (browse) -> <ViewName1> (list)
- [ ] <ScreenName2> (edit) -> <ViewName2> (detail)
...

### Wave 6: Security
- [ ] <RoleName1>
- [ ] <RoleName2>
- [ ] Add UiMinimalRole with ui.loginToUi

## Notes
- <any special considerations, detected add-ons, potential issues>
```

## Step 6: Present summary

After generating PLAN.md, present a brief summary to the user:
- Total items to migrate per wave
- Any detected risks or blockers (missing target project, unusual add-ons, etc.)
- Recommend starting with `/sequential-migration <source> <target>` or wave-specific skills

## Important

- Never modify any source code during init — this is analysis only
- If source project uses CUBA platform (not Jmix 1.x), note this as it requires additional migration steps
- Always use JetBrains MCP tools when available for file operations
