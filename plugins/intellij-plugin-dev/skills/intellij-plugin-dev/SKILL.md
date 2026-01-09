---
name: intellij-plugin-dev
description: |
  Comprehensive guide for JetBrains/IntelliJ Platform Plugin SDK development.
  Use this skill when: (1) Creating new IntelliJ IDEA, Android Studio, or other JetBrains IDE plugins,
  (2) Working with plugin.xml configuration and extension points, (3) Configuring Gradle IntelliJ Plugin 2.x,
  (4) Implementing Actions, Services, Tool Windows, Inspections, or other extensions,
  (5) Working with PSI (Program Structure Interface) for code analysis and manipulation,
  (6) Working with VFS (Virtual File System) for file operations,
  (7) Testing plugins with runIde, runPluginVerifier, or writing unit/UI tests,
  (8) Publishing plugins to JetBrains Marketplace with signing and versioning,
  (9) Debugging plugin compatibility issues across IDE versions,
  (10) Understanding IntelliJ Platform architecture and best practices.
---

# IntelliJ Platform Plugin Development Guide

This skill provides comprehensive guidance for developing plugins for JetBrains IDEs including IntelliJ IDEA, Android Studio, PyCharm, WebStorm, GoLand, PhpStorm, RubyMine, CLion, and Rider.

## Quick Reference

### Essential Gradle Commands
```bash
./gradlew build              # Build the plugin
./gradlew runIde             # Run in sandbox IDE
./gradlew runPluginVerifier  # Verify compatibility
./gradlew test               # Run unit tests
./gradlew buildPlugin        # Create distributable ZIP
./gradlew publishPlugin      # Publish to Marketplace
```

### Project Structure
```
plugin-name/
├── build.gradle.kts          # Gradle build configuration
├── settings.gradle.kts       # Gradle settings
├── gradle.properties         # Version properties
├── gradle/
│   └── libs.versions.toml    # Version catalog (recommended)
└── src/
    └── main/
        ├── kotlin/           # Kotlin source files
        │   └── com/example/plugin/
        └── resources/
            └── META-INF/
                └── plugin.xml  # Plugin descriptor
```

## Plugin Configuration

### build.gradle.kts (Gradle IntelliJ Plugin 2.x)

For target IDE version 2024.2+, use IntelliJ Platform Gradle Plugin 2.x:

```kotlin
plugins {
    id("java")
    id("org.jetbrains.kotlin.jvm") version "2.0.21"
    id("org.jetbrains.intellij.platform") version "2.1.0"
}

repositories {
    mavenCentral()
    intellijPlatform {
        defaultRepositories()
    }
}

dependencies {
    intellijPlatform {
        intellijIdeaCommunity("2024.2")
        // Or for Android Studio:
        // androidStudio("2024.2.1")

        bundledPlugin("com.intellij.java")
        bundledPlugin("Git4Idea")

        pluginVerifier()
        zipSigner()
        instrumentationTools()
    }
}

intellijPlatform {
    pluginConfiguration {
        id = "com.example.myplugin"
        name = "My Plugin"
        version = "1.0.0"

        ideaVersion {
            sinceBuild = "242"
            untilBuild = "262.*"
        }
    }

    signing {
        certificateChain = providers.environmentVariable("CERTIFICATE_CHAIN")
        privateKey = providers.environmentVariable("PRIVATE_KEY")
        password = providers.environmentVariable("PRIVATE_KEY_PASSWORD")
    }

    publishing {
        token = providers.environmentVariable("PUBLISH_TOKEN")
    }
}
```

### plugin.xml Structure

```xml
<idea-plugin>
    <id>com.example.myplugin</id>
    <name>My Plugin</name>
    <vendor email="[email protected]" url="https://example.com">Example</vendor>
    <description><![CDATA[
        Plugin description in HTML format.
    ]]></description>

    <!-- Plugin dependencies -->
    <depends>com.intellij.modules.platform</depends>
    <depends>com.intellij.modules.vcs</depends>
    <depends optional="true" config-file="git-features.xml">Git4Idea</depends>

    <!-- Extension points -->
    <extensions defaultExtensionNs="com.intellij">
        <!-- Actions, services, tool windows, etc. -->
    </extensions>

    <!-- Actions -->
    <actions>
        <action id="MyAction" class="com.example.MyAction" text="My Action">
            <add-to-group group-id="EditorPopupMenu" anchor="last"/>
            <keyboard-shortcut keymap="$default" first-keystroke="ctrl alt M"/>
        </action>
    </actions>
</idea-plugin>
```

## Core Extension Points

### Actions (AnAction)

Actions are the primary way to add menu items, toolbar buttons, and keyboard shortcuts:

```kotlin
class MyAction : AnAction() {
    override fun actionPerformed(e: AnActionEvent) {
        val project = e.project ?: return
        val editor = e.getData(CommonDataKeys.EDITOR) ?: return
        val file = e.getData(CommonDataKeys.VIRTUAL_FILE)

        // Perform action
        Messages.showInfoMessage(project, "Hello!", "My Plugin")
    }

    override fun update(e: AnActionEvent) {
        // Enable/disable action based on context
        e.presentation.isEnabledAndVisible = e.project != null
    }

    override fun getActionUpdateThread(): ActionUpdateThread {
        return ActionUpdateThread.BGT  // Background thread for update()
    }
}
```

Register in plugin.xml:
```xml
<actions>
    <action id="MyAction" class="com.example.MyAction"
            text="My Action" description="Does something useful">
        <add-to-group group-id="ToolsMenu" anchor="last"/>
    </action>
</actions>
```

### Services

Services provide singleton instances scoped to application, project, or module:

**Application Service** (global singleton):
```kotlin
@Service
class MyApplicationService {
    fun doSomething() { /* ... */ }

    companion object {
        fun getInstance(): MyApplicationService = service()
    }
}
```

**Project Service** (one per project):
```kotlin
@Service(Service.Level.PROJECT)
class MyProjectService(private val project: Project) {
    fun doSomething() { /* ... */ }

    companion object {
        fun getInstance(project: Project): MyProjectService = project.service()
    }
}
```

Register in plugin.xml:
```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationService serviceImplementation="com.example.MyApplicationService"/>
    <projectService serviceImplementation="com.example.MyProjectService"/>
</extensions>
```

### Tool Windows

Tool windows appear in the IDE's side panels:

```kotlin
class MyToolWindowFactory : ToolWindowFactory {
    override fun createToolWindowContent(project: Project, toolWindow: ToolWindow) {
        val contentFactory = ContentFactory.getInstance()
        val panel = MyToolWindowPanel(project)
        val content = contentFactory.createContent(panel, "My Tool", false)
        toolWindow.contentManager.addContent(content)
    }

    override fun shouldBeAvailable(project: Project): Boolean {
        return true  // Conditionally show tool window
    }
}

class MyToolWindowPanel(private val project: Project) : JPanel(BorderLayout()) {
    init {
        val label = JBLabel("Hello from tool window!")
        add(label, BorderLayout.CENTER)
    }
}
```

Register in plugin.xml:
```xml
<extensions defaultExtensionNs="com.intellij">
    <toolWindow id="MyToolWindow"
                anchor="right"
                factoryClass="com.example.MyToolWindowFactory"
                icon="AllIcons.General.Information"/>
</extensions>
```

### Checkin Handlers (VCS Integration)

For commit dialog customization:

```kotlin
class MyCheckinHandlerFactory : CheckinHandlerFactory() {
    override fun createHandler(panel: CheckinProjectPanel,
                               commitContext: CommitContext): CheckinHandler {
        return MyCheckinHandler(panel)
    }
}

class MyCheckinHandler(private val panel: CheckinProjectPanel) : CheckinHandler() {
    override fun getBeforeCheckinConfigurationPanel(): RefreshableOnComponent? {
        return object : RefreshableOnComponent {
            override fun getComponent(): JComponent {
                return JPanel().apply {
                    add(JBLabel("Custom commit UI"))
                }
            }
            override fun refresh() {}
            override fun saveState() {}
            override fun restoreState() {}
        }
    }

    override fun beforeCheckin(): ReturnResult {
        // Validate before commit
        return ReturnResult.COMMIT
    }
}
```

Register in plugin.xml:
```xml
<extensions defaultExtensionNs="com.intellij">
    <checkinHandlerFactory implementation="com.example.MyCheckinHandlerFactory"/>
</extensions>
```

## Working with PSI (Program Structure Interface)

PSI provides a structured representation of source code. For detailed patterns, see `references/psi-patterns.md`.

### Basic PSI Navigation

```kotlin
// Get PSI file from editor
val psiFile = PsiManager.getInstance(project).findFile(virtualFile)

// Find elements at caret
val element = psiFile?.findElementAt(editor.caretModel.offset)

// Navigate to parent class
val psiClass = PsiTreeUtil.getParentOfType(element, PsiClass::class.java)

// Find all methods in a class
val methods = PsiTreeUtil.findChildrenOfType(psiClass, PsiMethod::class.java)
```

### PSI Modifications

Always wrap PSI modifications in a write action:

```kotlin
WriteCommandAction.runWriteCommandAction(project) {
    val factory = PsiElementFactory.getInstance(project)
    val newMethod = factory.createMethodFromText(
        "public void newMethod() {}", psiClass
    )
    psiClass.add(newMethod)
}
```

## Working with VFS (Virtual File System)

VFS provides an abstraction layer over file systems. For detailed patterns, see `references/vfs-patterns.md`.

```kotlin
// Get virtual file
val virtualFile = LocalFileSystem.getInstance()
    .findFileByPath("/path/to/file.kt")

// Read file content
val content = virtualFile?.let {
    VfsUtil.loadText(it)
}

// Write to file (must be in write action)
ApplicationManager.getApplication().runWriteAction {
    virtualFile?.let {
        VfsUtil.saveText(it, "new content")
    }
}

// Listen for file changes
VirtualFileManager.getInstance().addVirtualFileListener(
    object : VirtualFileListener {
        override fun contentsChanged(event: VirtualFileEvent) {
            // Handle file change
        }
    },
    disposable
)
```

## UI Components

### Notifications

```kotlin
// Balloon notification
NotificationGroupManager.getInstance()
    .getNotificationGroup("MyPlugin.Notifications")
    .createNotification("Title", "Message", NotificationType.INFORMATION)
    .notify(project)

// Status bar message
StatusBar.Info.set("Processing...", project)
```

Register notification group in plugin.xml:
```xml
<extensions defaultExtensionNs="com.intellij">
    <notificationGroup id="MyPlugin.Notifications"
                       displayType="BALLOON"/>
</extensions>
```

### Dialogs

```kotlin
// Simple message dialog
Messages.showMessageDialog(
    project,
    "Message content",
    "Dialog Title",
    Messages.getInformationIcon()
)

// Input dialog
val input = Messages.showInputDialog(
    project,
    "Enter value:",
    "Input Required",
    Messages.getQuestionIcon()
)

// Custom dialog
class MyDialog(project: Project) : DialogWrapper(project) {
    init {
        title = "My Dialog"
        init()
    }

    override fun createCenterPanel(): JComponent {
        return panel {
            row("Name:") {
                textField()
            }
            row("Description:") {
                textArea()
            }
        }
    }
}
```

## Testing

### Unit Tests

```kotlin
class MyServiceTest : BasePlatformTestCase() {
    fun testSomething() {
        val service = MyProjectService.getInstance(project)
        val result = service.doSomething()
        assertEquals("expected", result)
    }
}
```

### UI Tests

```kotlin
class MyActionTest : LightPlatformCodeInsightFixture4TestCase() {
    fun testAction() {
        myFixture.configureByText("Test.kt", "class Test {}")
        myFixture.performEditorAction("MyAction")
        // Assert results
    }
}
```

### Plugin Verifier

Run `./gradlew runPluginVerifier` to check compatibility with multiple IDE versions.

Configure in build.gradle.kts:
```kotlin
intellijPlatform {
    pluginVerification {
        ides {
            recommended()
            // Or specify versions:
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.1")
        }
    }
}
```

## Best Practices

1. **Use Kotlin** - Preferred over Java for new plugins (required for 2025.1+)
2. **Prefer BGT for updates** - Use `ActionUpdateThread.BGT` for action updates
3. **Use Services** - Avoid static state, use services for singletons
4. **Handle Disposal** - Register disposables properly to avoid memory leaks
5. **Use WriteAction** - All PSI/VFS modifications must be in write actions
6. **Test Compatibility** - Use runPluginVerifier before releases
7. **Follow Platform Conventions** - Use IntelliJ Platform APIs over custom implementations

## Bundled References

For comprehensive details, see the reference files in `references/`:
- `extension-points.md` - Complete catalog of extension points
- `plugin-xml-guide.md` - Full plugin.xml reference
- `gradle-config.md` - Gradle IntelliJ Plugin 2.x configuration
- `psi-patterns.md` - PSI navigation and manipulation patterns
- `vfs-patterns.md` - Virtual File System usage patterns
- `ui-components.md` - Tool windows, dialogs, notifications
- `testing-guide.md` - Unit, UI, and integration testing

## External Resources

- [IntelliJ Platform SDK Documentation](https://plugins.jetbrains.com/docs/intellij/welcome.html)
- [IntelliJ Platform Plugin Template](https://github.com/JetBrains/intellij-platform-plugin-template)
- [JetBrains Marketplace](https://plugins.jetbrains.com/)
