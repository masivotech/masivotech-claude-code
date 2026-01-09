# IntelliJ Platform Extension Points Reference

This reference catalogs commonly used extension points for IntelliJ Platform plugin development.

## Actions & Menus

### AnAction
The most common way to add functionality triggered by users.

```xml
<actions>
    <action id="MyAction" class="com.example.MyAction"
            text="My Action" description="Description"
            icon="AllIcons.Actions.Execute">
        <add-to-group group-id="EditorPopupMenu" anchor="first"/>
        <keyboard-shortcut keymap="$default" first-keystroke="ctrl alt M"/>
    </action>
</actions>
```

### Common Action Groups
| Group ID | Location |
|----------|----------|
| `MainMenu` | Main menu bar |
| `EditorPopupMenu` | Editor right-click menu |
| `ProjectViewPopupMenu` | Project view right-click |
| `ToolsMenu` | Tools menu |
| `RefactoringMenu` | Refactor menu |
| `GenerateGroup` | Generate menu (Alt+Insert) |
| `Vcs.MessageActionGroup` | Commit message toolbar |
| `ChangesViewToolbar` | VCS changes toolbar |

## Services

### Application Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationService
        serviceInterface="com.example.MyService"
        serviceImplementation="com.example.MyServiceImpl"/>
</extensions>
```

### Project Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <projectService
        serviceInterface="com.example.MyProjectService"
        serviceImplementation="com.example.MyProjectServiceImpl"/>
</extensions>
```

### Module Service
```xml
<extensions defaultExtensionNs="com.intellij">
    <moduleService
        serviceInterface="com.example.MyModuleService"
        serviceImplementation="com.example.MyModuleServiceImpl"/>
</extensions>
```

## UI Components

### Tool Window
```xml
<extensions defaultExtensionNs="com.intellij">
    <toolWindow id="MyToolWindow"
                anchor="right"
                secondary="false"
                icon="AllIcons.General.Information"
                factoryClass="com.example.MyToolWindowFactory"/>
</extensions>
```

Anchor values: `left`, `right`, `bottom`

### Status Bar Widget
```xml
<extensions defaultExtensionNs="com.intellij">
    <statusBarWidgetFactory
        id="MyWidget"
        implementation="com.example.MyWidgetFactory"
        order="after Position"/>
</extensions>
```

### Notification Group
```xml
<extensions defaultExtensionNs="com.intellij">
    <notificationGroup id="MyPlugin.Notifications"
                       displayType="BALLOON"
                       isLogByDefault="true"/>
</extensions>
```

Display types: `BALLOON`, `TOOL_WINDOW`, `STICKY_BALLOON`, `NONE`

## VCS Integration

### Checkin Handler Factory
```xml
<extensions defaultExtensionNs="com.intellij">
    <checkinHandlerFactory
        implementation="com.example.MyCheckinHandlerFactory"/>
</extensions>
```

### VCS Changes Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <vcs.changes.localIgnoredPaths
        implementation="com.example.MyIgnoredPathsProvider"/>
</extensions>
```

### Commit Message Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <commitMessageProvider
        implementation="com.example.MyCommitMessageProvider"/>
</extensions>
```

## Code Analysis

### Local Inspection
```xml
<extensions defaultExtensionNs="com.intellij">
    <localInspection
        language="JAVA"
        groupPath="Java"
        groupBundle="messages.InspectionsBundle"
        groupKey="group.names.probable.bugs"
        enabledByDefault="true"
        level="WARNING"
        implementationClass="com.example.MyInspection"/>
</extensions>
```

### Annotator
```xml
<extensions defaultExtensionNs="com.intellij">
    <annotator language="JAVA"
               implementationClass="com.example.MyAnnotator"/>
</extensions>
```

### Completion Contributor
```xml
<extensions defaultExtensionNs="com.intellij">
    <completion.contributor
        language="JAVA"
        implementationClass="com.example.MyCompletionContributor"/>
</extensions>
```

### Reference Contributor
```xml
<extensions defaultExtensionNs="com.intellij">
    <psi.referenceContributor
        language="JAVA"
        implementation="com.example.MyReferenceContributor"/>
</extensions>
```

## Editor Enhancements

### Line Marker Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <codeInsight.lineMarkerProvider
        language="JAVA"
        implementationClass="com.example.MyLineMarkerProvider"/>
</extensions>
```

### Inlay Hints Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <codeInsight.inlayProvider
        language="JAVA"
        implementationClass="com.example.MyInlayProvider"/>
</extensions>
```

### Documentation Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <documentationProvider
        implementation="com.example.MyDocumentationProvider"/>
</extensions>
```

## Project & Files

### Project Open Processor
```xml
<extensions defaultExtensionNs="com.intellij">
    <projectOpenProcessor
        implementation="com.example.MyProjectOpenProcessor"/>
</extensions>
```

### File Type
```xml
<extensions defaultExtensionNs="com.intellij">
    <fileType name="MyFile"
              implementationClass="com.example.MyFileType"
              fieldName="INSTANCE"
              language="MyLanguage"
              extensions="myext"/>
</extensions>
```

### File Editor Provider
```xml
<extensions defaultExtensionNs="com.intellij">
    <fileEditorProvider
        implementation="com.example.MyFileEditorProvider"/>
</extensions>
```

## Configuration & Settings

### Configurable (Settings Page)
```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationConfigurable
        parentId="tools"
        instance="com.example.MySettingsConfigurable"
        id="com.example.MySettings"
        displayName="My Plugin Settings"/>
</extensions>
```

### Project Configurable
```xml
<extensions defaultExtensionNs="com.intellij">
    <projectConfigurable
        parentId="project.propVCSSupport"
        instance="com.example.MyProjectSettingsConfigurable"
        id="com.example.MyProjectSettings"
        displayName="My Project Settings"/>
</extensions>
```

## Run Configurations

### Configuration Type
```xml
<extensions defaultExtensionNs="com.intellij">
    <configurationType
        implementation="com.example.MyRunConfigurationType"/>
</extensions>
```

### Run Configuration Producer
```xml
<extensions defaultExtensionNs="com.intellij">
    <runConfigurationProducer
        implementation="com.example.MyRunConfigurationProducer"/>
</extensions>
```

## Startup & Lifecycle

### Project Activity (Post-Startup)
```xml
<extensions defaultExtensionNs="com.intellij">
    <postStartupActivity
        implementation="com.example.MyStartupActivity"/>
</extensions>
```

### Background Post-Startup Activity
```xml
<extensions defaultExtensionNs="com.intellij">
    <backgroundPostStartupActivity
        implementation="com.example.MyBackgroundStartupActivity"/>
</extensions>
```

### Application Initializer
```xml
<extensions defaultExtensionNs="com.intellij">
    <applicationInitializedListener
        implementation="com.example.MyAppInitializedListener"/>
</extensions>
```

## Listeners

### Application Listeners
```xml
<applicationListeners>
    <listener class="com.example.MyAppListener"
              topic="com.intellij.openapi.project.ProjectManagerListener"/>
</applicationListeners>
```

### Project Listeners
```xml
<projectListeners>
    <listener class="com.example.MyProjectListener"
              topic="com.intellij.openapi.vfs.VirtualFileListener"/>
</projectListeners>
```

## Intent Actions & Quick Fixes

### Intention Action
```xml
<extensions defaultExtensionNs="com.intellij">
    <intentionAction>
        <language>JAVA</language>
        <className>com.example.MyIntentionAction</className>
        <category>Java/Declaration</category>
    </intentionAction>
</extensions>
```

## External Resources

- [Extension Point List (Official)](https://plugins.jetbrains.com/docs/intellij/extension-point-list.html)
- [Custom Extension Points](https://plugins.jetbrains.com/docs/intellij/plugin-extension-points.html)
