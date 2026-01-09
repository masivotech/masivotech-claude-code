---
description: Check and fix IDE version compatibility settings
---

# Check IntelliJ Plugin IDE Compatibility

Analyze and update plugin compatibility settings for target IDE versions.

## Instructions

1. Read the current compatibility configuration from:
   - `build.gradle.kts` - sinceBuild and untilBuild settings
   - `plugin.xml` - idea-version element
   - `gradle.properties` - version properties

2. Display current settings:

```
Current Compatibility:
- Since Build: <sinceBuild>
- Until Build: <untilBuild>
- Target IDE: <platformVersion>
```

3. Explain the build number format:

| Build Number | IDE Version | Release |
|--------------|-------------|---------|
| 233 | 2023.3 | Dec 2023 |
| 241 | 2024.1 | Apr 2024 |
| 242 | 2024.2 | Aug 2024 |
| 243 | 2024.3 | Dec 2024 |
| 251 | 2025.1 | Apr 2025 |

4. Ask the user what they want to do:
   - Update to support newer IDE versions
   - Narrow support to specific versions
   - Check if current settings are optimal

5. When updating compatibility:

### build.gradle.kts (Gradle IntelliJ Plugin 2.x)
```kotlin
intellijPlatform {
    pluginConfiguration {
        ideaVersion {
            sinceBuild = "<newSinceBuild>"
            untilBuild = "<newUntilBuild>.*"
        }
    }
}
```

### build.gradle.kts (Gradle IntelliJ Plugin 1.x)
```kotlin
patchPluginXml {
    sinceBuild.set("<newSinceBuild>")
    untilBuild.set("<newUntilBuild>.*")
}
```

### gradle.properties
```properties
pluginSinceBuild=<newSinceBuild>
pluginUntilBuild=<newUntilBuild>.*
```

6. Common compatibility patterns:

### Support Latest 3 Versions
```kotlin
sinceBuild = "241"    // 2024.1
untilBuild = "251.*"  // 2025.1.x
```

### Support Single Major Version
```kotlin
sinceBuild = "243"    // 2024.3
untilBuild = "243.*"  // 2024.3.x only
```

### Open-Ended Support (Not Recommended)
```kotlin
sinceBuild = "242"
// No untilBuild - supports all future versions
```

7. Check for breaking changes:

When increasing `sinceBuild`, check for removed APIs:
- Look for `@Deprecated` annotations with `forRemoval = true`
- Check [IntelliJ Platform SDK](https://plugins.jetbrains.com/docs/intellij/api-changes-list.html) for API changes

When increasing `untilBuild`, verify:
- No internal API usage that changed
- Run `./gradlew runPluginVerifier` for the new versions

8. Version-specific code:

If you need to support multiple IDE versions with API differences:

```kotlin
// Check IDE version at runtime
val buildNumber = ApplicationInfo.getInstance().build
if (buildNumber >= BuildNumber.fromString("243")!!) {
    // Use new API (2024.3+)
} else {
    // Use old API (pre-2024.3)
}
```

9. Provide recommendations:

**For new plugins:**
- Start with the 3 most recent versions
- Use `untilBuild` with wildcard for patch versions

**For existing plugins:**
- Maintain compatibility with versions users have
- Deprecate old version support gradually
- Announce EOL for older IDE versions

**For enterprise plugins:**
- Support versions in use by your organization
- Consider LTS IDE releases

10. Remind user to run verifier after changes:

```bash
./gradlew runPluginVerifier
```

## Quick Reference

| IDE Version | Build Number | Kotlin Version | Java Version |
|-------------|--------------|----------------|--------------|
| 2023.3 | 233 | 1.9.x | 17 |
| 2024.1 | 241 | 1.9.x | 17 |
| 2024.2 | 242 | 2.0.x | 17 |
| 2024.3 | 243 | 2.0.x | 17 |
| 2025.1 | 251 | 2.1.x | 21 |

## Platform Dependencies

For cross-platform support, use module dependencies:

```xml
<!-- Works in all JetBrains IDEs -->
<depends>com.intellij.modules.platform</depends>

<!-- Requires language support (most IDEs) -->
<depends>com.intellij.modules.lang</depends>

<!-- Specific to IntelliJ IDEA -->
<depends>com.intellij.modules.java</depends>

<!-- Specific to Ultimate edition -->
<depends>com.intellij.modules.ultimate</depends>
```
