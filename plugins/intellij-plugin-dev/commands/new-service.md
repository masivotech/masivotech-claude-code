---
description: Scaffold a new Service for IntelliJ plugin development
---

# Scaffold IntelliJ Service

Create a new Service class for the IntelliJ plugin in this project.

## Instructions

1. Ask the user for the following information if not provided:
   - Service name (e.g., "MyFeatureService")
   - Service level: Application, Project, or Module
   - Brief description of what the service does

2. Determine the project's package structure by examining existing source files.

3. Create the Service class based on the selected level:

### Application Service (Global Singleton)
```kotlin
package <detected.package>.services

import com.intellij.openapi.components.Service
import com.intellij.openapi.components.service

@Service
class <ServiceName> {

    // TODO: Add service methods
    fun doSomething(): String {
        return "result"
    }

    companion object {
        fun getInstance(): <ServiceName> = service()
    }
}
```

### Project Service (One Per Project)
```kotlin
package <detected.package>.services

import com.intellij.openapi.components.Service
import com.intellij.openapi.components.service
import com.intellij.openapi.project.Project

@Service(Service.Level.PROJECT)
class <ServiceName>(private val project: Project) {

    // TODO: Add service methods
    fun doSomething(): String {
        return "result for ${project.name}"
    }

    companion object {
        fun getInstance(project: Project): <ServiceName> = project.service()
    }
}
```

### Module Service (One Per Module)
```kotlin
package <detected.package>.services

import com.intellij.openapi.components.Service
import com.intellij.openapi.module.Module

@Service(Service.Level.MODULE)
class <ServiceName>(private val module: Module) {

    // TODO: Add service methods
    fun doSomething(): String {
        return "result for ${module.name}"
    }
}
```

4. Register the service in plugin.xml:

### Application Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationService serviceImplementation="<package>.services.<ServiceName>"/>
</extensions>
```

### Project Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <projectService serviceImplementation="<package>.services.<ServiceName>"/>
</extensions>
```

### Module Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <moduleService serviceImplementation="<package>.services.<ServiceName>"/>
</extensions>
```

5. Show usage examples:

### Using Application Service
```kotlin
val service = <ServiceName>.getInstance()
val result = service.doSomething()
```

### Using Project Service
```kotlin
val service = <ServiceName>.getInstance(project)
val result = service.doSomething()
```

### Using Module Service
```kotlin
val service = module.getService(<ServiceName>::class.java)
val result = service.doSomething()
```

6. Provide the user with:
   - The file path where the service was created
   - The plugin.xml registration snippet
   - Usage example code
   - Instructions to test: `./gradlew runIde`

## Service Best Practices

- Use `@Service` annotation (no interface needed for simple services)
- Use constructor injection for dependencies
- Keep services stateless when possible
- For persistent state, combine with `PersistentStateComponent`
