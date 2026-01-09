---
description: Scaffold a new Tool Window for IntelliJ plugin development
---

# Scaffold IntelliJ Tool Window

Create a new Tool Window for the IntelliJ plugin in this project.

## Instructions

1. Ask the user for the following information if not provided:
   - Tool window ID (e.g., "MyToolWindow")
   - Display name/title
   - Anchor position: left, right, or bottom
   - Icon (optional, defaults to AllIcons.General.Information)
   - Should include toolbar? (yes/no)

2. Determine the project's package structure by examining existing source files.

3. Create the ToolWindowFactory class:

```kotlin
package <detected.package>.toolwindow

import com.intellij.openapi.project.Project
import com.intellij.openapi.wm.ToolWindow
import com.intellij.openapi.wm.ToolWindowFactory
import com.intellij.ui.content.ContentFactory

class <ToolWindowId>Factory : ToolWindowFactory {
    override fun createToolWindowContent(project: Project, toolWindow: ToolWindow) {
        val panel = <ToolWindowId>Panel(project)
        val contentFactory = ContentFactory.getInstance()
        val content = contentFactory.createContent(panel, "<DisplayName>", false)
        toolWindow.contentManager.addContent(content)
    }

    override fun shouldBeAvailable(project: Project): Boolean {
        // Return true to always show, or add conditions
        return true
    }

    override fun init(toolWindow: ToolWindow) {
        toolWindow.stripeTitle = "<DisplayName>"
    }
}
```

4. Create the Panel class:

### Basic Panel (without toolbar)
```kotlin
package <detected.package>.toolwindow

import com.intellij.openapi.project.Project
import com.intellij.ui.components.JBLabel
import com.intellij.ui.components.JBPanel
import java.awt.BorderLayout

class <ToolWindowId>Panel(private val project: Project) : JBPanel<JBPanel<*>>(BorderLayout()) {
    init {
        val label = JBLabel("Hello from <DisplayName>!")
        add(label, BorderLayout.CENTER)
    }
}
```

### Panel with Toolbar
```kotlin
package <detected.package>.toolwindow

import com.intellij.openapi.actionSystem.ActionManager
import com.intellij.openapi.actionSystem.AnAction
import com.intellij.openapi.actionSystem.AnActionEvent
import com.intellij.openapi.actionSystem.DefaultActionGroup
import com.intellij.openapi.project.Project
import com.intellij.openapi.ui.SimpleToolWindowPanel
import com.intellij.icons.AllIcons
import com.intellij.ui.components.JBLabel
import com.intellij.ui.components.JBPanel
import java.awt.BorderLayout

class <ToolWindowId>Panel(private val project: Project) : SimpleToolWindowPanel(true, true) {
    init {
        // Create toolbar
        val actionGroup = DefaultActionGroup().apply {
            add(RefreshAction())
            addSeparator()
            add(SettingsAction())
        }

        val actionManager = ActionManager.getInstance()
        val toolbar = actionManager.createActionToolbar(
            "<ToolWindowId>Toolbar",
            actionGroup,
            true
        )
        toolbar.targetComponent = this
        setToolbar(toolbar.component)

        // Create content
        val content = JBPanel<JBPanel<*>>(BorderLayout()).apply {
            add(JBLabel("Hello from <DisplayName>!"), BorderLayout.CENTER)
        }
        setContent(content)
    }

    private inner class RefreshAction : AnAction("Refresh", "Refresh content", AllIcons.Actions.Refresh) {
        override fun actionPerformed(e: AnActionEvent) {
            // TODO: Implement refresh logic
        }
    }

    private inner class SettingsAction : AnAction("Settings", "Open settings", AllIcons.General.Settings) {
        override fun actionPerformed(e: AnActionEvent) {
            // TODO: Implement settings logic
        }
    }
}
```

5. Register the tool window in plugin.xml:

```xml
<extensions defaultExtensionNs="com.intellij">
    <toolWindow id="<ToolWindowId>"
                anchor="<anchor>"
                secondary="false"
                icon="AllIcons.General.Information"
                factoryClass="<package>.toolwindow.<ToolWindowId>Factory"/>
</extensions>
```

6. Provide the user with:
   - The file paths where files were created
   - The plugin.xml registration snippet
   - Instructions to test: `./gradlew runIde`
   - How to access: View → Tool Windows → <DisplayName>

## Programmatic Access

```kotlin
// Get tool window
val toolWindow = ToolWindowManager.getInstance(project)
    .getToolWindow("<ToolWindowId>")

// Show/hide
toolWindow?.show()
toolWindow?.hide()

// Activate (bring to front)
toolWindow?.activate(null)
```

## Anchor Positions

| Anchor | Location |
|--------|----------|
| `left` | Left side panel |
| `right` | Right side panel |
| `bottom` | Bottom panel |
