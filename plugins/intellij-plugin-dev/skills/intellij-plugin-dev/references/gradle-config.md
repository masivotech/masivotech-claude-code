# Gradle IntelliJ Plugin 2.x Configuration

This reference covers configuration for IntelliJ Platform Gradle Plugin 2.x, required for IDE versions 2024.2+.

## Plugin Setup

### build.gradle.kts

```kotlin
plugins {
    id("java")
    id("org.jetbrains.kotlin.jvm") version "2.0.21"
    id("org.jetbrains.intellij.platform") version "2.1.0"
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
    intellijPlatform {
        defaultRepositories()
    }
}

dependencies {
    intellijPlatform {
        // Target IDE - choose one:
        intellijIdeaCommunity("2024.2")
        // intellijIdeaUltimate("2024.2")
        // androidStudio("2024.2.1")
        // pycharmCommunity("2024.2")
        // webStorm("2024.2")
        // goland("2024.2")
        // phpStorm("2024.2")
        // rider("2024.2")
        // clion("2024.2")
        // rubyMine("2024.2")

        // Bundled plugins
        bundledPlugin("com.intellij.java")
        bundledPlugin("Git4Idea")
        bundledPlugin("org.jetbrains.kotlin")

        // Plugin dependencies from Marketplace
        plugin("com.example.otherplugin:1.0.0")

        // Development tools
        pluginVerifier()
        zipSigner()
        instrumentationTools()
    }

    // Test dependencies
    testImplementation("junit:junit:4.13.2")
}

kotlin {
    jvmToolchain(17)
}

intellijPlatform {
    pluginConfiguration {
        id = "com.example.myplugin"
        name = "My Plugin"
        version = project.version.toString()

        description = """
            Plugin description here.
        """.trimIndent()

        changeNotes = """
            <h3>${project.version}</h3>
            <ul>
                <li>Initial release</li>
            </ul>
        """.trimIndent()

        ideaVersion {
            sinceBuild = "242"
            untilBuild = "262.*"
        }

        vendor {
            name = "Your Name"
            email = "[email protected]"
            url = "https://example.com"
        }
    }

    signing {
        certificateChain = providers.environmentVariable("CERTIFICATE_CHAIN")
        privateKey = providers.environmentVariable("PRIVATE_KEY")
        password = providers.environmentVariable("PRIVATE_KEY_PASSWORD")
    }

    publishing {
        token = providers.environmentVariable("PUBLISH_TOKEN")
        channels = listOf("stable")
        // channels = listOf("beta")
        // channels = listOf("eap")
    }

    pluginVerification {
        ides {
            recommended()
            // Or specify exact versions:
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.1")
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.2")
        }
    }
}

// Disable searchable options (speeds up build)
tasks {
    buildSearchableOptions {
        enabled = false
    }
}
```

### settings.gradle.kts

```kotlin
plugins {
    id("org.gradle.toolchains.foojay-resolver-convention") version "0.8.0"
}

rootProject.name = "my-plugin"
```

### gradle.properties

```properties
# Plugin metadata
pluginVersion=1.0.0
pluginSinceBuild=242
pluginUntilBuild=262.*

# IDE version for development
platformVersion=2024.2

# Kotlin
kotlin.stdlib.default.dependency=false
kotlin.incremental.useClasspathSnapshot=false

# Gradle
org.gradle.caching=true
org.gradle.parallel=true
```

### gradle/libs.versions.toml (Version Catalog)

```toml
[versions]
kotlin = "2.0.21"
intellij-platform = "2.1.0"
junit = "4.13.2"

[plugins]
kotlin = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
intellij-platform = { id = "org.jetbrains.intellij.platform", version.ref = "intellij-platform" }

[libraries]
junit = { group = "junit", name = "junit", version.ref = "junit" }
```

## Common Tasks

### Development

```bash
# Build plugin
./gradlew build

# Run in sandbox IDE
./gradlew runIde

# Run with custom IDE
./gradlew runIde -PplatformType=IC -PplatformVersion=2024.3

# Run with debug
./gradlew runIde --debug-jvm
```

### Testing

```bash
# Run all tests
./gradlew test

# Run specific test
./gradlew test --tests "com.example.MyTest"

# Generate test reports
./gradlew test jacocoTestReport
```

### Verification

```bash
# Verify plugin structure
./gradlew verifyPlugin

# Run plugin verifier against multiple IDEs
./gradlew runPluginVerifier

# Check compatibility with specific IDE
./gradlew runPluginVerifier -PverifierIdeVersions=IC-2024.1,IC-2024.2
```

### Publishing

```bash
# Build distributable ZIP
./gradlew buildPlugin

# Sign the plugin
./gradlew signPlugin

# Publish to JetBrains Marketplace
./gradlew publishPlugin
```

## IDE Types

| Type Code | IDE |
|-----------|-----|
| `IC` | IntelliJ IDEA Community |
| `IU` | IntelliJ IDEA Ultimate |
| `AS` | Android Studio |
| `PC` | PyCharm Community |
| `PY` | PyCharm Professional |
| `WS` | WebStorm |
| `GO` | GoLand |
| `PS` | PhpStorm |
| `RD` | Rider |
| `CL` | CLion |
| `RM` | RubyMine |

## Bundled Plugins Reference

Common bundled plugin IDs:
| Plugin | ID |
|--------|-----|
| Java | `com.intellij.java` |
| Kotlin | `org.jetbrains.kotlin` |
| Git | `Git4Idea` |
| Gradle | `com.intellij.gradle` |
| Maven | `org.jetbrains.idea.maven` |
| Terminal | `org.jetbrains.plugins.terminal` |
| Markdown | `org.intellij.plugins.markdown` |

## Multi-Platform Support

To support multiple JetBrains IDEs:

```kotlin
dependencies {
    intellijPlatform {
        // Use local IDE for development
        local(providers.gradleProperty("localIde").orNull ?: "/path/to/ide")

        // Or use common platform modules
        intellijIdeaCommunity("2024.2")

        // Only depend on platform modules
        bundledModule("com.intellij.modules.platform")
        bundledModule("com.intellij.modules.lang")
    }
}
```

## Kotlin Configuration

```kotlin
kotlin {
    jvmToolchain(17)  // Required for 2024.2+

    compilerOptions {
        apiVersion.set(KotlinVersion.KOTLIN_1_9)
        languageVersion.set(KotlinVersion.KOTLIN_1_9)
        freeCompilerArgs.addAll(
            "-Xjvm-default=all",
            "-Xcontext-receivers"
        )
    }
}

// Avoid conflicts with bundled Kotlin
tasks {
    compileKotlin {
        kotlinOptions {
            // Match IDE's bundled Kotlin version
            apiVersion = "1.9"
            languageVersion = "1.9"
        }
    }
}
```

## Custom Run Configurations

```kotlin
intellijPlatform {
    // Custom sandbox directory
    sandboxContainer = layout.buildDirectory.dir("custom-sandbox")

    // Run with specific VM options
    runIde {
        jvmArgs("-Xmx2g", "-XX:+UseG1GC")
        systemProperty("idea.is.internal", "true")
    }
}
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - uses: gradle/actions/setup-gradle@v3
      - run: ./gradlew build
      - run: ./gradlew runPluginVerifier
```

## Troubleshooting

### Common Issues

1. **Kotlin version conflicts**: Ensure your Kotlin version matches IDE's bundled version
2. **Missing dependencies**: Check `bundledPlugin()` declarations
3. **Build fails on 2024.2+**: Update to Gradle IntelliJ Plugin 2.x
4. **Test failures**: Ensure test fixtures use `BasePlatformTestCase`

### Gradle Debug

```bash
# Show dependency tree
./gradlew dependencies --configuration runtimeClasspath

# Build with stacktrace
./gradlew build --stacktrace

# Refresh dependencies
./gradlew build --refresh-dependencies
```
