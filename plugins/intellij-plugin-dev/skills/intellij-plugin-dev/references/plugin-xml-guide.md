# plugin.xml Complete Reference

The `plugin.xml` file is the plugin descriptor that defines your plugin's metadata, dependencies, and extension points.

## File Location

```
src/main/resources/META-INF/plugin.xml
```

## Complete Structure

```xml
<idea-plugin>
    <!-- Basic Information -->
    <id>com.example.myplugin</id>
    <name>My Plugin Name</name>
    <version>1.0.0</version>
    <vendor email="[email protected]" url="https://example.com">Company Name</vendor>

    <!-- Description (HTML supported) -->
    <description><![CDATA[
        <p>Plugin description paragraph.</p>
        <ul>
            <li>Feature 1</li>
            <li>Feature 2</li>
        </ul>
    ]]></description>

    <!-- Change Notes (HTML supported) -->
    <change-notes><![CDATA[
        <h3>1.0.0</h3>
        <ul>
            <li>Initial release</li>
        </ul>
    ]]></change-notes>

    <!-- IDE Compatibility -->
    <idea-version since-build="242" until-build="262.*"/>

    <!-- Dependencies -->
    <depends>com.intellij.modules.platform</depends>
    <depends>com.intellij.modules.lang</depends>
    <depends optional="true" config-file="optional-feature.xml">Git4Idea</depends>

    <!-- Extensions -->
    <extensions defaultExtensionNs="com.intellij">
        <!-- Your extensions here -->
    </extensions>

    <!-- Actions -->
    <actions>
        <!-- Your actions here -->
    </actions>

    <!-- Listeners -->
    <applicationListeners>
        <!-- Application-level listeners -->
    </applicationListeners>

    <projectListeners>
        <!-- Project-level listeners -->
    </projectListeners>
</idea-plugin>
```

## Element Reference

### `<id>`
Unique identifier for the plugin. Use reverse domain notation.

```xml
<id>com.example.myplugin</id>
```

**Best Practice**: Match your package name structure.

### `<name>`
Display name shown in the Marketplace and settings.

```xml
<name>My Awesome Plugin</name>
```

### `<version>`
Plugin version. Use semantic versioning.

```xml
<version>1.2.3</version>
```

**Note**: Often patched by Gradle from `build.gradle.kts`.

### `<vendor>`
Publisher information.

```xml
<vendor email="[email protected]" url="https://example.com">
    Company Name
</vendor>
```

### `<description>`
Plugin description for Marketplace. Supports HTML in CDATA.

```xml
<description><![CDATA[
    <p><b>My Plugin</b> provides amazing features:</p>
    <ul>
        <li>Feature A - Does something cool</li>
        <li>Feature B - Does something cooler</li>
    </ul>
    <p>Visit <a href="https://example.com">our website</a> for docs.</p>
]]></description>
```

### `<change-notes>`
Release notes for the current version.

```xml
<change-notes><![CDATA[
    <h3>Version 1.2.0</h3>
    <ul>
        <li>Added new feature X</li>
        <li>Fixed bug in Y</li>
    </ul>
]]></change-notes>
```

### `<idea-version>`
IDE version compatibility range.

```xml
<idea-version since-build="242" until-build="262.*"/>
```

Build number format: `BRANCH.BUILD.FIX`
- `242` = 2024.2
- `243` = 2024.3
- `251` = 2025.1

**Wildcards**: Use `*` for open-ended support: `until-build="262.*"`

### `<depends>`
Plugin and module dependencies.

**Required Platform Modules**:
```xml
<!-- Core platform (always required) -->
<depends>com.intellij.modules.platform</depends>

<!-- Language support (PSI, syntax highlighting) -->
<depends>com.intellij.modules.lang</depends>

<!-- VCS integration -->
<depends>com.intellij.modules.vcs</depends>

<!-- Java support -->
<depends>com.intellij.modules.java</depends>

<!-- Ultimate-only features -->
<depends>com.intellij.modules.ultimate</depends>
```

**Bundled Plugin Dependencies**:
```xml
<depends>Git4Idea</depends>
<depends>org.jetbrains.kotlin</depends>
<depends>com.intellij.gradle</depends>
```

**Optional Dependencies**:
```xml
<depends optional="true" config-file="kotlin-features.xml">
    org.jetbrains.kotlin
</depends>
```

When optional dependency is present, `kotlin-features.xml` is loaded.

### `<extensions>`
Register extension point implementations.

```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationService
        serviceImplementation="com.example.MyAppService"/>

    <projectService
        serviceImplementation="com.example.MyProjectService"/>

    <toolWindow id="MyTool" anchor="right"
        factoryClass="com.example.MyToolWindowFactory"/>

    <notificationGroup id="MyPlugin.Notifications"
        displayType="BALLOON"/>
</extensions>
```

### `<actions>`
Register menu items, toolbar buttons, and keyboard shortcuts.

```xml
<actions>
    <!-- Single action -->
    <action id="MyPlugin.MyAction"
            class="com.example.actions.MyAction"
            text="My Action"
            description="Does something useful"
            icon="AllIcons.Actions.Execute">
        <add-to-group group-id="ToolsMenu" anchor="last"/>
        <keyboard-shortcut keymap="$default"
            first-keystroke="ctrl alt M"/>
    </action>

    <!-- Action group (submenu) -->
    <group id="MyPlugin.MyGroup"
           text="My Plugin"
           description="My Plugin Actions"
           popup="true"
           icon="AllIcons.General.Settings">
        <add-to-group group-id="MainMenu" anchor="after"
            relative-to-action="ToolsMenu"/>
        <action id="MyPlugin.Action1" class="com.example.Action1"
                text="Action 1"/>
        <action id="MyPlugin.Action2" class="com.example.Action2"
                text="Action 2"/>
        <separator/>
        <action id="MyPlugin.Action3" class="com.example.Action3"
                text="Action 3"/>
    </group>
</actions>
```

**Anchor Values**: `first`, `last`, `before`, `after`

**Common Keystrokes**:
- `ctrl`, `alt`, `shift`, `meta` (Cmd on Mac)
- `first-keystroke` and `second-keystroke` for chords

### `<applicationListeners>`
Application-level event listeners.

```xml
<applicationListeners>
    <listener class="com.example.MyAppListener"
              topic="com.intellij.openapi.project.ProjectManagerListener"/>
</applicationListeners>
```

### `<projectListeners>`
Project-level event listeners.

```xml
<projectListeners>
    <listener class="com.example.MyFileListener"
              topic="com.intellij.openapi.vfs.VirtualFileListener"/>
</projectListeners>
```

## Optional Dependency Files

When using optional dependencies, create separate XML files:

**plugin.xml**:
```xml
<depends optional="true" config-file="git-features.xml">Git4Idea</depends>
```

**META-INF/git-features.xml**:
```xml
<idea-plugin>
    <extensions defaultExtensionNs="com.intellij">
        <checkinHandlerFactory
            implementation="com.example.git.MyGitCheckinHandler"/>
    </extensions>
</idea-plugin>
```

## Gradle Patching

Many values can be patched by Gradle at build time:

**build.gradle.kts**:
```kotlin
intellijPlatform {
    pluginConfiguration {
        id = "com.example.myplugin"
        name = "My Plugin"
        version = project.version.toString()

        ideaVersion {
            sinceBuild = "242"
            untilBuild = "262.*"
        }

        description = providers.fileContents(
            layout.projectDirectory.file("README.md")
        ).asText

        changeNotes = """
            <h3>${project.version}</h3>
            <ul>
                <li>New features</li>
            </ul>
        """.trimIndent()
    }
}
```

## Validation

Check your plugin.xml with:
```bash
./gradlew verifyPlugin
./gradlew runPluginVerifier
```

## Common Mistakes

1. **Missing required dependency**: Always include `com.intellij.modules.platform`
2. **Wrong build numbers**: Use branch numbers (242, 243), not version strings
3. **Hardcoded version**: Let Gradle patch version from `gradle.properties`
4. **Missing CDATA**: Use `<![CDATA[...]]>` for HTML in description/change-notes
5. **Wrong extension namespace**: Default is `com.intellij`, specify if different
