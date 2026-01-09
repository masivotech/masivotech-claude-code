---
description: Run IntelliJ Plugin Verifier to check compatibility
---

# Verify IntelliJ Plugin Compatibility

Run the IntelliJ Plugin Verifier to check plugin compatibility across multiple IDE versions.

## Instructions

1. Check if the project has Gradle IntelliJ Plugin configured:
   - Look for `intellijPlatform` or `intellij` configuration in `build.gradle.kts`

2. Run the plugin verifier:

```bash
./gradlew runPluginVerifier
```

3. If verifier is not configured, help the user add it to `build.gradle.kts`:

### For Gradle IntelliJ Plugin 2.x
```kotlin
dependencies {
    intellijPlatform {
        pluginVerifier()
    }
}

intellijPlatform {
    pluginVerification {
        ides {
            recommended()
            // Or specify exact versions:
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.1")
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.2")
        }
    }
}
```

### For Gradle IntelliJ Plugin 1.x
```kotlin
tasks {
    runPluginVerifier {
        ideVersions.set(listOf(
            "IC-2024.1",
            "IC-2024.2",
            "IC-2024.3"
        ))
    }
}
```

4. Interpret the results:

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| **Deprecated API usage** | Update to the new API. Check SDK docs for replacement. |
| **Experimental API usage** | Add `@OptIn` annotation or use stable alternative. |
| **Internal API usage** | Replace with public API. Internal APIs can change without notice. |
| **Missing dependency** | Add `<depends>` in plugin.xml for the required plugin. |
| **Incompatible change** | API changed between versions. Implement version-specific handling. |
| **Non-extendable API** | Class/method not designed for extension. Find alternative approach. |

5. For deprecation warnings, provide the recommended replacement:

```kotlin
// Old (deprecated)
ApplicationManager.getApplication().invokeLater { ... }

// New (recommended)
ApplicationManager.getApplication().invokeLater(ModalityState.defaultModalityState()) { ... }
```

6. Muting specific issues (use sparingly):

```kotlin
intellijPlatform {
    pluginVerification {
        freeArgs.addAll(
            "-mute", "TemplateWordInPluginId",
            "-mute", "ForbiddenPluginIdPrefix"
        )
    }
}
```

## Verifier Output

The verifier creates reports in:
```
build/reports/pluginVerifier/
```

Each IDE version gets its own report showing:
- Compatible: No issues found
- Warnings: Non-critical issues
- Problems: Breaking compatibility issues

## CI Integration

Add to GitHub Actions workflow:

```yaml
- name: Run Plugin Verifier
  run: ./gradlew runPluginVerifier

- name: Upload Verifier Results
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: pluginVerifier-results
    path: build/reports/pluginVerifier
```

## Best Practices

1. Run verifier before every release
2. Test against at least the last 3 major IDE versions
3. Include Android Studio if targeting that platform
4. Fix deprecation warnings proactively
5. Never suppress internal API warnings without a migration plan
