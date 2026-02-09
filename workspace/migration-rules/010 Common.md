# Common Migration Rules (Jmix 1 -> Jmix 2)

## System Requirements

| Requirement      | Jmix 1.x         | Jmix 2.x             |
|------------------|------------------|----------------------|
| **JDK**          | 11+              | **17+** (mandatory)  |
| **Tomcat** (WAR) | 9.x              | **10.x** (mandatory) |
| **Spring Boot**  | 2.7.x            | 3.x                  |
| **Vaadin**       | 8.x (Classic UI) | 24.x (Flow UI)       |
| **Gradle**       | 7.x              | 8.x (recommended)    |

**⚠️ CRITICAL:** Jmix 2.x requires JDK 17+ and Tomcat 10+ for deployment. These are mandatory requirements due to Jakarta EE namespace migration.

For detailed build configuration, see **140 Build and Deployment.md**.

## Imports

The main backend change is migration from Java EE (`javax.*`) to Jakarta EE (`jakarta.*`):

```java
// Jmix 1.x (before)

import javax.persistence.*;
import javax.validation.constraints.*;

import io.jmix.ui.screen.*;
import io.jmix.ui.navigation.Route;
import io.jmix.ui.component.*;
```

```java
// Jmix 2.x (after)

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import io.jmix.flowui.view.*;
import com.vaadin.flow.router.Route;
import io.jmix.flowui.component.*;
```

### UI Package Mapping

| Jmix 1.x                      | Jmix 2.x                       |
|-------------------------------|--------------------------------|
| `io.jmix.ui.screen.*`         | `io.jmix.flowui.view.*`        |
| `io.jmix.ui.component.*`      | `io.jmix.flowui.component.*`   |
| `io.jmix.ui.model.*`          | `io.jmix.flowui.model.*`       |
| `io.jmix.ui.action.*`         | `io.jmix.flowui.action.*`      |
| `io.jmix.ui.navigation.Route` | `com.vaadin.flow.router.Route` |
| `io.jmix.ui.Notifications`    | `io.jmix.flowui.Notifications` |
| `io.jmix.ui.Dialogs`          | `io.jmix.flowui.Dialogs`       |

## Build Configuration

```groovy
// build.gradle (Jmix 1.x)
plugins {
    id 'io.jmix' version '1.x.x'
    // ...
}

dependencies {
    implementation 'io.jmix.core:jmix-core-starter'
    implementation 'io.jmix.ui:jmix-ui-starter'
    implementation 'io.jmix.ui:jmix-ui-data-starter'
}
```

```groovy
// build.gradle (Jmix 2.x)
plugins {
    id 'io.jmix' version '2.x.x'
    id 'com.vaadin' version '24.x.x'  // Required for Flow UI!
    // ...
}

vaadin {
    optimizeBundle = false  // Required for all builds (since Vaadin 24.1.9)
}

dependencies {
    implementation 'io.jmix.core:jmix-core-starter'
    implementation 'io.jmix.flowui:jmix-flowui-starter'
    implementation 'io.jmix.flowui:jmix-flowui-data-starter'
    implementation 'io.jmix.datatools:jmix-datatools-starter'
}
```

**Important:** Jmix 2.x requires the Vaadin Gradle plugin (`com.vaadin`) and the `vaadin` block configuration.

## Messages

Messages format is similar between Jmix 1 and Jmix 2. The main difference is the key prefix for views:

```properties
# Jmix 1.x (screen)
com.company.myapp.screen.partner/partnerBrowse.caption=Partner list
# Jmix 2.x (view)
com.company.myapp.view.partner/PartnerListView.title=Partner list
```

Note: `caption` -> `title` for view titles.

## Common classes

Most Jmix core classes remain the same. Key changes:

- `io.jmix.ui.screen.MessageBundle` -> `io.jmix.flowui.view.MessageBundle`
- `io.jmix.ui.screen.Screen` -> `io.jmix.flowui.view.View`
- `io.jmix.ui.screen.ScreenFragment` -> `io.jmix.flowui.fragment.Fragment`

## Logger

No changes needed for logging. Continue using:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class SampleBean {
    private static final Logger log = LoggerFactory.getLogger(SampleBean.class);
}
```

## Configs for multi-data stores

Configuration remains similar. Main change is UI configuration class:

```java

@Configuration
@ComponentScan(basePackages = "com.company.myapp")
@ConfigurationPropertiesScan
@JmixModule(dependsOn = {EclipselinkConfiguration.class, FlowuiConfiguration.class})
@PropertySource(
        name = "com.company.myapp",
        value = "classpath:/com/company/myapp/module.properties"
)
public class MyappConfiguration {

    @Bean("myapp_MyappViewControllers")
    public ViewControllersConfiguration screens(final ApplicationContext applicationContext,
                                                final AnnotationScanMetadataReaderFactory metadataReaderFactory) {
        final ViewControllersConfiguration viewControllers
                = new ViewControllersConfiguration(applicationContext, metadataReaderFactory);
        viewControllers.setBasePackages(Collections.singletonList("com.company.myapp"));
        return viewControllers;
    }

    // DataSource beans remain the same as in Jmix 1.x
    // ...
}
```

## Security permission checking

Security API remains mostly the same between Jmix 1 and Jmix 2:

```java

@Component
public class SecurityPermissions {

    @Autowired
    private Metadata metadata;
    @Autowired
    private PolicyStore policyStore;
    @Autowired
    private SecureOperations secureOperations;

    public boolean isEntityOpPermitted(Class<?> entityClass, EntityPolicyAction entityPolicyAction) {
        MetaClass metaClass = metadata.getClass(entityClass);
        switch (entityPolicyAction) {
            case CREATE -> {
                return secureOperations.isEntityCreatePermitted(metaClass, policyStore);
            }
            case UPDATE -> {
                return secureOperations.isEntityUpdatePermitted(metaClass, policyStore);
            }
            case READ -> {
                return secureOperations.isEntityReadPermitted(metaClass, policyStore);
            }
            case DELETE -> {
                return secureOperations.isEntityDeletePermitted(metaClass, policyStore);
            }
            default -> throw new UnsupportedOperationException();
        }
    }

    public void checkSpecificPermission(String permissionId) {
        if (!secureOperations.isSpecificPermitted(permissionId, policyStore))
            throw new AccessDeniedException("specific", permissionId);
    }
}
```
