# VFS (Virtual File System) Patterns

The Virtual File System (VFS) provides an abstraction layer for file operations in IntelliJ Platform, supporting local files, archives, and remote file systems.

## Core Concepts

### VirtualFile vs File
- `VirtualFile` - IntelliJ's abstraction, may not exist on disk
- `java.io.File` - Physical file on disk
- Always prefer `VirtualFile` for platform operations

### Getting VirtualFiles

#### From Local Path
```kotlin
val virtualFile = LocalFileSystem.getInstance()
    .findFileByPath("/path/to/file.kt")

// With NIO path
val virtualFile = LocalFileSystem.getInstance()
    .findFileByNioFile(path)

// Refresh and find (if file might not be cached)
val virtualFile = LocalFileSystem.getInstance()
    .refreshAndFindFileByPath("/path/to/file.kt")
```

#### From Project
```kotlin
// Project base directory
val baseDir = project.baseDir

// Project file (.idea/misc.xml)
val projectFile = project.projectFile

// Find file in project
val file = baseDir?.findFileByRelativePath("src/main/kotlin/Main.kt")
```

#### From Editor/Document
```kotlin
val editor: Editor = ...
val document = editor.document
val virtualFile = FileDocumentManager.getInstance().getFile(document)
```

#### From PSI
```kotlin
val psiFile: PsiFile = ...
val virtualFile = psiFile.virtualFile
```

## Reading Files

### Read Content
```kotlin
// As string
val content = VfsUtil.loadText(virtualFile)

// As bytes
val bytes = virtualFile.contentsToByteArray()

// As input stream
virtualFile.inputStream.use { stream ->
    // Process stream
}

// With charset
val content = String(virtualFile.contentsToByteArray(), virtualFile.charset)
```

### File Properties
```kotlin
val name = virtualFile.name                    // "MyFile.kt"
val extension = virtualFile.extension          // "kt"
val path = virtualFile.path                    // Full path
val url = virtualFile.url                      // file:// URL
val canonicalPath = virtualFile.canonicalPath  // Resolved symlinks
val parent = virtualFile.parent                // Parent directory
val isDirectory = virtualFile.isDirectory
val isValid = virtualFile.isValid
val exists = virtualFile.exists()
val length = virtualFile.length                // File size in bytes
val modificationStamp = virtualFile.modificationStamp
val timeStamp = virtualFile.timeStamp          // Last modified time
```

## Writing Files

### Write Action Requirement
All file modifications must be in a write action:

```kotlin
ApplicationManager.getApplication().runWriteAction {
    // File modifications here
}

// Or with WriteCommandAction for undo support
WriteCommandAction.runWriteCommandAction(project) {
    // File modifications here
}
```

### Write Content
```kotlin
ApplicationManager.getApplication().runWriteAction {
    // Write text
    VfsUtil.saveText(virtualFile, "new content")

    // Write bytes
    virtualFile.setBinaryContent(bytes)

    // Via output stream
    virtualFile.getOutputStream(this).use { stream ->
        stream.write(content.toByteArray())
    }
}
```

### Create Files and Directories
```kotlin
ApplicationManager.getApplication().runWriteAction {
    // Create file
    val newFile = parentDir.createChildData(this, "NewFile.kt")

    // Create file with content
    VfsUtil.saveText(newFile, "package com.example")

    // Create directory
    val newDir = parentDir.createChildDirectory(this, "newpackage")

    // Create nested directories
    VfsUtil.createDirectoryIfMissing(parentDir, "path/to/nested")
}
```

### Delete Files
```kotlin
ApplicationManager.getApplication().runWriteAction {
    virtualFile.delete(this)
}
```

### Move/Rename Files
```kotlin
ApplicationManager.getApplication().runWriteAction {
    // Rename
    virtualFile.rename(this, "NewName.kt")

    // Move
    virtualFile.move(this, newParentDir)
}
```

### Copy Files
```kotlin
ApplicationManager.getApplication().runWriteAction {
    // Copy file
    val copy = virtualFile.copy(this, targetDir, "CopyName.kt")

    // Copy directory recursively
    VfsUtil.copyDirectory(this, sourceDir, targetDir, null)
}
```

## File Events & Listeners

### VirtualFileListener
```kotlin
class MyFileListener : VirtualFileListener {
    override fun contentsChanged(event: VirtualFileEvent) {
        val file = event.file
        // Handle content change
    }

    override fun fileCreated(event: VirtualFileEvent) {
        // Handle file creation
    }

    override fun fileDeleted(event: VirtualFileEvent) {
        // Handle deletion
    }

    override fun fileMoved(event: VirtualFileMoveEvent) {
        // Handle move
    }

    override fun propertyChanged(event: VirtualFilePropertyEvent) {
        // Handle property change (rename, etc.)
    }
}

// Register listener
VirtualFileManager.getInstance().addVirtualFileListener(
    myListener,
    disposable  // Listener removed when disposed
)
```

### Via plugin.xml
```xml
<projectListeners>
    <listener class="com.example.MyFileListener"
              topic="com.intellij.openapi.vfs.VirtualFileListener"/>
</projectListeners>
```

### Bulk File Listener
For batch operations:

```kotlin
class MyBulkFileListener : BulkFileListener {
    override fun before(events: List<VFileEvent>) {
        // Before changes
    }

    override fun after(events: List<VFileEvent>) {
        events.forEach { event ->
            when (event) {
                is VFileCreateEvent -> handleCreate(event)
                is VFileDeleteEvent -> handleDelete(event)
                is VFileContentChangeEvent -> handleChange(event)
                is VFileMoveEvent -> handleMove(event)
                is VFilePropertyChangeEvent -> handlePropertyChange(event)
            }
        }
    }
}
```

## File Refresh

### Manual Refresh
```kotlin
// Refresh single file
virtualFile.refresh(false, false)  // async=false, recursive=false

// Refresh with callback
virtualFile.refresh(true, false) // async

// Refresh directory recursively
directory.refresh(false, true)  // recursive=true

// Refresh and find file
LocalFileSystem.getInstance().refreshAndFindFileByPath(path)
```

### Refresh After External Changes
```kotlin
// After writing via java.io
LocalFileSystem.getInstance().refreshIoFiles(listOf(file))

// After command line operations
VirtualFileManager.getInstance().asyncRefresh {
    // Callback after refresh
}
```

## Working with Documents

### Get Document from VirtualFile
```kotlin
val document = FileDocumentManager.getInstance().getDocument(virtualFile)
```

### Document <-> VirtualFile
```kotlin
// Document to VirtualFile
val virtualFile = FileDocumentManager.getInstance().getFile(document)

// VirtualFile to Document
val document = FileDocumentManager.getInstance().getDocument(virtualFile)
```

### Save Documents
```kotlin
// Save specific document
FileDocumentManager.getInstance().saveDocument(document)

// Save all documents
FileDocumentManager.getInstance().saveAllDocuments()

// Reload from disk
FileDocumentManager.getInstance().reloadFromDisk(document)
```

## Common Patterns

### Find or Create File
```kotlin
fun findOrCreateFile(parent: VirtualFile, name: String, content: String): VirtualFile {
    return parent.findChild(name) ?: run {
        ApplicationManager.getApplication().runWriteAction(Computable {
            val file = parent.createChildData(this, name)
            VfsUtil.saveText(file, content)
            file
        })
    }
}
```

### Safe File Read
```kotlin
fun safeReadFile(virtualFile: VirtualFile?): String? {
    if (virtualFile == null || !virtualFile.isValid || virtualFile.isDirectory) {
        return null
    }
    return try {
        VfsUtil.loadText(virtualFile)
    } catch (e: IOException) {
        null
    }
}
```

### Process Files Recursively
```kotlin
fun processFilesRecursively(
    directory: VirtualFile,
    processor: (VirtualFile) -> Boolean
) {
    VfsUtilCore.iterateChildrenRecursively(
        directory,
        { it.isDirectory || it.extension == "kt" },  // Filter
        { processor(it) }  // Processor
    )
}
```

### File Path Utilities
```kotlin
// Relative path
val relativePath = VfsUtilCore.getRelativePath(file, baseDir)

// Find common ancestor
val commonAncestor = VfsUtil.getCommonAncestor(file1, file2)

// Is ancestor
val isAncestor = VfsUtilCore.isAncestor(parent, child, false)

// Path to URL and back
val url = VfsUtilCore.pathToUrl(path)
val path = VfsUtilCore.urlToPath(url)
```

## Archive Files

### Working with JARs
```kotlin
val jarFile = JarFileSystem.getInstance()
    .findFileByPath("/path/to/file.jar!/com/example/Class.class")

// List JAR contents
val jarRoot = JarFileSystem.getInstance()
    .getJarRootForLocalFile(localJarFile)
jarRoot?.children?.forEach { entry ->
    println(entry.path)
}
```

## Performance Tips

1. **Cache VirtualFiles carefully** - They can become invalid
2. **Check `isValid`** before using cached files
3. **Use async refresh** for UI responsiveness
4. **Batch write operations** in single write action
5. **Use `BulkFileListener`** for monitoring many files
6. **Avoid blocking on file operations** in EDT
