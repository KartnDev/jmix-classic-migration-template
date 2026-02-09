# UI View Handlers Migration Rules (Jmix 1 -> Jmix 2)

## Lifecycle Event Mapping (Jmix 1.x -> Jmix 2.x)

When migrating from Jmix 1.x (Classic UI) to Jmix 2.x (Flow UI), screen lifecycle events have been renamed or refactored:

| **Jmix 1.x (Screen Controller)** | **Jmix 2.x (View Controller)** | **Notes**                                               |
|----------------------------------|--------------------------------|---------------------------------------------------------|
| `InitEvent`                      | `InitEvent`                    | Both fire after UI components are created and injected  |
| `AfterInitEvent`                 | *(Merged into BeforeShow)*     | Jmix 2.x triggers data loaders before showing           |
| `BeforeShowEvent`                | `BeforeShowEvent`              | Similar purpose                                         |
| `AfterShowEvent`                 | **`ReadyEvent`**               | Renamed in Jmix 2.x. Fired when the view is fully ready |
| `BeforeCloseEvent`               | `BeforeCloseEvent`             | No major change. Allows vetoing close                   |
| `AfterCloseEvent`                | `AfterCloseEvent`              | No major change                                         |
| `BeforeCommitChangesEvent`       | **`BeforeSaveEvent`**          | Fired before entity is saved                            |
| `AfterCommitChangesEvent`        | **`AfterSaveEvent`**           | Fired after entity is saved                             |

### Detail View Save Events

```java
// Jmix 1.x
@Subscribe
public void onBeforeCommitChanges(BeforeCommitChangesEvent event) {
    getEditedEntity().setStatus(OrderStatus.CONFIRMED);
}

// Jmix 2.x
@Subscribe
public void onBeforeSave(BeforeSaveEvent event) {
    getEditedEntity().setStatus(OrderStatus.CONFIRMED);
}
```

## Dependency Injection Changes

In Jmix 1.x screens, `@Autowired` was used for both UI components and beans. In Jmix 2.x, use `@ViewComponent` for UI components and `@Autowired` for backend beans.

```java
// Jmix 1.x screen controller
@Autowired
private TextField<String> nameField;

@Autowired
private DataManager dataManager;
```

```java
// Jmix 2.x view controller
@ViewComponent
private TypedTextField<String> nameField;

@Autowired
private DataManager dataManager;
```

## Event Subscription Examples

### Jmix 1.x

```java
@UiController("Customer.browse")
@UiDescriptor("customer-browse.xml")
public class CustomerBrowse extends StandardLookup<Customer> {

    @Subscribe
    protected void onInit(InitEvent event) {
        // Initialize UI
    }

    @Subscribe
    protected void onBeforeShow(BeforeShowEvent event) {
        // Prepare data before showing
    }

    @Subscribe
    protected void onAfterShow(AfterShowEvent event) {
        // Final setup after screen is shown
    }
}
```

### Jmix 2.x

```java
@Route(value = "customers", layout = MainView.class)
@ViewController("Customer.list")
@ViewDescriptor("customer-list-view.xml")
public class CustomerListView extends StandardListView<Customer> {

    @Subscribe
    public void onInit(InitEvent event) {
        // Initialize UI
    }

    @Subscribe
    public void onBeforeShow(BeforeShowEvent event) {
        // Prepare data before showing
    }

    @Subscribe
    public void onReady(ReadyEvent event) {
        // Final setup after view is ready (replaces AfterShowEvent)
    }
}
```

## Fragment Lifecycle Events

**Fragments** have a different lifecycle in Jmix 2.x compared to full views:

- **No separate BeforeShow for fragments**: Fragments don't get their own `BeforeShowEvent`. Use the host view's events instead.
- **`Fragment.ReadyEvent`**: Fired when the fragment and its components are fully initialized. Happens BEFORE host's ready event - can produce exceptions if logics depends on host
- **Subscribe to host events**: Use `@Subscribe(target = Target.HOST_CONTROLLER)` in the fragment controller.

### Jmix 1.x Fragment

```java
@UiController("AddressFragment")
@UiDescriptor("address-fragment.xml")
public class AddressFragment extends ScreenFragment {

    @Autowired
    private CollectionLoader<City> citiesDl;

    @Subscribe(target = Target.PARENT_CONTROLLER)
    protected void onHostBeforeShow(Screen.BeforeShowEvent event) {
        citiesDl.load();
    }
}
```

### Jmix 2.x Fragment

```java
@FragmentDescriptor("address-fragment.xml")
public class AddressFragment extends Fragment<FormLayout> {

    @ViewComponent
    private CollectionLoader<City> citiesDl;

    @Subscribe(target = Target.HOST_CONTROLLER)
    public void onHostBeforeShow(View.BeforeShowEvent event) {
        citiesDl.load();
    }
}
```

**Key change:** `Target.PARENT_CONTROLLER` -> `Target.HOST_CONTROLLER`

## Shared DataContext

Jmix 2.x uses a single DataContext for a view and all its fragments. Changes in fragment data containers are reflected in the host view and saved together.

In XML, use `provided="true"` to indicate that the host view supplies the data container:

```xml
<!-- Fragment XML -->
<data>
    <instance id="addressDc"
              class="com.company.entity.Address"
              provided="true"/>
</data>
```

## Selection Events

### Jmix 1.x

```java
@Subscribe("customersTable")
public void onCustomersTableSelection(Table.SelectionEvent<Customer> event) {
    refreshRelatedData();
}
```

### Jmix 2.x

```java
@Subscribe("customersDataGrid")
public void onCustomersDataGridSelection(
        SelectionEvent<DataGrid<Customer>, Customer> event) {
    refreshRelatedData();
}
```

## Value Change Events

Value change events have a different signature in Jmix 2.x:

### Jmix 1.x

```java
@Subscribe("fulfilledByField")
public void onFulfilledByFieldValueChange(HasValue.ValueChangeEvent<FulfillmentCenter> event) {
    // handle value change
}
```

### Jmix 2.x

```java
@Subscribe("fulfilledByField")
public void onFulfilledByFieldComponentValueChange(
        AbstractField.ComponentValueChangeEvent<EntityComboBox<FulfillmentCenter>, FulfillmentCenter> event) {
    // handle value change
}
```

**Key changes:**
- `HasValue.ValueChangeEvent<T>` → `AbstractField.ComponentValueChangeEvent<ComponentType, T>`
- Import `com.vaadin.flow.component.AbstractField`

**Recommended:** For typed components (textField, comboBox, etc.), prefer using `TypedValueChangeEvent` instead of `ComponentValueChangeEvent` to ensure proper type handling:

```java
@Subscribe("fulfilledByField")
public void onFulfilledByFieldTypedValueChange(
        SupportsTypedValue.TypedValueChangeEvent<EntityComboBox<FulfillmentCenter>, FulfillmentCenter> event) {
    FulfillmentCenter oldValue = event.getOldValue();
    // handle value change with correct types
}
```

## TreeDataGrid Custom Hierarchy Column

For TreeDataGrid hierarchy columns with custom renderers, you need to manually reconfigure columns:

```java
@ViewComponent
private TreeDataGrid<Asset> assetsDataGrid;

@Subscribe
public void onInit(InitEvent event) {
    // Remove the existing hierarchy column
    if (assetsDataGrid.getColumnByKey("name") != null) {
        assetsDataGrid.removeColumn(assetsDataGrid.getColumnByKey("name"));
    }

    // Add a new hierarchy column with custom content
    TreeDataGrid.Column<Asset> nameColumn = assetsDataGrid.addComponentHierarchyColumn(asset -> {
        Icon icon = ComponentUtils.parseIcon(
                Boolean.TRUE.equals(asset.getIsFolder()) ? "vaadin:folder" : "vaadin:file");
        icon.setColor(Boolean.TRUE.equals(asset.getIsFolder()) ? "#ddd009" : "#8383f7");

        Span name = new Span(asset.getName());
        HorizontalLayout row = new HorizontalLayout(icon, name);
        row.setAlignItems(FlexComponent.Alignment.CENTER);
        return row;
    })
    .setHeader("Name")
    .setAutoWidth(true);

    assetsDataGrid.setColumnPosition(nameColumn, 0);
}
```

## Quick Reference

| Jmix 1.x                       | Jmix 2.x                                        |
|--------------------------------|-------------------------------------------------|
| `@Autowired` (UI)              | `@ViewComponent`                                |
| `@Autowired` (beans)           | `@Autowired`                                    |
| `AfterShowEvent`               | `ReadyEvent`                                    |
| `BeforeCommitChangesEvent`     | `BeforeSaveEvent`                               |
| `AfterCommitChangesEvent`      | `AfterSaveEvent`                                |
| `Target.PARENT_CONTROLLER`     | `Target.HOST_CONTROLLER`                        |
| `Table.SelectionEvent`         | `SelectionEvent<DataGrid<T>, T>`                |
| `HasValue.ValueChangeEvent<T>` | `AbstractField.ComponentValueChangeEvent<C, T>` |
| `getSelected()`                | `getSelectedItems()`                            |
| `protected void onXxx()`       | `public void onXxx()`                           |