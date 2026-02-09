# UI Fragments Migration Rules (Jmix 1 -> Jmix 2)

## Overview

Fragments are reusable UI parts that can be embedded into screens (Jmix 1.x) or views (Jmix 2.x). The migration involves changes in how fragments are defined, included, and interact with data.

## Defining a Fragment: Class and XML

### Jmix 1.x

```java
@UiController("demo_AddressFragment")
@UiDescriptor("address-fragment.xml")
public class AddressFragment extends ScreenFragment {
    // ... fragment logic
}
```

```xml
<!-- address-fragment.xml -->
<fragment xmlns="http://jmix.io/schema/ui/fragment">
    <layout>
        <textField id="cityField" caption="City"/>
        <textField id="zipField" caption="Zip"/>
    </layout>
</fragment>
```

### Jmix 2.x

```java
@FragmentDescriptor("address-fragment.xml")
public class AddressFragment extends Fragment<FormLayout> {
    // ... fragment logic
}
```

```xml
<!-- address-fragment.xml -->
<fragment xmlns="http://jmix.io/schema/flowui/fragment">
    <content>
        <formLayout>
            <textField id="cityField" label="City"/>
            <textField id="zipcodeField" label="Zipcode"/>
        </formLayout>
    </content>
</fragment>
```

**Key differences:**
- No `@UiController` ID needed in Jmix 2.x - fragment is referenced by class
- `<layout>` -> `<content>` wrapper
- `caption` -> `label` for components

## Including Fragments in a Screen/View

### Declarative Inclusion (XML)

```xml
<!-- Jmix 1.x host screen -->
<window xmlns="http://jmix.io/schema/ui/window">
    <layout>
        <groupBox id="addressBox" caption="Address">
            <fragment screen="demo_AddressFragment"/>
        </groupBox>
    </layout>
</window>
```

```xml
<!-- Jmix 2.x host view -->
<view xmlns="http://jmix.io/schema/flowui/view">
    <layout>
        <details id="addressDetails" summaryText="Address" opened="true">
            <fragment class="com.company.project.view.address.AddressFragment"/>
        </details>
    </layout>
</view>
```

**Key difference:** `screen="fragmentId"` -> `class="fully.qualified.ClassName"`

### Programmatic Inclusion

```java
// Jmix 1.x
@Autowired
private Fragments fragments;
@Autowired
private GroupBoxLayout addressBox;

@Subscribe
private void onInit(InitEvent event) {
    AddressFragment addrFragment = fragments.create(this, AddressFragment.class);
    addressBox.add(addrFragment.getFragment());
}
```

```java
// Jmix 2.x
@Autowired
private Fragments fragments;
@ViewComponent
private Details addressDetails;

@Subscribe
public void onInit(InitEvent event) {
    AddressFragment addrFragment = fragments.create(this, AddressFragment.class);
    addressDetails.add(addrFragment);  // no getFragment() needed
}
```

## Dependency Injection in Fragment Controllers

```java
// Jmix 1.x
@UiController("demo_AddressFragment")
public class AddressFragment extends ScreenFragment {

    @Autowired
    private TextField<String> cityField;

    @Autowired
    private CollectionLoader<City> citiesDl;
}
```

```java
// Jmix 2.x
@FragmentDescriptor("address-fragment.xml")
public class AddressFragment extends Fragment<FormLayout> {

    @ViewComponent
    private EntityComboBox<City> cityField;

    @ViewComponent
    private CollectionLoader<City> citiesDl;

    @Autowired
    private Messages messages;
}
```

## Passing Parameters to Fragments

### Jmix 1.x

```xml
<fragment screen="demo_AddressFragment">
    <properties>
        <property name="customer" ref="customerDc"/>
        <property name="note" value="Additional Info"/>
    </properties>
</fragment>
```

### Jmix 2.x

```xml
<fragment class="com.company.project.view.AddressFragment">
    <properties>
        <property name="customerDc" value="customersDc" type="CONTAINER_REF"/>
        <property name="note" value="Additional Info"/>
    </properties>
</fragment>
```

**Key difference:** Use `type="CONTAINER_REF"` or `type="COMPONENT_REF"` instead of `ref` attribute.

## Data in Fragments (Provided Containers)

```xml
<!-- Jmix 1.x fragment with provided data -->
<fragment xmlns="http://jmix.io/schema/ui/fragment">
    <data>
        <instance id="addressDc"
                  class="com.company.demo.entity.Address"
                  provided="true"/>
        <collection id="citiesDc" class="com.company.demo.entity.City">
            <loader id="citiesLd">
                <query><![CDATA[select e from demo_City e]]></query>
            </loader>
        </collection>
    </data>
    <layout>
        <lookupField id="cityField"
                     optionsContainer="citiesDc"
                     dataContainer="addressDc"
                     property="city"/>
    </layout>
</fragment>
```

```xml
<!-- Jmix 2.x fragment with provided data -->
<fragment xmlns="http://jmix.io/schema/flowui/fragment">
    <data>
        <instance id="addressDc"
                  class="com.company.onboarding.entity.Address"
                  provided="true"/>
        <collection id="citiesDc"
                    class="com.company.onboarding.entity.City"
                    provided="true"/>
    </data>
    <content>
        <formLayout dataContainer="addressDc">
            <entityComboBox id="cityField"
                            itemsContainer="citiesDc"
                            property="city"/>
        </formLayout>
    </content>
</fragment>
```

## Loading Data in Fragments

Fragments don't support `<dataLoadCoordinator>` facet. Load data manually:

```java
// Jmix 1.x
@Subscribe(target = Target.PARENT_CONTROLLER)
protected void onHostBeforeShow(Screen.BeforeShowEvent event) {
    citiesDl.load();
}
```

```java
// Jmix 2.x - NOTE: method must be PUBLIC, not protected!
@Subscribe(target = Target.HOST_CONTROLLER)
public void onHostBeforeShow(View.BeforeShowEvent event) {
    citiesDl.load();
}
```

## Quick Migration Checklist

| Jmix 1.x                    | Jmix 2.x                                   |
|-----------------------------|--------------------------------------------|
| `@UiController("id")`       | Remove (not needed)                        |
| `@UiDescriptor("file.xml")` | `@FragmentDescriptor("file.xml")`          |
| `extends ScreenFragment`    | `extends Fragment<LayoutType>`             |
| `screen="fragmentId"`       | `class="full.ClassName"`                   |
| `ref="containerId"`         | `value="containerId" type="CONTAINER_REF"` |
| `<layout>`                  | `<content>`                                |
| `Target.PARENT_CONTROLLER`  | `Target.HOST_CONTROLLER`                   |
| `getFragment()`             | Not needed                                 |
| `protected void onXxx()`    | `public void onXxx()`                      |

**⚠️ IMPORTANT:** In Jmix 2.x, all event handler methods must be `public`, not `protected`!
