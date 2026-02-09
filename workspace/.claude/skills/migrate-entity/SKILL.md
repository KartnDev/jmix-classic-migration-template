---
name: migrate-entity
description: Migrate a single Jmix entity class from 1.x to 2.x. Handles javax->jakarta imports, Date->java.time conversion, and annotation updates. Use when migrating a specific entity or a small set of entities.
argument-hint: "[entity-class-name-or-path]"
allowed-tools: Read, Grep, Glob, Edit, Write, mcp__jetbrains__get_file_text_by_path, mcp__jetbrains__find_files_by_name_keyword, mcp__jetbrains__find_files_by_glob, mcp__jetbrains__search_in_files_by_text, mcp__jetbrains__create_new_file, mcp__jetbrains__replace_text_in_file, mcp__jetbrains__get_file_problems, mcp__jetbrains__reformat_file, mcp__jetbrains__open_file_in_editor
---

# Migrate Entity

You are a Jmix entity migration specialist. You migrate a single entity class from Jmix 1.x to Jmix 2.x.

## Before starting

1. Read `migration-rules/010 Common.md` and `migration-rules/020 Entities.md`
2. Identify source and target projects from `PLAN.md` or ask the user

## Input

`$ARGUMENTS` can be:
- An entity class name (e.g. `Owner`, `Pet`, `petclinic_Owner`)
- A relative path to the entity file in source project
- A package path (e.g. `entity.owner`) to migrate all entities in that package

## Migration steps

### 1. Locate the source entity
Search in `jmix1-source/` for the entity class. If multiple matches, ask the user to clarify.

### 2. Read and analyze
- Read the full source entity file
- Identify: superclass, fields, associations, enums, embedded types
- Note any non-standard patterns

### 3. Apply transformations

**Imports:**
- `javax.persistence.*` -> `jakarta.persistence.*`
- `javax.validation.*` -> `jakarta.validation.*`
- `javax.annotation.*` -> `jakarta.annotation.*`

**Date/Time types:**
- `java.util.Date` with `@Temporal(TemporalType.TIMESTAMP)` -> `java.time.OffsetDateTime`
- `java.util.Date` with `@Temporal(TemporalType.DATE)` -> `java.time.LocalDate`
- `java.util.Date` with `@Temporal(TemporalType.TIME)` -> `java.time.LocalTime`
- `java.util.Date` without `@Temporal` -> `java.time.OffsetDateTime` (default)
- Remove all `@Temporal` annotations after conversion

**Annotations:**
- If `@InstanceName` is on a method, ensure `@DependsOnProperties({"field1", "field2"})` is present
- All Jmix annotations stay unchanged: `@JmixEntity`, `@JmixGeneratedValue`, `@Composition`, `@OnDelete`, `@OnDeleteInverse`, `@PropertyDatatype`, etc.
- `@MetaProperty` stays unchanged
- `@NumberFormat`, `@CaseConversion` stay unchanged

**Unchanged elements:**
- `EnumClass<T>` implementations — only need import fixes
- `@Composition`, `@OnDelete` — no changes
- Fetch plan annotations — no changes
- `@JmixEntity`, `@Store`, `@DbView` — no changes

### 4. Create in target project
- Place entity in the same package structure under `jmix2-target/`
- Preserve the same file name
- Also migrate any closely related enums or embedded entities

### 5. Validate
- Run `mcp__jetbrains__get_file_problems` on the created file
- Fix any compilation errors
- If something can't be resolved, add `// TODO: migration <description>`

### 6. Update PLAN.md
If PLAN.md exists, mark this entity as done: `- [x] EntityName`

## Important
- Never modify files in `jmix1-source/`
- Do not commit changes
- If the entity extends a base class, migrate the base class first
