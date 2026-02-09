# Entity Migration Rules (Jmix 1 -> Jmix 2)

## Overview

Entity migration from Jmix 1.x to Jmix 2.x involves several changes beyond just namespace migration. The main changes are:

1. **Namespace migration** from Java EE (`javax.*`) to Jakarta EE (`jakarta.*`)
2. **Date/Time types** migration from `java.util.Date` to `java.time.*` classes
3. **@InstanceName methods** require additional annotations

## Primary Change: javax -> jakarta

```java
// Jmix 1.x
import javax.persistence.*;
import javax.validation.constraints.*;
```

```java
// Jmix 2.x
import jakarta.persistence.*;
import jakarta.validation.constraints.*;
```

## Date/Time Migration

Jmix 2.x recommends Java 8+ time API. `java.util.Date` is **still supported** but `java.time.*` is preferred:

```java
// Jmix 1.x
import java.util.Date;

@CreatedDate
@Column(name = "CREATED_DATE")
@Temporal(TemporalType.TIMESTAMP)
private Date createdDate;

@LastModifiedDate
@Column(name = "LAST_MODIFIED_DATE")
@Temporal(TemporalType.TIMESTAMP)
private Date lastModifiedDate;
```

```java
// Jmix 2.x - No @Temporal needed!
import java.time.OffsetDateTime;

@CreatedDate
@Column(name = "CREATED_DATE")
private OffsetDateTime createdDate;

@LastModifiedDate
@Column(name = "LAST_MODIFIED_DATE")
private OffsetDateTime lastModifiedDate;
```

**Key changes:**
- `Date` → `OffsetDateTime` (or `LocalDateTime`, `LocalDate` depending on use case)
- for audit of creation or modifiaation Jmix Studio generates and prefers OffsetDateTime
- Remove `@Temporal(TemporalType.TIMESTAMP)` annotation - not needed with Java 8 time types
- Update all getters/setters to use the new type

## @InstanceName Methods

When using `@InstanceName` on methods, `@DependsOnProperties` is **required** in both Jmix 1.x and 2.x:

```java
// Both Jmix 1.x and Jmix 2.x - @DependsOnProperties REQUIRED for methods
@InstanceName
@DependsOnProperties({"firstName", "lastName"})  // REQUIRED for @InstanceName on methods
public String getFullName() {
    String name = firstName;
    if (lastName != null) {
        name += " " + lastName;
    }
    return name;
}
```

**Reminder:** If migrating code that is missing `@DependsOnProperties` on `@InstanceName` methods, add it during migration. This annotation is required for:
- Correct `_instance_name` fetch plan generation
- UI change tracking (EntityPropertyChangeEvent)
- Loading references from different data stores

**Note:** `@JmixProperty` is **optional** - only required if:
- `@JmixEntity(annotatedPropertiesOnly = true)` is set, OR
- You need this property exposed in metadata (e.g., for security policies)

## What Stays the Same

All Jmix-specific annotations remain unchanged:

- `@JmixEntity`
- `@JmixGeneratedValue`
- `@InstanceName`
- `@DependsOnProperties`
- `@JmixProperty`
- `@Composition`
- `@OnDelete`
- `@Store`

## Example: Full Entity Migration

```java
// Jmix 1.x
package io.jmix.petclinic.entity.owner;

import io.jmix.core.DeletePolicy;
import io.jmix.core.entity.annotation.OnDelete;
import io.jmix.core.metamodel.annotation.Composition;
import io.jmix.core.metamodel.annotation.JmixEntity;
import io.jmix.petclinic.entity.Person;
import io.jmix.petclinic.entity.pet.Pet;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.OneToMany;
import javax.persistence.OrderBy;
import javax.persistence.Table;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotNull;
import java.util.List;

@JmixEntity
@Table(name = "PETCLINIC_OWNER")
@Entity(name = "petclinic_Owner")
public class Owner extends Person {

    @NotNull
    @Column(name = "ADDRESS", nullable = false)
    private String address;

    @Email
    @Column(name = "EMAIL")
    private String email;

    @OnDelete(DeletePolicy.CASCADE)
    @Composition
    @OrderBy("identificationNumber")
    @OneToMany(mappedBy = "owner")
    private List<Pet> pets;

    // getters and setters
}
```

```java
// Jmix 2.x
package io.jmix.petclinic.entity.owner;

import io.jmix.core.DeletePolicy;
import io.jmix.core.entity.annotation.OnDelete;
import io.jmix.core.metamodel.annotation.Composition;
import io.jmix.core.metamodel.annotation.JmixEntity;
import io.jmix.petclinic.entity.Person;
import io.jmix.petclinic.entity.pet.Pet;

import jakarta.persistence.*;            // Changed from javax
import jakarta.validation.constraints.*; // Changed from javax
import java.util.List;

@JmixEntity
@Table(name = "PETCLINIC_OWNER")
@Entity(name = "petclinic_Owner")
public class Owner extends Person {

    @NotNull
    @Column(name = "ADDRESS", nullable = false)
    private String address;

    @Email
    @Column(name = "EMAIL")
    private String email;

    @OnDelete(DeletePolicy.CASCADE)
    @Composition
    @OrderBy("identificationNumber")
    @OneToMany(mappedBy = "owner")
    private List<Pet> pets;

    // getters and setters
}
```

## Example: Base Entity with Audit Fields

```java
// Jmix 1.x
@JmixEntity(name = "petclinic_Person")
@MappedSuperclass
public class Person {
    @CreatedDate
    @Column(name = "CREATED_DATE")
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @InstanceName
    public String getName() {
        return firstName + " " + lastName;
    }
    // ...
}
```

```java
// Jmix 2.x
@JmixEntity(name = "petclinic_Person")
@MappedSuperclass
public class Person {
    @CreatedDate
    @Column(name = "CREATED_DATE")
    private OffsetDateTime createdDate;  // Date → OffsetDateTime, no @Temporal

    @InstanceName
    @DependsOnProperties({"firstName", "lastName"})  // NEW
    public String getFullName() {
        return firstName + " " + lastName;
    }
    // ...
}
```

## Migration Checklist

For each entity file:

1. Replace `import javax.persistence.*` with `import jakarta.persistence.*`
2. Replace `import javax.validation.*` with `import jakarta.validation.*`
3. Replace `import javax.annotation.*` with `import jakarta.annotation.*` (if used)
4. **Date fields:**
   - Replace `Date` with `OffsetDateTime` (or `LocalDateTime`/`LocalDate`)
   - Remove `@Temporal(TemporalType.TIMESTAMP)` annotations
   - Update getters/setters
5. **@InstanceName methods:**
   - Add `@DependsOnProperties({"prop1", "prop2"})` with referenced properties (**required**)
   - Add `@JmixProperty` annotation (only if `annotatedPropertiesOnly = true` or metadata exposure needed)
   - Import `io.jmix.core.metamodel.annotation.DependsOnProperties`
   - Import `io.jmix.core.metamodel.annotation.JmixProperty` (if used)

## Enums

Enums implementing `EnumClass` remain the same:

```java
// Same in both Jmix 1.x and 2.x
public enum BookingChannelEnum implements EnumClass<String> {
    WEB("web"),
    APP("app");

    private String id;

    BookingChannelEnum(String value) {
        this.id = value;
    }

    public String getId() {
        return id;
    }

    @Nullable
    public static BookingChannelEnum fromId(String id) {
        for (BookingChannelEnum en : BookingChannelEnum.values()) {
            if (en.getId().equals(id)) {
                return en;
            }
        }
        return null;
    }
}
```

## Embedded Entities

No changes needed beyond imports:

```java
// Same structure, just change javax -> jakarta
@JmixEntity
@Embeddable
public class Address {
    @Column(name = "STREET")
    private String street;
    @Column(name = "CITY")
    private String city;
}
```

## DTO / Non-persisted Entities

No changes needed:

```java
@JmixEntity(name = "myapp_ZoneCountry")
public class ZoneCountry {

    @JmixGeneratedValue
    @JmixId
    private String id;

    @InstanceName
    @JmixProperty(mandatory = true)
    @Size(min = 2, max = 3)
    private String code;

    // getters and setters
}
```

## Messages

Entity messages format remains the same:

```properties
# Same format in Jmix 1.x and 2.x
com.company.myapp.entity.partner/Partner=Partner
com.company.myapp.entity.partner/Partner.name=Name
com.company.myapp.entity.partner/Partner.email=Email
```

## Quick Migration Script

For bulk replacement, you can use these patterns:

| Find                                | Replace                            |
|-------------------------------------|------------------------------------|
| `import javax.persistence.`         | `import jakarta.persistence.`      |
| `import javax.validation.`          | `import jakarta.validation.`       |
| `import javax.annotation.`          | `import jakarta.annotation.`       |
| `import java.util.Date;`            | `import java.time.OffsetDateTime;` |
| `private Date `                     | `private OffsetDateTime `          |
| `@Temporal(TemporalType.TIMESTAMP)` | *(remove)*                         |

**Note:** Date/Time and @InstanceName changes require manual review - they cannot be safely automated.