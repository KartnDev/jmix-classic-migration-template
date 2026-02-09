# UI View Descriptors Data Section Migration Rules (Jmix 1 -> Jmix 2)

## Overview

The data section in XML descriptors remains mostly the same between Jmix 1.x and Jmix 2.x. Key differences are minimal.

## Basic Data Container

Jump into summary if you no need details. Almost all things same as in Jmix 1 also same for Jmix 2.

### Jmix 1.x

```xml
<data readOnly="true">
    <collection id="ownersDc"
                class="io.jmix.petclinic.entity.owner.Owner">
        <fetchPlan extends="_base"/>
        <loader id="ownersDl">
            <query>
                <![CDATA[select e from petclinic_Owner e]]>
            </query>
        </loader>
    </collection>
</data>
```

### Jmix 2.x

```xml
<data readOnly="true">
    <collection id="ownersDc"
                class="io.jmix.petclinic.entity.owner.Owner">
        <fetchPlan extends="_base"/>
        <loader id="ownersDl">
            <query>
                <![CDATA[select e from petclinic_Owner e]]>
            </query>
        </loader>
    </collection>
</data>
```

**No changes needed** - the data section syntax is identical.

## Instance Containers (Edit/Detail Views)

### Jmix 1.x

```xml
<data>
    <instance id="ownerDc"
              class="io.jmix.petclinic.entity.owner.Owner">
        <fetchPlan extends="_base">
            <property name="pets" fetchPlan="_base"/>
        </fetchPlan>
        <loader/>
        <collection id="petsDc" property="pets"/>
    </instance>
</data>
```

### Jmix 2.x

```xml
<data>
    <instance id="ownerDc"
              class="io.jmix.petclinic.entity.owner.Owner">
        <fetchPlan extends="_base">
            <property name="pets" fetchPlan="_base">
                <property name="type" fetchPlan="_instance_name"/>
            </property>
        </fetchPlan>
        <loader/>
        <collection id="petsDc" property="pets"/>
    </instance>
</data>
```

**No structural changes** - only content may differ based on entity requirements.

## Facets Section

### Jmix 1.x

```xml
<facets>
    <dataLoadCoordinator auto="true"/>
    <screenSettings id="settingsFacet" auto="true"/>
</facets>
```

### Jmix 2.x

```xml
<facets>
    <dataLoadCoordinator auto="true"/>
    <urlQueryParameters>
        <genericFilter component="genericFilter"/>
        <pagination component="pagination"/>
    </urlQueryParameters>
    <settings auto="true"/>
</facets>
```

**Key changes:**
- `screenSettings` -> `settings` (simplified name)
- New `urlQueryParameters` for filter and pagination state in URL

## Provided Data Containers (Fragments)

When using fragments with data from host screen, mark containers as `provided="true"`.

### Both Jmix 1.x and 2.x

```xml
<data>
    <instance id="ownerDc"
              class="io.jmix.petclinic.entity.owner.Owner"
              provided="true">
        <collection id="petsDc" property="pets" provided="true"/>
    </instance>
</data>
```

**No changes** - `provided="true"` works the same way.

## Complex Nested Data

### Jmix 1.x

```xml
<data>
    <instance id="platformDc"
              class="com.company.addon.registry.entity.Platform">
        <fetchPlan extends="platform-fetchPlan"/>

        <collection id="settingsDc" property="settings"/>

        <collection id="productMappingsDc" property="productMappings">
            <collection id="mappingValuesDc" property="values">
                <collection id="conditionalValuesDc" property="conditionalValues"/>
            </collection>
        </collection>

        <loader id="platformDl"/>
    </instance>
</data>
```

### Jmix 2.x

```xml
<data>
    <instance id="platformDc"
              class="com.company.addon.registry.entity.Platform">
        <fetchPlan extends="platform-fetchPlan"/>

        <collection id="settingsDc" property="settings"/>

        <collection id="productMappingsDc" property="productMappings">
            <collection id="mappingValuesDc" property="values">
                <collection id="conditionalValuesDc" property="conditionalValues"/>
            </collection>
        </collection>

        <loader id="platformDl"/>
    </instance>
</data>
```

**Identical** - nested collections work the same way.

## Quick Reference

| Element                | Jmix 1.x | Jmix 2.x              |
|------------------------|----------|-----------------------|
| `<data>`               | Same     | Same                  |
| `<collection>`         | Same     | Same                  |
| `<instance>`           | Same     | Same                  |
| `<fetchPlan>`          | Same     | Same                  |
| `<loader>`             | Same     | Same                  |
| `provided="true"`      | Same     | Same                  |
| `<screenSettings>`     | Yes      | No - use `<settings>` |
| `<settings>`           | No       | Yes                   |
| `<urlQueryParameters>` | No       | Yes (optional)        |

## Migration Summary

1. **Data containers** - copy as-is, no changes
2. **Fetch plans** - copy as-is, no changes
3. **Loaders** - copy as-is, no changes
4. **Facets** - rename `screenSettings` to `settings`, optionally add `urlQueryParameters`