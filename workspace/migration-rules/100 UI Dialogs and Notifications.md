# UI Dialogs & Notifications Migration Rules (Jmix 1 -> Jmix 2)

## Confirmation Dialog (Option Dialog)

### Jmix 1.x

```java
@Autowired
private Dialogs dialogs;

public void onRemoveItems() {
    dialogs.createOptionDialog()
            .withCaption("Confirm")
            .withMessage("Are you sure you want to delete selected items?")
            .withActions(
                    new DialogAction(DialogAction.Type.YES, Action.Status.PRIMARY)
                            .withHandler(e -> removeItems()),
                    new DialogAction(DialogAction.Type.NO)
            )
            .show();
}
```

### Jmix 2.x

```java
@Autowired
private Dialogs dialogs;

public void onRemoveItems() {
    dialogs.createOptionDialog()
            .withHeader("Confirm")  // withCaption -> withHeader
            .withText("Are you sure you want to delete selected items?")  // withMessage -> withText
            .withActions(
                    new DialogAction(DialogAction.Type.YES)
                            .withHandler(e -> removeItems()),
                    new DialogAction(DialogAction.Type.NO)
            )
            .open();  // show() -> open()
}
```

## Input Dialog

### Jmix 1.x

```java
@Autowired
private Dialogs dialogs;

dialogs.createInputDialog(this)
        .withCaption("Enter Values")
        .withParameters(
                InputParameter.stringParameter("name")
                        .withCaption("Name")
                        .withRequired(true),
                InputParameter.doubleParameter("amount")
                        .withCaption("Amount")
                        .withDefaultValue(100.0),
                InputParameter.entityParameter("customer", Customer.class)
                        .withCaption("Customer"),
                InputParameter.enumParameter("status", OrderStatus.class)
                        .withCaption("Status")
        )
        .withActions(DialogActions.OK_CANCEL)
        .withCloseListener(closeEvent -> {
            if (closeEvent.closedWith(DialogOutcome.OK)) {
                String name = closeEvent.getValue("name");
                Double amount = closeEvent.getValue("amount");
                Customer customer = closeEvent.getValue("customer");
                processValues(name, amount, customer);
            }
        })
        .show();
```

### Jmix 2.x

```java
@Autowired
private Dialogs dialogs;

dialogs.createInputDialog(this)
        .withHeader("Enter Values")  // withCaption -> withHeader
        .withParameters(
                InputParameter.stringParameter("name")
                        .withLabel("Name")  // withCaption -> withLabel
                        .withRequired(true),
                InputParameter.doubleParameter("amount")
                        .withLabel("Amount")
                        .withDefaultValue(100.0),
                InputParameter.entityParameter("customer", Customer.class)
                        .withLabel("Customer"),
                InputParameter.enumParameter("status", OrderStatus.class)
                        .withLabel("Status")
        )
        .withActions(DialogActions.OK_CANCEL)
        .withCloseListener(closeEvent -> {
            if (closeEvent.closedWith(DialogOutcome.OK)) {
                String name = closeEvent.getValue("name");
                Double amount = closeEvent.getValue("amount");
                Customer customer = closeEvent.getValue("customer");
                processValues(name, amount, customer);
            }
        })
        .open();  // show() -> open()
```

### Important: Dialogs from Fragments

```java
// Jmix 2.x - calling from fragment
@Subscribe("createBtn")
public void onCreateBtnClick(ClickEvent<JmixButton> event) {
    // Get parent View - MUST pass View, not fragment!
    View<?> parentView = UiComponentUtils.getView(this);

    dialogs.createInputDialog(parentView)
            .withHeader("New Item")
            .withParameters(/* ... */)
            .open();
}
```

## Lookup Dialog

### Jmix 1.x

```java
@Autowired
private ScreenBuilders screenBuilders;

screenBuilders.lookup(Customer.class, this)
        .withScreenClass(CustomerBrowse.class)
        .withSelectHandler(customers -> {
            for (Customer customer : customers) {
                addCustomerToOrder(customer);
            }
        })
        .build()
        .show();
```

### Jmix 2.x

```java
@Autowired
private DialogWindows dialogWindows;

dialogWindows.lookup(this, Customer.class)
        .withViewClass(CustomerListView.class)
        .withSelectHandler(customers -> {
            customers.forEach(this::addCustomerToOrder);
        })
        .withViewConfigurer(view -> {
            // Configure view before opening
            view.setExcludedIds(getExcludedIds());
        })
        .open();
```

## Detail/Editor Dialog

### Jmix 1.x

```java
@Autowired
private ScreenBuilders screenBuilders;

// Creating new entity
screenBuilders.editor(Customer.class, this)
        .newEntity()
        .withScreenClass(CustomerEdit.class)
        .withParentDataContext(dataContext)
        .withInitializer(customer -> {
            customer.setStatus(CustomerStatus.NEW);
            customer.setRegistrationDate(new Date());
        })
        .withAfterCloseListener(afterCloseEvent -> {
            if (afterCloseEvent.closedWith(StandardCloseAction.COMMIT)) {
                Customer saved = afterCloseEvent.getScreen().getEditedEntity();
                customersTable.setSelected(saved);
            }
        })
        .build()
        .show();

// Editing existing entity
screenBuilders.editor(customersTable)
        .withScreenClass(CustomerEdit.class)
        .build()
        .show();
```

### Jmix 2.x

```java
@Autowired
private DialogWindows dialogWindows;

// Creating new entity
dialogWindows.detail(this, Customer.class)
        .newEntity()
        .withViewClass(CustomerDetailView.class)
        .withParentDataContext(dataContext)
        .withInitializer(customer -> {
            customer.setStatus(CustomerStatus.NEW);
            customer.setRegistrationDate(LocalDate.now());  // Date -> LocalDate
        })
        .withAfterCloseListener(closeEvent -> {
            if (closeEvent.closedWith(StandardOutcome.SAVE)) {  // COMMIT -> SAVE
                Customer saved = closeEvent.getView().getEditedEntity();
                customersDataGrid.select(saved);  // setSelected -> select
            }
        })
        .open();

// Editing existing entity (with DataGrid)
dialogWindows.detail(customersDataGrid)
        .withViewClass(CustomerDetailView.class)
        .open();
```

## Notifications

### Jmix 1.x

```java
@Autowired
private Notifications notifications;

// Simple notification
notifications.create()
        .withCaption("Success")
        .withDescription("Customer saved successfully")
        .withType(Notifications.NotificationType.HUMANIZED)
        .show();

// Warning
notifications.create()
        .withCaption("Warning")
        .withDescription("Please check the input data")
        .withType(Notifications.NotificationType.WARNING)
        .show();

// Error
notifications.create()
        .withCaption("Error")
        .withDescription("Failed to save customer")
        .withType(Notifications.NotificationType.ERROR)
        .show();
```

### Jmix 2.x

```java
@Autowired
private Notifications notifications;

// Simple notification - short form
notifications.create("Customer saved successfully")
        .withType(Notifications.Type.SUCCESS)
        .show();

// Full form with title
notifications.create("Success", "Customer saved successfully")
        .withType(Notifications.Type.SUCCESS)
        .withPosition(Notification.Position.TOP_END)
        .withDuration(3000)
        .show();

// Warning
notifications.create("Warning", "Please check the input data")
        .withType(Notifications.Type.WARNING)
        .show();

// Error with HTML content (use Vaadin Html component)
notifications.show(new com.vaadin.flow.component.Html(
        "<span><strong>Failed to save</strong><br/>Check the log for details</span>"));
```

## Message Dialog

### Jmix 1.x

```java
@Autowired
private Dialogs dialogs;

dialogs.createMessageDialog()
        .withCaption("Information")
        .withMessage("Operation completed successfully")
        .withType(Dialogs.MessageType.CONFIRMATION)
        .show();
```

### Jmix 2.x

```java
@Autowired
private Dialogs dialogs;

dialogs.createMessageDialog()
        .withHeader("Information")
        .withText("Operation completed successfully")
        .withModal(true)
        .withCloseOnOutsideClick(false)
        .open();
```

## Quick Reference

| Jmix 1.x                                 | Jmix 2.x                                       |
|------------------------------------------|------------------------------------------------|
| `@Autowired`                             | `@Autowired`                                   |
| `.withCaption()`                         | `.withHeader()`                                |
| `.withMessage()`                         | `.withText()`                                  |
| `.withCaption()` (parameters)            | `.withLabel()`                                 |
| `.show()`                                | `.open()`                                      |
| `StandardCloseAction.COMMIT`             | `StandardOutcome.SAVE`                         |
| `ScreenBuilders`                         | `DialogWindows`                                |
| `.withScreenClass()`                     | `.withViewClass()`                             |
| `getScreen()`                            | `getView()`                                    |
| `Date`                                   | `LocalDate` / `LocalDateTime`                  |
| `createBackgroundWorkDialog(this, task)` | `createBackgroundTaskDialog(task)`             |
| `io.jmix.ui.executor.BackgroundTask`     | `io.jmix.flowui.backgroundtask.BackgroundTask` |

## Background Task Dialog

Background task dialogs have different API in Jmix 2.x:

### Jmix 1.x

```java
@Autowired
private Dialogs dialogs;

BackgroundTask<Integer, Result> task = new MyBackgroundTask();
dialogs.createBackgroundWorkDialog(this, task)  // Note: requires 'this'
        .withCaption("Processing")
        .withMessage("Please wait...")
        .withCancelAllowed(true)
        .withTotal(items.size())
        .withShowProgressInPercentage(true)
        .show();
```

### Jmix 2.x

```java
@Autowired
private Dialogs dialogs;

BackgroundTask<Integer, Result> task = new MyBackgroundTask();
dialogs.createBackgroundTaskDialog(task)  // Note: NO 'this' parameter!
        .withHeader("Processing")         // withCaption -> withHeader
        .withText("Please wait...")       // withMessage -> withText
        .withCancelAllowed(true)
        .withTotal(items.size())
        .withShowProgressInPercentage(true)
        .open();                          // show() -> open()
```

**Key changes:**
- `createBackgroundWorkDialog(this, task)` → `createBackgroundTaskDialog(task)` - removed `this` parameter
- Import: `io.jmix.ui.executor.BackgroundTask` → `io.jmix.flowui.backgroundtask.BackgroundTask`
- Import: `io.jmix.ui.executor.TaskLifeCycle` → `io.jmix.flowui.backgroundtask.TaskLifeCycle`

## Common Migration Mistakes

### 1. Wrong Owner for Dialogs from Fragments

```java
// Wrong - from fragment
dialogs.createInputDialog(this)  // this is fragment, not View!
        .open();

// Correct
View<?> view = UiComponentUtils.getView(this);
dialogs.createInputDialog(view)
        .open();
```

### 2. Incorrect Close Handling

```java
// Wrong
if (closeEvent.getCloseAction() == COMMIT_ACTION_ID)

// Correct
if (closeEvent.closedWith(StandardOutcome.SAVE))
```