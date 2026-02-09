---
name: migrate-security
description: Migrate Jmix 1.x security roles to Jmix 2.x. Converts ScreenPolicy to ViewPolicy, updates screen IDs, adds ui.loginToUi permission. Use when migrating security roles and permissions.
argument-hint: "[role-class-name]"
allowed-tools: Read, Grep, Glob, Edit, Write, mcp__jetbrains__get_file_text_by_path, mcp__jetbrains__find_files_by_name_keyword, mcp__jetbrains__find_files_by_glob, mcp__jetbrains__search_in_files_by_text, mcp__jetbrains__create_new_file, mcp__jetbrains__replace_text_in_file, mcp__jetbrains__get_file_problems, mcp__jetbrains__reformat_file
---

# Migrate Security

You are a Jmix security migration specialist. You convert Jmix 1.x security roles to Jmix 2.x.

## Before starting

1. Read `migration-rules/010 Common.md` and `migration-rules/120 Security Migration.md`
2. Identify source and target projects from `PLAN.md` or ask the user

## Input

`$ARGUMENTS` can be:
- A role class name (e.g. `VeterinarianRole`)
- `all` to migrate all security roles
- Empty — migrate all roles

## Migration steps

### 1. Find all security roles in source

Search for classes annotated with:
- `@ResourceRole`
- `@RowLevelRole`

### 2. Migrate each ResourceRole

**Import changes:**
- `io.jmix.securityui.role.annotation.ScreenPolicy` -> `io.jmix.securityflowui.role.annotation.ViewPolicy`
- `io.jmix.securityui.role.annotation.MenuPolicy` -> `io.jmix.securityflowui.role.annotation.MenuPolicy` (same package in flowui)

**Annotation changes:**
- `@ScreenPolicy(screenIds = {...})` -> `@ViewPolicy(viewIds = {...})`
- `@ScreenPolicy(screenClasses = {...})` -> `@ViewPolicy(viewClasses = {...})`

**Update all screen IDs in `viewIds` and `menuIds`:**
- `*.browse` -> `*.list`
- `*.edit` -> `*.detail`
- `*.lookup` -> `*.list`

**Entity policies and row-level policies:** Unchanged — copy as-is.

### 3. Create UiMinimalRole

If no role with `@SpecificPolicy(resources = "ui.loginToUi")` exists, create one:

```java
package <base.package>.security;

import io.jmix.security.model.SecurityScope;
import io.jmix.security.role.annotation.ResourceRole;
import io.jmix.securityflowui.role.annotation.ViewPolicy;
import io.jmix.security.role.annotation.SpecificPolicy;

@ResourceRole(name = "UI: minimal access", code = UiMinimalRole.CODE, scope = SecurityScope.UI)
public interface UiMinimalRole {
    String CODE = "ui-minimal";

    @ViewPolicy(viewIds = "MainView")
    @SpecificPolicy(resources = "ui.loginToUi")
    void main();
}
```

### 4. Migrate RowLevelRoles

Row-level roles typically need only import changes:
- `javax.persistence.*` -> `jakarta.persistence.*` (if JPQL predicates reference entity classes)
- The `@RowLevelRole` annotation and predicate methods are unchanged

### 5. Check for database-stored roles

If the project uses database-stored roles, note that SQL migration is needed:
- Update `TYPE_` column: `'screen'` -> `'view'`
- Update `RESOURCE_` values: `*.browse` -> `*.list`, `*.edit` -> `*.detail`
- Add `// TODO: migration - SQL script needed for database-stored roles` comment

### 6. Validate

- Run `mcp__jetbrains__get_file_problems` on created files
- Verify all referenced view IDs match the actually migrated views
- Fix compilation errors

### 7. Update PLAN.md

Mark migrated roles as done.

## Important

- Never modify files in `jmix1-source/`
- Do not commit changes
- `ui.loginToUi` is mandatory in Jmix 2.x — without it users cannot log in
- Entity policies (`@EntityPolicy`, `@EntityAttributePolicy`) are unchanged
