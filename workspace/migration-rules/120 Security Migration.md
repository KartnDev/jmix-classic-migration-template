# Security Migration Rules (Jmix 1 -> Jmix 2)

## Overview

Security in Jmix 2.x (Flow UI) has several important changes compared to Jmix 1.x (Classic UI). The core security model (ResourceRole, RowLevelRole) remains similar, but UI-specific annotations and permissions have changed.

## Dependencies

Ensure your `build.gradle` includes the correct security dependencies:

```groovy
// Jmix 1.x
implementation 'io.jmix.security:jmix-security-starter'
implementation 'io.jmix.securityui:jmix-securityui-starter'
implementation 'io.jmix.securitydata:jmix-securitydata-starter'
```

```groovy
// Jmix 2.x
implementation 'io.jmix.security:jmix-security-starter'
implementation 'io.jmix.security:jmix-security-flowui-starter'
implementation 'io.jmix.security:jmix-security-data-starter'
```

## Key Annotation Changes

### Screen/View Policy

```java
// Jmix 1.x (Classic UI)
import io.jmix.securityui.role.annotation.ScreenPolicy;
import io.jmix.securityui.role.annotation.MenuPolicy;

@ResourceRole(name = "Employee", code = "employee", scope = "UI")
public interface EmployeeRole {

    @ScreenPolicy(screenIds = {"Customer.browse", "Customer.edit"})
    @MenuPolicy(menuIds = {"Customer.browse"})
    void customers();
}
```

```java
// Jmix 2.x (Flow UI)
import io.jmix.securityflowui.role.annotation.ViewPolicy;
import io.jmix.securityflowui.role.annotation.MenuPolicy;

@ResourceRole(name = "Employee", code = "employee", scope = "UI")
public interface EmployeeRole {

    @ViewPolicy(viewIds = {"Customer.list", "Customer.detail"})
    @MenuPolicy(menuIds = {"Customer.list"})
    void customers();
}
```

**Key changes:**
- `@ScreenPolicy` → `@ViewPolicy`
- `screenIds` → `viewIds`
- Screen IDs change: `*.browse` → `*.list`, `*.edit` → `*.detail`
- `@MenuPolicy` stays but menu IDs follow view naming

### Wildcard Permissions

```java
// Allow all views and menu items
@ViewPolicy(viewIds = "*")
@MenuPolicy(menuIds = "*")
void allAccess();
```

## Specific Permissions

### Login Permission Change

```java
// Jmix 2.x
@SpecificPolicy(resources = "ui.loginToUi")
```

**Important:** In Jmix 1.x Classic UI there was no `loginToUi` permission. The permission `flowui.loginToUi` appeared in early Jmix 2.0 and was renamed to `ui.loginToUi`. Projects migrating from Jmix 1.x should use `ui.loginToUi` directly.

### GenericFilter Permissions

These permissions control what users can do with `genericFilter` component:

| Permission                                   | Description                              |
|----------------------------------------------|------------------------------------------|
| `ui.genericfilter.modifyConfiguration`       | Create/edit/delete filter configurations |
| `ui.genericfilter.modifyGlobalConfiguration` | Modify global (shared) configurations    |
| `ui.genericfilter.modifyJpqlCondition`       | Create/change JPQL conditions at runtime |

```java
// Example: Allow user to modify filter configurations
@SpecificPolicy(resources = {
    "ui.loginToUi",
    "ui.genericfilter.modifyConfiguration"
})
void requirements();
```

## Built-in Roles Migration

When migrating, copy these role classes from a new Jmix 2.x project:

### FullAccessRole

```java
// Copy from new Jmix 2.x project
@ResourceRole(name = "Full Access", code = FullAccessRole.CODE, scope = "UI")
public interface FullAccessRole {
    String CODE = "full-access";

    @ViewPolicy(viewIds = "*")
    @MenuPolicy(menuIds = "*")
    @SpecificPolicy(resources = "*")
    void requirements();
}
```

### UiMinimalRole

```java
// Copy from new Jmix 2.x project
@ResourceRole(name = "UI: minimal access", code = UiMinimalRole.CODE, scope = "UI")
public interface UiMinimalRole {
    String CODE = "ui-minimal";

    @SpecificPolicy(resources = "ui.loginToUi")
    void requirements();
}
```

**Important:** Users MUST have `UiMinimalRole` (or a role with `ui.loginToUi` permission) to log in to the UI.

## Migration Steps for Existing Roles

1. **Remove old annotations:**
   ```java
   // Remove these imports
   import io.jmix.securityui.role.annotation.MenuPolicy;
   import io.jmix.securityui.role.annotation.ScreenPolicy;
   ```

2. **Add new annotations:**
   ```java
   // Add these imports
   import io.jmix.securityflowui.role.annotation.MenuPolicy;
   import io.jmix.securityflowui.role.annotation.ViewPolicy;
   ```

3. **Update screen IDs to view IDs:**

   | Jmix 1.x Screen ID | Jmix 2.x View ID                 |
   |--------------------|----------------------------------|
   | `Entity.browse`    | `Entity.list`                    |
   | `Entity.edit`      | `Entity.detail`                  |
   | `Entity.lookup`    | `Entity.list` (with lookup mode) |

4. **Update specific permissions (if migrating from early Jmix 2.0):**
   - `flowui.loginToUi` → `ui.loginToUi`
   - For Jmix 1.x migrations: add `ui.loginToUi` permission (did not exist in Classic UI)

## Application Properties

```properties
# Jmix 2.0 (early naming)
jmix.flowui.login-view-id=LoginView
jmix.flowui.main-view-id=MainView

# Jmix 2.x (current, since ~2.1+)
jmix.ui.login-view-id=LoginView
jmix.ui.main-view-id=MainView
```

> **Note:** In Jmix 1.x Classic UI, these properties did not exist. The `jmix.flowui.*` prefix appeared in early Jmix 2.0 and was renamed to `jmix.ui.*`. Projects migrating from Jmix 1.x should use the `jmix.ui.*` prefix directly.

## Database Roles Migration

If you have roles stored in the database (runtime roles), you need to:

1. Update `RESOURCE_POLICY` records where `TYPE = 'screen'`:
   - Change `TYPE` to `'view'`
   - Update `RESOURCE` values (screen IDs → view IDs)

2. Update specific policy resources (if migrating from early Jmix 2.0):
   - `flowui.loginToUi` → `ui.loginToUi`
   - For Jmix 1.x: insert new `ui.loginToUi` permission records

**Example SQL:**
```sql
-- Update screen policies to view policies
UPDATE SEC_RESOURCE_POLICY
SET TYPE = 'view',
    RESOURCE = REPLACE(REPLACE(RESOURCE, '.browse', '.list'), '.edit', '.detail')
WHERE TYPE = 'screen';

-- Update specific permission
UPDATE SEC_RESOURCE_POLICY
SET RESOURCE = 'ui.loginToUi'
WHERE RESOURCE = 'flowui.loginToUi';
```

## Entity Policies (No Changes)

Entity policies remain the same:

```java
// Same in both Jmix 1.x and 2.x
@EntityPolicy(entityClass = Customer.class,
              actions = {EntityPolicyAction.READ, EntityPolicyAction.UPDATE})
@EntityAttributePolicy(entityClass = Customer.class,
                       attributes = "*",
                       action = EntityAttributePolicyAction.MODIFY)
void customers();
```

## Row-Level Roles (No Changes)

Row-level security works the same way:

```java
// Same in both Jmix 1.x and 2.x
@RowLevelRole(name = "See only own orders", code = "own-orders")
public interface OwnOrdersRole {

    @JpqlRowLevelPolicy(entityClass = Order.class,
                        where = "{E}.createdBy = :current_user_username")
    void ownOrders();
}
```

## UI Constraints Add-on (Optional)

For granular component-level control in Jmix 2.x, consider using the UI Constraints add-on:

```java
@ResourceRole(name = "Manager", code = "manager")
public interface ManagerRole {

    @UiComponentPolicy(
        viewClass = OrderDetailView.class,
        componentIds = {"approveBtn", "rejectBtn"},
        action = UiComponentPolicyAction.ENABLED,
        effect = UiComponentPolicyEffect.DENY
    )
    void denyApproval();
}
```

## Quick Reference

| Aspect            | Jmix 1.x                  | Jmix 2.x                       |
|-------------------|---------------------------|--------------------------------|
| Package           | `io.jmix.securityui.*`    | `io.jmix.securityflowui.*`     |
| Screen annotation | `@ScreenPolicy`           | `@ViewPolicy`                  |
| Login permission  | N/A (new in Jmix 2.x)     | `ui.loginToUi`                 |
| Properties prefix | N/A                       | `jmix.ui.*`                    |
| Dependency        | `jmix-securityui-starter` | `jmix-security-flowui-starter` |
| Entity policies   | No changes                | No changes                     |
| Row-level roles   | No changes                | No changes                     |

## Common Migration Mistakes

### 1. Forgetting ui.loginToUi permission
```java
// Wrong - user cannot log in!
@ResourceRole(name = "Employee", code = "employee")
public interface EmployeeRole {
    @ViewPolicy(viewIds = {"Customer.list"})
    void customers();
}

// Correct - include login permission
@ResourceRole(name = "Employee", code = "employee")
public interface EmployeeRole {
    @SpecificPolicy(resources = "ui.loginToUi")
    @ViewPolicy(viewIds = {"Customer.list"})
    @MenuPolicy(menuIds = {"Customer.list"})
    void requirements();
}
```

### 2. Using old screen IDs
```java
// Wrong
@ViewPolicy(viewIds = {"Customer.browse", "Customer.edit"})

// Correct
@ViewPolicy(viewIds = {"Customer.list", "Customer.detail"})
```

### 3. Missing menu policy
```java
// User can access view by URL but won't see it in menu!
@ViewPolicy(viewIds = {"Customer.list"})

// Correct - add menu policy
@ViewPolicy(viewIds = {"Customer.list"})
@MenuPolicy(menuIds = {"Customer.list"})
```
