# AGENTS.md - Migration from Jmix 1 to Jmix 2

This file is a guide for AI agents to migrate an application project from Jmix 1.x (Classic UI) to Jmix 2.x (Flow UI).

## Project Structure

```
workspace/
│
├── migration-rules/                                # Wave-specific migration rules
│    ├── 010 Common.md                              # System requirements, imports, basics
│    ├── 020 Entities.md                            # Entity classes migration
│    ├── 030 Fetch Plans.md                         # Fetch plans (minimal changes)
│    ├── 040 Business Logic.md                      # Services, DataManager, listeners
│    ├── 050 UI Fragments.md                        # Reusable UI fragments
│    ├── 060 UI View Controllers.md                 # Screen/View controllers
│    ├── 070 UI Data Section.md                     # Data containers in XML
│    ├── 080 UI Handlers.md                         # Lifecycle events, handlers
│    ├── 090 UI Tables and Actions.md               # DataGrid, actions, TreeDataGrid
│    ├── 100 UI Dialogs and Notifications.md        # Dialogs, notifications
│    ├── 110 UI UX Rules.md                         # Layout, styling, UX patterns
│    ├── 120 Security Migration.md                  # Security roles, policies, permissions
│
├── jmix1-source/                                   # Reference Jmix 1.x projects
│    ├── source-jmix1-project-1/                    # First Jmix1 source project  (can be module or app)
│    └── source-jmix1-project-2/                    # Second source project
│
├── jmix2-target/                                   # Reference Jmix 2.x projects
│    ├── target-jmix2x-project-1/                   # First target FlowUI project 
│    └── target-jmix2x-project-2/                   # Second target project
│
└── AGENTS.md                                       # This document
```

Never read any files outside of the root workspace folder where this AGENTS.md file is located.

## Key Migration Differences

| Component            | Jmix 1.x                         | Jmix 2.x                                        |
|----------------------|----------------------------------|-------------------------------------------------|
| **Persistence**      | `javax.persistence.*`            | `jakarta.persistence.*`                         |
| **Validation**       | `javax.validation.*`             | `jakarta.validation.*`                          |
| **UI Package**       | `io.jmix.ui.*`                   | `io.jmix.flowui.*`                              |
| **Security UI**      | `io.jmix.securityui.*`           | `io.jmix.securityflowui.*`                      |
| **Controller**       | `@UiController`, `@UiDescriptor` | `@ViewController`, `@ViewDescriptor`            |
| **Route**            | `io.jmix.ui.navigation.Route`    | `com.vaadin.flow.router.Route`                  |
| **Browse**           | `StandardLookup<T>`              | `StandardListView<T>`                           |
| **Edit**             | `StandardEditor<T>`              | `StandardDetailView<T>`                         |
| **Injection**        | `@Inject` for UI                 | `@ViewComponent` for UI, `@Autowired` for beans |
| **XML Root**         | `<window>`                       | `<view>`                                        |
| **Tables**           | `table`, `groupTable`            | `dataGrid`                                      |
| **Screen Policy**    | `@ScreenPolicy`                  | `@ViewPolicy`                                   |
| **Login Permission** | N/A                              | `ui.loginToUi`                                  |

## Migration Strategy

Migration proceeds incrementally in the following waves:

1. **Entities** - domain model classes (mainly `javax.*` -> `jakarta.*`)
2. **Fetch Plans** - data loading configurations (minimal changes)
3. **Business Logic** - services, beans, entity and transaction listeners
4. **Fragments** - reusable UI elements
5. **Screens** - UI layer (major changes)

Do one wave at a time. You will be explicitly asked to proceed with a particular wave or a specific part of the project.

If you cannot migrate something, keep it with the comment `// TODO: migration <description>`

Do not commit any changes automatically.

Read **010 Common.md** when starting any migration step.
When migrating entities, read **020 Entities.md**.
When migrating fetch plans, read **030 Fetch Plans.md**.
When migrating business logic, read **040 Business Logic.md** and **030 Fetch Plans.md**.
When migrating fragments and screens, read all documents from **050 UI Fragments.md** to **110 UI UX Rules.md**, and **030 Fetch Plans.md**.
When migrating screens with maps (GeoMap), also read **120 Maps Migration.md**.
When migrating security roles and permissions, read **130 Security Migration.md**.
When setting up build configuration or deployment, read **140 Build and Deployment.md**.

## Important: Always Start by Reading Guidelines

Before working on ANY migration task:
1. ALWAYS read **010 Common.md** first
2. Read the wave-specific file(s) for your task. E.g. 030 if works with fetch plan.
3. Verify you've read all required files before making changes

## Using Tools

Always use Jetbrains MCP server in the target project.
