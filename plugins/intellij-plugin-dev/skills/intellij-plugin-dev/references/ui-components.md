# UI Components Reference

This reference covers common UI patterns for IntelliJ Platform plugin development.

## Tool Windows

### Basic Tool Window Factory
```kotlin
class MyToolWindowFactory : ToolWindowFactory {
    override fun createToolWindowContent(project: Project, toolWindow: ToolWindow) {
        val panel = MyToolWindowPanel(project)
        val contentFactory = ContentFactory.getInstance()
        val content = contentFactory.createContent(panel, "My Tool", false)
        toolWindow.contentManager.addContent(content)
    }

    override fun shouldBeAvailable(project: Project): Boolean {
        // Conditionally show tool window
        return true
    }

    override fun init(toolWindow: ToolWindow) {
        // Configure tool window
        toolWindow.stripeTitle = "My Tool"
    }
}
```

### Tool Window Panel
```kotlin
class MyToolWindowPanel(private val project: Project) : SimpleToolWindowPanel(true, true) {
    init {
        // Toolbar
        val actionManager = ActionManager.getInstance()
        val actionGroup = DefaultActionGroup().apply {
            add(RefreshAction())
            addSeparator()
            add(SettingsAction())
        }
        val toolbar = actionManager.createActionToolbar(
            "MyToolWindow",
            actionGroup,
            true
        )
        toolbar.targetComponent = this
        setToolbar(toolbar.component)

        // Content
        val content = JBPanel<JBPanel<*>>(BorderLayout()).apply {
            add(JBLabel("Content here"), BorderLayout.CENTER)
        }
        setContent(content)
    }
}
```

### Registration
```xml
<extensions defaultExtensionNs="com.intellij">
    <toolWindow id="MyToolWindow"
                anchor="right"
                secondary="false"
                icon="AllIcons.General.Information"
                factoryClass="com.example.MyToolWindowFactory"/>
</extensions>
```

### Programmatic Access
```kotlin
// Get tool window
val toolWindow = ToolWindowManager.getInstance(project)
    .getToolWindow("MyToolWindow")

// Show/hide
toolWindow?.show()
toolWindow?.hide()

// Activate
toolWindow?.activate(null)
```

## Dialogs

### DialogWrapper (Custom Dialogs)
```kotlin
class MyDialog(project: Project?) : DialogWrapper(project) {
    private val nameField = JBTextField()
    private val descriptionArea = JBTextArea(3, 30)

    init {
        title = "My Dialog"
        init()
    }

    override fun createCenterPanel(): JComponent {
        return panel {
            row("Name:") {
                cell(nameField)
                    .columns(COLUMNS_MEDIUM)
                    .focused()
            }
            row("Description:") {
                cell(JBScrollPane(descriptionArea))
                    .align(AlignX.FILL)
            }
        }
    }

    override fun doOKAction() {
        // Validate and process
        if (nameField.text.isBlank()) {
            setErrorText("Name is required", nameField)
            return
        }
        super.doOKAction()
    }

    fun getName(): String = nameField.text
    fun getDescription(): String = descriptionArea.text
}

// Usage
val dialog = MyDialog(project)
if (dialog.showAndGet()) {
    val name = dialog.getName()
    // Process result
}
```

### Simple Message Dialogs
```kotlin
// Information
Messages.showInfoMessage(project, "Message", "Title")

// Warning
Messages.showWarningDialog(project, "Warning message", "Warning")

// Error
Messages.showErrorDialog(project, "Error message", "Error")

// Yes/No
val result = Messages.showYesNoDialog(
    project,
    "Are you sure?",
    "Confirm",
    Messages.getQuestionIcon()
)
if (result == Messages.YES) {
    // User clicked Yes
}

// Yes/No/Cancel
val result = Messages.showYesNoCancelDialog(
    project,
    "Save changes?",
    "Confirm",
    "Save",
    "Don't Save",
    "Cancel",
    Messages.getQuestionIcon()
)

// Input dialog
val input = Messages.showInputDialog(
    project,
    "Enter name:",
    "Input Required",
    Messages.getQuestionIcon()
)
```

### Chooser Dialogs
```kotlin
// List chooser
val items = listOf("Option 1", "Option 2", "Option 3")
val selected = Messages.showChooseDialog(
    project,
    "Select option:",
    "Choose",
    Messages.getQuestionIcon(),
    items.toTypedArray(),
    items[0]
)

// File chooser
val descriptor = FileChooserDescriptorFactory.createSingleFileDescriptor()
val file = FileChooser.chooseFile(descriptor, project, null)
```

## Notifications

### Balloon Notifications
```kotlin
// Define notification group in plugin.xml
// <notificationGroup id="MyPlugin.Notifications" displayType="BALLOON"/>

val notificationGroup = NotificationGroupManager.getInstance()
    .getNotificationGroup("MyPlugin.Notifications")

// Information
notificationGroup.createNotification(
    "Title",
    "Notification message",
    NotificationType.INFORMATION
).notify(project)

// With actions
notificationGroup.createNotification(
    "Title",
    "Message with action",
    NotificationType.INFORMATION
).addAction(object : AnAction("Open Settings") {
    override fun actionPerformed(e: AnActionEvent) {
        ShowSettingsUtil.getInstance()
            .showSettingsDialog(project, "MySettings")
    }
}).notify(project)

// Sticky (doesn't auto-dismiss)
notificationGroup.createNotification(
    "Important",
    "This stays visible",
    NotificationType.WARNING
).setImportant(true).notify(project)
```

### Tool Window Notifications
```xml
<notificationGroup id="MyPlugin.ToolWindow"
                   displayType="TOOL_WINDOW"
                   toolWindowId="MyToolWindow"/>
```

### Status Bar
```kotlin
// Simple message
StatusBar.Info.set("Processing...", project)

// Clear message
StatusBar.Info.set("", project)
```

## Kotlin UI DSL

### Basic Form
```kotlin
panel {
    row("Label:") {
        textField()
            .columns(COLUMNS_MEDIUM)
            .bindText(settings::property)
    }
    row("Number:") {
        intTextField(0..100)
            .bindIntText(settings::count)
    }
    row("Checkbox:") {
        checkBox("Enable feature")
            .bindSelected(settings::enabled)
    }
    row("Combo:") {
        comboBox(listOf("A", "B", "C"))
            .bindItem(settings::selection)
    }
}
```

### Groups and Sections
```kotlin
panel {
    group("General") {
        row("Name:") {
            textField()
        }
    }

    collapsibleGroup("Advanced") {
        row("Option:") {
            checkBox("Enable")
        }
    }

    separator()

    row {
        comment("This is a helpful comment")
    }
}
```

### Layout Controls
```kotlin
panel {
    row {
        label("Left")
        label("Right").align(AlignX.RIGHT)
    }
    row {
        textField()
            .align(AlignX.FILL)
            .resizableColumn()
    }
    row {
        button("Action") {
            // Handle click
        }
    }
}
```

## Popups

### List Popup
```kotlin
val items = listOf("Item 1", "Item 2", "Item 3")
val popup = JBPopupFactory.getInstance().createListPopup(
    object : BaseListPopupStep<String>("Title", items) {
        override fun onChosen(selectedValue: String, finalChoice: Boolean): PopupStep<*>? {
            // Handle selection
            return FINAL_CHOICE
        }
    }
)
popup.showInBestPositionFor(editor)
```

### Action Popup
```kotlin
val actionGroup = DefaultActionGroup().apply {
    add(object : AnAction("Action 1") {
        override fun actionPerformed(e: AnActionEvent) {}
    })
    add(object : AnAction("Action 2") {
        override fun actionPerformed(e: AnActionEvent) {}
    })
}

val popup = JBPopupFactory.getInstance().createActionGroupPopup(
    "Title",
    actionGroup,
    dataContext,
    JBPopupFactory.ActionSelectionAid.SPEEDSEARCH,
    false
)
popup.showInBestPositionFor(dataContext)
```

### Hint Popup
```kotlin
val hint = JBPopupFactory.getInstance().createHtmlTextBalloonBuilder(
    "<b>Title</b><br>Content here",
    MessageType.INFO,
    null
).createBalloon()

hint.show(
    RelativePoint.getCenterOf(component),
    Balloon.Position.above
)
```

## Progress Indicators

### Background Task with Progress
```kotlin
object : Task.Backgroundable(project, "Processing...", true) {
    override fun run(indicator: ProgressIndicator) {
        indicator.text = "Starting..."
        indicator.isIndeterminate = false

        for (i in 0..100) {
            indicator.checkCanceled()
            indicator.fraction = i / 100.0
            indicator.text = "Processing item $i"
            Thread.sleep(50)
        }
    }

    override fun onSuccess() {
        // Task completed
    }

    override fun onCancel() {
        // Task was cancelled
    }
}.queue()
```

### Modal Progress
```kotlin
ProgressManager.getInstance().runProcessWithProgressSynchronously(
    {
        val indicator = ProgressManager.getInstance().progressIndicator
        indicator.text = "Working..."
        // Do work
    },
    "Processing",
    true,  // cancellable
    project
)
```

## Tables and Lists

### JBTable
```kotlin
val model = object : AbstractTableModel() {
    private val data = mutableListOf<MyData>()

    override fun getRowCount() = data.size
    override fun getColumnCount() = 3
    override fun getValueAt(row: Int, col: Int): Any = when (col) {
        0 -> data[row].name
        1 -> data[row].value
        2 -> data[row].enabled
        else -> ""
    }
    override fun getColumnName(col: Int) = when (col) {
        0 -> "Name"
        1 -> "Value"
        2 -> "Enabled"
        else -> ""
    }
}

val table = JBTable(model).apply {
    setShowGrid(false)
    intercellSpacing = Dimension(0, 0)
}
```

### JBList
```kotlin
val listModel = CollectionListModel(items)
val list = JBList(listModel).apply {
    cellRenderer = object : ColoredListCellRenderer<MyItem>() {
        override fun customizeCellRenderer(
            list: JList<out MyItem>,
            value: MyItem,
            index: Int,
            selected: Boolean,
            hasFocus: Boolean
        ) {
            append(value.name)
            append(" - ${value.description}", SimpleTextAttributes.GRAYED_ATTRIBUTES)
        }
    }
}
```

## Icons

### Using Built-in Icons
```kotlin
// AllIcons class has all platform icons
val icon = AllIcons.Actions.Execute
val settingsIcon = AllIcons.General.Settings
val errorIcon = AllIcons.General.Error
```

### Custom Icons
```kotlin
// Load from resources
val icon = IconLoader.getIcon("/icons/myicon.svg", javaClass)

// In plugin.xml
icon="com.example.icons.MyIcons.TOOL_ICON"
```

### Icon Provider Class
```kotlin
object MyIcons {
    @JvmField
    val TOOL_ICON = IconLoader.getIcon("/icons/tool.svg", MyIcons::class.java)

    @JvmField
    val ACTION_ICON = IconLoader.getIcon("/icons/action.svg", MyIcons::class.java)
}
```
