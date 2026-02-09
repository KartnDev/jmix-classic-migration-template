# UI View Controller Migration Rules (Jmix 1 -> Jmix 2)

## Overview

Screen controllers in Jmix 1.x (Classic UI) need significant changes when migrating to Jmix 2.x (Flow UI). The component model, annotations, and base classes are different.

## Basic controller transformation

### Classes and annotations

```java
// Jmix 1.x
@UiController("petclinic_Owner.browse")
@UiDescriptor("owner-browse.xml")
@LookupComponent("ownersTable")
@Route(value = "owners")
public class OwnerBrowse extends StandardLookup<Owner> {
    // ...
}
```

```java
// Jmix 2.x
@Route(value = "owners", layout = MainView.class)
@ViewController("petclinic_Owner.list")
@ViewDescriptor("owner-list-view.xml")
@LookupComponent("ownersDataGrid")
@DialogMode(width = "50em")
public class OwnerListView extends StandardListView<Owner> {
    // ...
}
```

### Edit -> Detail screens

```java
// Jmix 1.x
@UiController("petclinic_Owner.edit")
@UiDescriptor("owner-edit.xml")
@EditedEntityContainer("ownerDc")
@Route(value = "owners/edit", parentPrefix = "owners")
public class OwnerEdit extends StandardEditor<Owner> {
    // ...
}
```

```java
// Jmix 2.x
@Route(value = "owners/:id", layout = MainView.class)
@ViewController("petclinic_Owner.detail")
@ViewDescriptor("owner-detail-view.xml")
@EditedEntityContainer("ownerDc")
public class OwnerDetailView extends StandardDetailView<Owner> {
    // ...
}
```

### Naming conventions

| Jmix 1.x                      | Jmix 2.x                       |
|-------------------------------|--------------------------------|
| `@UiController`               | `@ViewController`              |
| `@UiDescriptor`               | `@ViewDescriptor`              |
| `io.jmix.ui.navigation.Route` | `com.vaadin.flow.router.Route` |
| `StandardLookup<T>`           | `StandardListView<T>`          |
| `StandardEditor<T>`           | `StandardDetailView<T>`        |
| `*Browse` class name          | `*ListView` class name         |
| `*Edit` class name            | `*DetailView` class name       |
| `EntityName.browse` ID        | `EntityName.list` ID           |
| `EntityName.edit` ID          | `EntityName.detail` ID         |

### @DialogMode annotation (Jmix 2.x only)

In Jmix 2.x, use `@DialogMode` to configure how the view appears when opened as a dialog:

```java
@Route(value = "owners", layout = MainView.class)
@ViewController("petclinic_Owner.list")
@ViewDescriptor("owner-list-view.xml")
@DialogMode(width = "64em", height = "AUTO", resizable = true)
public class OwnerListView extends StandardListView<Owner> {
    // ...
}
```

**@DialogMode parameters:**
- `width` - dialog width (e.g., "50em", "80%")
- `height` - dialog height (e.g., "AUTO", "600px")
- `resizable` - whether user can resize (default: false)
- `modal` - whether dialog is modal (default: true)
- `closeOnOutsideClick` - close when clicking outside (default: false)
- `closeOnEsc` - close on Escape key (default: true)

**⚠️ IMPORTANT:** `layout = MainView.class` (or `DefaultMainViewParent.class` for addons) is REQUIRED in `@Route` annotation for proper navigation!

### Route changes

```java
// Jmix 1.x - Route from io.jmix.ui.navigation
import io.jmix.ui.navigation.Route;

@Route(value = "owners")
@Route(value = "owners/edit", parentPrefix = "owners")
```

```java
// Jmix 2.x - Route from Vaadin Flow, requires layout
import com.vaadin.flow.router.Route;

@Route(value = "owners", layout = MainView.class)
@Route(value = "owners/:id", layout = MainView.class)  // :id parameter for detail
```

## Dependency injection in controllers

### Injection rules

```java
// Jmix 1.x - @Autowired for both UI components and Spring beans
public class OwnerBrowse extends StandardLookup<Owner> {
    @Autowired
    private GroupTable<Owner> ownersTable;

    @Autowired
    private DataManager dataManager;

    @Autowired
    private Notifications notifications;

    @Autowired
    private Filter filter;

    @Autowired
    protected Label<String> titleLabel;
}
```

```java
// Jmix 2.x
public class OwnerListView extends StandardListView<Owner> {
    @ViewComponent  // UI components from the descriptor
    private DataGrid<Owner> ownersDataGrid;

    @ViewComponent
    private GenericFilter genericFilter;

    @ViewComponent
    private H3 nameHeader;  // Note: Label → H3 for headers

    @Autowired      // Spring beans ONLY
    private DataManager dataManager;

    @Autowired
    private Notifications notifications;
}
```

### Annotation mapping

| Component type        | Jmix 1.x     | Jmix 2.x         |
|-----------------------|--------------|------------------|
| UI component from XML | `@Autowired` | `@ViewComponent` |
| Spring bean           | `@Autowired` | `@Autowired`     |
| Data container        | `@Autowired` | `@ViewComponent` |
| Data loader           | `@Autowired` | `@ViewComponent` |
| DataContext           | `@Autowired` | `@ViewComponent` |
| MessageBundle         | `@Autowired` | `@ViewComponent` |

### Component type changes

Some UI components have different types in Jmix 2.x:

| Jmix 1.x            | Jmix 2.x                                 |
|---------------------|------------------------------------------|
| `Label<String>`     | `H3`, `Span`, or `JmixLabel`             |
| `TextField<String>` | `TypedTextField<String>`                 |
| `GroupTable<T>`     | `DataGrid<T>`                            |
| `Table<T>`          | `DataGrid<T>`                            |
| `LookupField<T>`    | `EntityComboBox<T>` or `JmixComboBox<T>` |
| `Filter`            | `GenericFilter`                          |

### Method visibility

In Jmix 2.x, event handler methods should be `public` (not `protected`):

```java
// Jmix 1.x
@Subscribe
protected void onAfterShow(AfterShowEvent event) {
    // ...
}

// Jmix 2.x
@Subscribe
public void onReady(ReadyEvent event) {
    // ...
}
```

### View-scoped beans

| Bean             | Jmix 2.x Injection |
|------------------|--------------------|
| `DataContext`    | `@ViewComponent`   |
| `MessageBundle`  | `@MessageBundle`   |
| `Notifications`  | `@Autowired`       |
| `Dialogs`        | `@Autowired`       |
| `DataManager`    | `@Autowired`       |
| `UiComponents`   | `@Autowired`       |
| `ViewNavigators` | `@Autowired`       |

## Working with messages

```java
// Jmix 1.x
@Autowired
protected MessageBundle messageBundle;
```

```java
// Jmix 2.x
@ViewComponent
private MessageBundle messageBundle;
```

## Event subscription

```java
// Jmix 1.x
@Subscribe
protected void onAfterShow(AfterShowEvent event) {
    // ...
}

@Subscribe
protected void onBeforeShow(BeforeShowEvent event) {
    // ...
}
```

```java
// Jmix 2.x
@Subscribe
public void onReady(final ReadyEvent event) {
    // replaces AfterShowEvent
}

@Subscribe
public void onBeforeShow(final BeforeShowEvent event) {
    // same name, different package
}
```

### Event mapping

| Jmix 1.x Event     | Jmix 2.x Event     |
|--------------------|--------------------|
| `InitEvent`        | `InitEvent`        |
| `BeforeShowEvent`  | `BeforeShowEvent`  |
| `AfterShowEvent`   | `ReadyEvent`       |
| `BeforeCloseEvent` | `BeforeCloseEvent` |
| `AfterCloseEvent`  | `AfterCloseEvent`  |

## Background tasks

```java
// Jmix 1.x
@Autowired
private BackgroundWorker backgroundWorker;
```

```java
// Jmix 2.x
@Autowired
private BackgroundWorker backgroundWorker;

// Using dialogs for background tasks
@Autowired
private Dialogs dialogs;

BackgroundTask<Integer, Void> task = new EmailTask(selected);
dialogs.createBackgroundTaskDialog(task)
        .withHeader("Sending reminder emails")
        .withText("Please wait while emails are being sent")
        .withTotal(selected.size())
        .withShowProgressInPercentage(true)
        .withCancelAllowed(true)
        .open();
```

## Main menu

```xml
<!-- Jmix 1.x menu.xml -->
<menu-config xmlns="http://jmix.io/schema/ui/menu">
    <menu id="application">
        <item screen="petclinic_Owner.browse"/>
    </menu>
</menu-config>
```

```xml
<!-- Jmix 2.x menu.xml -->
<menu-config xmlns="http://jmix.io/schema/flowui/menu">
    <menu id="application" title="msg://io.jmix.petclinic/menu.application.title" opened="true">
        <item view="petclinic_Owner.list" title="msg://io.jmix.petclinic.view.owner/ownerListView.title"/>
    </menu>
</menu-config>
```

## File naming conventions

| Jmix 1.x               | Jmix 2.x                |
|------------------------|-------------------------|
| `OwnerBrowse.java`     | `OwnerListView.java`    |
| `OwnerEdit.java`       | `OwnerDetailView.java`  |
| `owner-browse.xml`     | `owner-list-view.xml`   |
| `owner-edit.xml`       | `owner-detail-view.xml` |
| `screen/owner/` folder | `view/owner/` folder    |
