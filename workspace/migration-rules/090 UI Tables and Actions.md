# UI Tables & Actions Migration Rules (Jmix 1 -> Jmix 2)

## Important Warnings

**⚠️ GroupTable migration:**
- `groupTable` → `dataGrid` (default, no grouping out of the box)
- For grouping support: install the [Grouping Data Grid add-on](https://www.jmix.io/marketplace/group-data-grid/) (available since Jmix 2.7) and use `groupDataGrid`

**⚠️ relatedEntities component does NOT exist in Jmix 2.x!**
- Implement similar functionality programmatically using `DialogWindows`

**⚠️ Table multi-select behavior changed:**
- Jmix 1.x: `multiselect="true"` attribute
- Jmix 2.x: `selectionMode="MULTI"` attribute

## 1. Table/DataGrid Component

### Jmix 1.x: Table and GroupTable (Vaadin 8)

```xml
<groupTable id="ownersTable"
            multiselect="true"
            width="100%"
            dataContainer="ownersDc">
    <actions>
        <action id="create" type="create"/>
        <action id="view" type="view"/>
        <action id="remove" type="remove"/>
        <action id="excelExport" type="excelExport"/>
    </actions>
    <columns>
        <column id="firstName"/>
        <column id="lastName"/>
        <column id="email"/>
    </columns>
    <simplePagination/>
    <buttonsPanel id="buttonsPanel" alwaysVisible="true">
        <button id="createBtn" action="ownersTable.create"/>
        <button id="viewBtn" action="ownersTable.view"/>
        <button id="removeBtn" action="ownersTable.remove" stylename="danger"/>
        <button id="excelExportBtn" action="ownersTable.excelExport"/>
    </buttonsPanel>
</groupTable>
```

### Jmix 2.x: DataGrid (Vaadin 24)

```xml
<hbox id="buttonsPanel" classNames="buttons-panel">
    <startSlot>
        <button id="createBtn" action="ownersDataGrid.create"/>
        <button id="readBtn" action="ownersDataGrid.read"/>
        <button id="removeBtn" action="ownersDataGrid.remove"/>
        <button id="excelExportBtn" action="ownersDataGrid.excelExport"/>
    </startSlot>
    <endSlot>
        <simplePagination id="pagination" dataLoader="ownersDl"/>
    </endSlot>
</hbox>
<dataGrid id="ownersDataGrid"
          width="100%"
          minHeight="20em"
          dataContainer="ownersDc">
    <actions>
        <action id="create" type="list_create"/>
        <action id="read" type="list_read"/>
        <action id="remove" type="list_remove"/>
        <action id="excelExport" type="grdexp_excelExport"/>
    </actions>
    <columns>
        <column property="firstName"/>
        <column property="lastName"/>
        <column property="email"/>
    </columns>
</dataGrid>
```

**Key changes:**
- `groupTable`/`table` -> `dataGrid`
- `<buttonsPanel>` inside table -> `<hbox classNames="buttons-panel">` outside
- `<simplePagination/>` inside table -> `<simplePagination dataLoader="..."/>` outside
- `column id="..."` -> `column property="..."`
- Action types renamed (see below)
- Add `minHeight="20em"` to dataGrid for proper display
- Use `<startSlot>` and `<endSlot>` inside hbox for button alignment
- `relatedEntities` component is NOT available in Jmix 2.x

### startSlot / endSlot Pattern

In Jmix 2.x, `hbox` can have `<startSlot>` and `<endSlot>` children for alignment:

```xml
<hbox id="buttonsPanel" classNames="buttons-panel">
    <startSlot>
        <!-- Left-aligned buttons -->
        <button action="dataGrid.create"/>
        <button action="dataGrid.edit"/>
    </startSlot>
    <endSlot>
        <!-- Right-aligned components -->
        <simplePagination dataLoader="dl"/>
    </endSlot>
</hbox>
```

## 2. Action Type Mapping

| Jmix 1.x             | Jmix 2.x                    |
|----------------------|-----------------------------|
| `type="create"`      | `type="list_create"`        |
| `type="edit"`        | `type="list_edit"`          |
| `type="view"`        | `type="list_read"`          |
| `type="remove"`      | `type="list_remove"`        |
| `type="excelExport"` | `type="grdexp_excelExport"` |

## 3. Standard View Actions

### List View (Browse) Actions

```xml
<!-- Jmix 1.x -->
<actions>
    <action id="lookupSelectAction"
            caption="msg:///actions.Select"
            icon="LOOKUP_OK"
            primary="true"
            shortcut="${COMMIT_SHORTCUT}"/>
    <action id="lookupCancelAction"
            caption="msg:///actions.Cancel"
            icon="LOOKUP_CANCEL"/>
</actions>
<!-- ... -->
<hbox id="lookupActions" spacing="true" visible="false">
    <button action="lookupSelectAction"/>
    <button action="lookupCancelAction"/>
</hbox>
```

```xml
<!-- Jmix 2.x -->
<actions>
    <action id="selectAction" type="lookup_select"/>
    <action id="discardAction" type="lookup_discard"/>
</actions>
<!-- ... -->
<hbox id="lookupActions" visible="false">
    <button id="selectBtn" action="selectAction"/>
    <button id="discardBtn" action="discardAction"/>
</hbox>
```

### Detail View (Edit) Actions

```xml
<!-- Jmix 1.x -->
<actions>
    <action id="windowCommitAndClose" caption="msg:///actions.Ok"
            icon="EDITOR_OK"
            primary="true"
            shortcut="${COMMIT_SHORTCUT}"/>
    <action id="windowClose"
            caption="msg:///actions.Close"
            icon="EDITOR_CANCEL"/>
</actions>
<!-- ... -->
<hbox id="editActions" spacing="true">
    <button id="commitAndCloseBtn" action="windowCommitAndClose"/>
    <button id="closeBtn" action="windowClose"/>
</hbox>
```

```xml
<!-- Jmix 2.x -->
<actions>
    <action id="saveAction" type="detail_saveClose"/>
    <action id="closeAction" type="detail_close"/>
    <action id="enableEditingAction" type="detail_enableEditing"/>
</actions>
<!-- ... -->
<hbox id="detailActions">
    <button id="saveAndCloseBtn" action="saveAction"/>
    <button id="closeBtn" action="closeAction"/>
    <button id="enableEditingBtn" action="enableEditingAction"/>
</hbox>
```

## 4. Nested Tables (Detail Views)

When using tables inside detail views (e.g., for composition relationships), specify `openMode="DIALOG"` to open editors in a dialog:

### Jmix 1.x

```xml
<table id="petsTable" dataContainer="petsDc" width="100%">
    <actions>
        <action id="create" type="create"/>
        <action id="edit" type="edit"/>
        <action id="remove" type="remove"/>
    </actions>
    <columns>
        <column id="name"/>
        <column id="identificationNumber"/>
        <column id="birthdate"/>
    </columns>
    <buttonsPanel>
        <button action="petsTable.create"/>
        <button action="petsTable.edit"/>
        <button action="petsTable.remove" stylename="danger"/>
    </buttonsPanel>
</table>
```

### Jmix 2.x

```xml
<vbox width="100%" padding="false">
    <hbox id="buttonsPanel" classNames="buttons-panel">
        <button action="petsDataGrid.create"/>
        <button action="petsDataGrid.edit"/>
        <button action="petsDataGrid.remove"/>
    </hbox>
    <dataGrid id="petsDataGrid" dataContainer="petsDc" width="100%" minHeight="20em">
        <actions>
            <action id="create" type="list_create">
                <properties>
                    <property name="openMode" value="DIALOG"/>
                </properties>
            </action>
            <action id="edit" type="list_edit">
                <properties>
                    <property name="openMode" value="DIALOG"/>
                </properties>
            </action>
            <action id="remove" type="list_remove"/>
        </actions>
        <columns>
            <column property="name"/>
            <column property="identificationNumber"/>
            <column property="birthdate"/>
        </columns>
    </dataGrid>
</vbox>
```

**Key changes for nested tables:**
- Wrap in `<vbox padding="false">` for proper layout
- Add `<properties><property name="openMode" value="DIALOG"/></properties>` to create/edit actions
- Use `minHeight="20em"` to ensure proper grid display

## 5. Inline Editing in DataGrid (Jmix 2.x only)

Jmix 2.x introduces inline editing directly in DataGrid. This feature was not available in Jmix 1.x Classic UI tables.

### Buffered Mode

In buffered mode, changes are applied only after clicking Save button:

```xml
<dataGrid id="itemsDataGrid" dataContainer="itemsDc" width="100%" minHeight="20em"
          editorBuffered="true">
    <columns>
        <column property="name" editable="true"/>
        <column property="quantity" editable="true"/>
        <column property="price" editable="true"/>
        <editorActionsColumn width="12em">
            <editButton icon="PENCIL" text="Edit"/>
            <saveButton icon="CHECK" themeNames="success"/>
            <cancelButton icon="CLOSE" themeNames="error"/>
        </editorActionsColumn>
    </columns>
</dataGrid>
```

### Non-Buffered Mode

In non-buffered mode, changes are applied immediately on field blur:

```xml
<dataGrid id="itemsDataGrid" dataContainer="itemsDc" width="100%" minHeight="20em"
          editorBuffered="false">
    <columns>
        <column property="name" editable="true"/>
        <column property="quantity" editable="true"/>
        <column property="price" editable="true"/>
        <editorActionsColumn width="8em">
            <editButton icon="PENCIL"/>
            <closeButton icon="CLOSE"/>
        </editorActionsColumn>
    </columns>
</dataGrid>
```

**Key points:**
- `editorBuffered="true"` — requires explicit Save/Cancel, use `<saveButton>` and `<cancelButton>`
- `editorBuffered="false"` — changes apply immediately, use `<closeButton>` instead of save/cancel
- Mark columns as `editable="true"` to enable editing
- Use `<editorActionsColumn>` for row-level edit controls

## 6. Action Classes

| Jmix 1.x | Jmix 2.x |
|----------|----------|
| `io.jmix.ui.action.Action` | `io.jmix.flowui.kit.action.Action` |
| `io.jmix.ui.action.list.ItemTrackingAction` | `io.jmix.flowui.action.list.ItemTrackingAction` |

## 7. enabledRule for Actions

```java
// Jmix 1.x
@Install(to = "customersTable.remove", subject = "enabledRule")
private boolean customersTableRemoveEnabledRule() {
    Set<Customer> selected = customersTable.getSelected();
    return canBeRemoved(selected);
}
```

```java
// Jmix 2.x
@Install(to = "customersDataGrid.remove", subject = "enabledRule")
private boolean customersDataGridRemoveEnabledRule() {
    Set<Customer> selected = customersDataGrid.getSelectedItems();
    return canBeRemoved(selected);
}
```

**Changes:** `getSelected()` -> `getSelectedItems()`

## 8. Refreshing Action State

```java
// Jmix 1.x
@Subscribe("ownersTable")
public void onOwnersTableSelection(Table.SelectionEvent<Owner> event) {
    Action action = petsTable.getActionNN("create");
    action.refreshState();
}
```

```java
// Jmix 2.x
@Subscribe("ownersDataGrid")
public void onOwnersDataGridSelection(
        SelectionEvent<DataGrid<Owner>, Owner> event) {
    Action action = petsDataGrid.getAction("create");
    if (action != null) {
        action.refreshState();
    }
}
```

## 9. TreeDataGrid Migration

For hierarchical data display, use `treeDataGrid`:

### Jmix 1.x

```xml
<treeTable id="assetsTable"
           dataContainer="assetsDc"
           hierarchyProperty="parent">
    <columns>
        <column id="name"/>
        <column id="type"/>
    </columns>
</treeTable>
```

### Jmix 2.x

```xml
<treeDataGrid id="assetsDataGrid"
              dataContainer="assetsDc"
              hierarchyProperty="parent">
    <columns>
        <column property="name"/>
        <column property="type"/>
    </columns>
</treeDataGrid>
```

### Custom Hierarchy Column with Icons

```java
@ViewComponent
private TreeDataGrid<Asset> assetsDataGrid;

@Subscribe
public void onInit(InitEvent event) {
    // Remove default hierarchy column
    if (assetsDataGrid.getColumnByKey("name") != null) {
        assetsDataGrid.removeColumn(assetsDataGrid.getColumnByKey("name"));
    }

    // Add custom hierarchy column with icons
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

| Jmix 1.x                        | Jmix 2.x                                                                                        |
|---------------------------------|-------------------------------------------------------------------------------------------------|
| `table`                         | `dataGrid`                                                                                      |
| `groupTable`                    | `dataGrid` (or `groupDataGrid` with [add-on](https://www.jmix.io/marketplace/group-data-grid/)) |
| `treeTable`                     | `treeDataGrid`                                                                                  |
| `buttonsPanel` inside table     | `hbox classNames="buttons-panel"` outside                                                       |
| `simplePagination` inside table | `simplePagination dataLoader="..."` outside                                                     |
| `column id="prop"`              | `column property="prop"`                                                                        |
| `multiselect="true"`            | `selectionMode="MULTI"`                                                                         |
| `stylename="danger"`            | `themeNames="error"`                                                                            |
| `getSelected()`                 | `getSelectedItems()`                                                                            |
| `getActionNN()`                 | `getAction()` (nullable)                                                                        |
| `relatedEntities`               | **Not available** - use DialogWindows                                                           |
| *(implicit)*                    | `minHeight="20em"` on dataGrid                                                                  |
| *(implicit)*                    | `<startSlot>` / `<endSlot>` in hbox                                                             |
| *(implicit)*                    | `openMode="DIALOG"` for nested table actions                                                    |
