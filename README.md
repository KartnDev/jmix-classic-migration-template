# Jmix AI-Assisted Migration (Jmix 1 -> Jmix 2)

This repository is designed to facilitate migration of application projects from Jmix 1.x (Classic UI) to Jmix 2.x (Flow UI) with the help of AI agents.

Below we assume you are using Claude Code, but other agents should work as well.

## Key Differences: Jmix 1 vs Jmix 2

| Aspect             | Jmix 1.x                          | Jmix 2.x                            |
|--------------------|-----------------------------------|-------------------------------------|
| **Vaadin**         | Vaadin 8 (Classic UI)             | Vaadin 24 (Flow UI)                 |
| **Persistence**    | `javax.persistence.*`             | `jakarta.persistence.*`             |
| **Validation**     | `javax.validation.*`              | `jakarta.validation.*`              |
| **UI Package**     | `io.jmix.ui.*`                    | `io.jmix.flowui.*`                  |
| **Screen Package** | `screen` folder                   | `view` folder                       |
| **Browse Screen**  | `StandardLookup`, `*Browse`       | `StandardListView`, `*ListView`     |
| **Edit Screen**    | `StandardEditor`, `*Edit`         | `StandardDetailView`, `*DetailView` |
| **Tables**         | `table`, `groupTable`             | `dataGrid`                          |
| **Filter**         | `filter`                          | `genericFilter`                     |
| **XML Root**       | `<window>`                        | `<view>`                            |
| **XML Schema**     | `http://jmix.io/schema/ui/window` | `http://jmix.io/schema/flowui/view` |

## Migration process

### Setup

- Create `workspace/jmix1-source` folder and copy or clone the source Jmix 1.x project into it, for example `jmix1-source/myapp`.
- Create project `workspace/jmix1-source/myapp-jmix2` using Jmix Studio 2.x, with the same base package as in source project.
- Init git repo and commit all.
- Setup IntelliJ IDEA MCP Server
	- Settings -> Tools -> MCP Server
		- Enable
		- Claude Code -> Auto-Configure
- Open both `myapp` and `myapp-jmix2` projects in IntelliJ to be accessible by MCP server.
- Open terminal in `workspace` and run Claude Code.
- `/mcp`, check that `jetbrains` is connected.

### Sequence

We recommend to proceed with migration in the following sequence: entities, fetch plans, business logic, fragments, screens. If the project is large, split each phase to the smaller steps, e.g. by packages.

### Example prompts

After starting the agent and before giving any migration commands, initialize it by the following prompt (replace `myapp` with your app folder name):

```
Your task is to migrate a project from Jmix 1.x to Jmix 2.x.
The source project is located in the `source-projects/myapp`. The target Jmix project is created in `target-projects/myapp-jmix2`.
Read `AGENTS.md` to understand the project structure, migration approach and rules.
```

**Migration prompts:**

```
Migrate all entities.
```

```
Migrate all shared fetch plans.
```

```
Migrate all business logic.
```

```
Migrate fragments if any.
```

```
Migrate all screens in `com.company.myapp.screen.sample` package.
```

```
Migrate all screens.
```
