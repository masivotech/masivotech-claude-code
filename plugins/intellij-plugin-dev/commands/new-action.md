---
description: Scaffold a new AnAction for IntelliJ plugin development
---

# Scaffold IntelliJ Action

Create a new AnAction class for the IntelliJ plugin in this project.

## Instructions

1. Ask the user for the following information if not provided:
   - Action name (e.g., "MyCustomAction")
   - Action text (display text in menus)
   - Action description
   - Target menu group (e.g., EditorPopupMenu, ToolsMenu, GenerateGroup)
   - Keyboard shortcut (optional)

2. Determine the project's package structure by examining existing source files.

3. Create the Action class with the following template:

```kotlin
package <detected.package>.actions

import com.intellij.openapi.actionSystem.ActionUpdateThread
import com.intellij.openapi.actionSystem.AnAction
import com.intellij.openapi.actionSystem.AnActionEvent
import com.intellij.openapi.actionSystem.CommonDataKeys
import com.intellij.openapi.ui.Messages

class <ActionName> : AnAction() {
    override fun actionPerformed(e: AnActionEvent) {
        val project = e.project ?: return
        val editor = e.getData(CommonDataKeys.EDITOR)
        val file = e.getData(CommonDataKeys.VIRTUAL_FILE)

        // TODO: Implement action logic
        Messages.showInfoMessage(project, "Action executed!", "<ActionText>")
    }

    override fun update(e: AnActionEvent) {
        // Enable action only when a project is open
        e.presentation.isEnabledAndVisible = e.project != null
    }

    override fun getActionUpdateThread(): ActionUpdateThread {
        return ActionUpdateThread.BGT
    }
}
```

4. Register the action in plugin.xml:

```xml
<actions>
    <action id="<PluginId>.<ActionName>"
            class="<package>.actions.<ActionName>"
            text="<ActionText>"
            description="<ActionDescription>">
        <add-to-group group-id="<TargetGroup>" anchor="last"/>
        <!-- Optional keyboard shortcut -->
        <keyboard-shortcut keymap="$default" first-keystroke="<shortcut>"/>
    </action>
</actions>
```

5. Provide the user with:
   - The file path where the action was created
   - The plugin.xml registration snippet
   - Instructions to test: `./gradlew runIde`

## Common Action Groups

| Group ID | Location |
|----------|----------|
| `MainMenu` | Main menu bar |
| `EditorPopupMenu` | Editor right-click menu |
| `ProjectViewPopupMenu` | Project view right-click |
| `ToolsMenu` | Tools menu |
| `RefactoringMenu` | Refactor menu |
| `GenerateGroup` | Generate menu (Alt+Insert) |
| `Vcs.MessageActionGroup` | Commit message toolbar |
