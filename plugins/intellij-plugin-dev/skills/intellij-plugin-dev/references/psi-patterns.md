# PSI (Program Structure Interface) Patterns

PSI is the foundation of IntelliJ Platform's code understanding. It provides a structured, navigable representation of source code.

## Core Concepts

### PSI Elements Hierarchy
```
PsiElement (base interface)
├── PsiFile (file root)
│   ├── PsiJavaFile
│   ├── KtFile
│   └── XmlFile
├── PsiClass
│   ├── PsiAnonymousClass
│   └── PsiTypeParameter
├── PsiMethod
├── PsiField
├── PsiStatement
└── PsiExpression
```

## Getting PSI Elements

### From Virtual File
```kotlin
val virtualFile: VirtualFile = ...
val psiFile = PsiManager.getInstance(project).findFile(virtualFile)
```

### From Editor
```kotlin
fun getPsiFromEditor(editor: Editor, project: Project): PsiFile? {
    val document = editor.document
    return PsiDocumentManager.getInstance(project).getPsiFile(document)
}

fun getElementAtCaret(editor: Editor, psiFile: PsiFile): PsiElement? {
    val offset = editor.caretModel.offset
    return psiFile.findElementAt(offset)
}
```

### From Action Event
```kotlin
override fun actionPerformed(e: AnActionEvent) {
    val psiFile = e.getData(CommonDataKeys.PSI_FILE)
    val psiElement = e.getData(CommonDataKeys.PSI_ELEMENT)
    val editor = e.getData(CommonDataKeys.EDITOR)
}
```

## Navigation Patterns

### Tree Traversal
```kotlin
// Get parent of specific type
val psiClass = PsiTreeUtil.getParentOfType(element, PsiClass::class.java)
val psiMethod = PsiTreeUtil.getParentOfType(element, PsiMethod::class.java)

// Get all children of type
val allMethods = PsiTreeUtil.findChildrenOfType(psiClass, PsiMethod::class.java)
val allFields = PsiTreeUtil.findChildrenOfType(psiClass, PsiField::class.java)

// Get direct children
val directChildren = PsiTreeUtil.getChildrenOfType(psiClass, PsiMethod::class.java)

// Check if element is ancestor
val isAncestor = PsiTreeUtil.isAncestor(parent, child, true)
```

### Sibling Navigation
```kotlin
// Next/previous siblings
val nextSibling = element.nextSibling
val prevSibling = element.prevSibling

// Skip whitespace
var next = element.nextSibling
while (next is PsiWhiteSpace) {
    next = next.nextSibling
}

// Using PsiTreeUtil
val nextMethod = PsiTreeUtil.getNextSiblingOfType(method, PsiMethod::class.java)
```

### Finding Elements by Condition
```kotlin
// Process all elements matching condition
PsiTreeUtil.processElements(psiFile) { element ->
    if (element is PsiMethodCallExpression) {
        // Process method calls
    }
    true // continue processing
}

// Find first matching element
val firstMethod = PsiTreeUtil.findChildOfType(psiClass, PsiMethod::class.java)

// Find all matching with custom filter
val publicMethods = PsiTreeUtil.findChildrenOfType(psiClass, PsiMethod::class.java)
    .filter { it.hasModifierProperty(PsiModifier.PUBLIC) }
```

## Working with Java PSI

### Classes
```kotlin
// Find class by name
val psiClass = JavaPsiFacade.getInstance(project)
    .findClass("com.example.MyClass", GlobalSearchScope.allScope(project))

// Get class info
val className = psiClass?.name
val qualifiedName = psiClass?.qualifiedName
val superClass = psiClass?.superClass
val interfaces = psiClass?.interfaces
val isInterface = psiClass?.isInterface ?: false
val isEnum = psiClass?.isEnum ?: false
```

### Methods
```kotlin
// Get all methods
val methods = psiClass?.methods ?: emptyArray()

// Find specific method
val mainMethod = psiClass?.findMethodsByName("main", false)?.firstOrNull()

// Get method info
val methodName = method.name
val returnType = method.returnType
val parameters = method.parameterList.parameters
val modifiers = method.modifierList
val isConstructor = method.isConstructor
```

### Fields
```kotlin
// Get all fields
val fields = psiClass?.fields ?: emptyArray()

// Find specific field
val field = psiClass?.findFieldByName("myField", false)

// Get field info
val fieldName = field?.name
val fieldType = field?.type
val initializer = field?.initializer
```

## Working with Kotlin PSI

### Import Kotlin PSI
```kotlin
import org.jetbrains.kotlin.psi.*
import org.jetbrains.kotlin.psi.psiUtil.*
```

### Classes and Functions
```kotlin
val ktFile = psiFile as? KtFile ?: return

// Get all classes
val ktClasses = ktFile.declarations.filterIsInstance<KtClass>()

// Get all functions
val ktFunctions = ktFile.declarations.filterIsInstance<KtNamedFunction>()

// Find function
val function = ktFile.findDescendantOfType<KtNamedFunction> {
    it.name == "myFunction"
}
```

### Properties
```kotlin
val ktClass: KtClass = ...
val properties = ktClass.getProperties()
val companionObject = ktClass.companionObjects.firstOrNull()
```

## PSI Modification

### Write Action Requirement
All PSI modifications must be wrapped in a write action:

```kotlin
WriteCommandAction.runWriteCommandAction(project) {
    // PSI modifications here
}

// Or with result
val result = WriteCommandAction.runWriteCommandAction(project, Computable {
    // modifications
    "result"
})
```

### Creating Elements (Java)
```kotlin
val factory = PsiElementFactory.getInstance(project)

// Create method
val method = factory.createMethodFromText(
    """
    public void newMethod() {
        System.out.println("Hello");
    }
    """.trimIndent(),
    psiClass
)

// Create field
val field = factory.createFieldFromText(
    "private String myField;",
    psiClass
)

// Create statement
val statement = factory.createStatementFromText(
    "System.out.println(x);",
    method
)
```

### Creating Elements (Kotlin)
```kotlin
val ktFactory = KtPsiFactory(project)

// Create function
val function = ktFactory.createFunction(
    """
    fun newFunction(): String {
        return "Hello"
    }
    """.trimIndent()
)

// Create property
val property = ktFactory.createProperty("val myProperty: String = \"value\"")

// Create expression
val expression = ktFactory.createExpression("println(\"Hello\")")
```

### Adding Elements
```kotlin
WriteCommandAction.runWriteCommandAction(project) {
    // Add method to class
    psiClass.add(newMethod)

    // Add at specific position
    psiClass.addBefore(newMethod, existingMethod)
    psiClass.addAfter(newMethod, existingMethod)

    // Replace element
    oldElement.replace(newElement)

    // Delete element
    element.delete()
}
```

## Search & Find Usages

### Find References
```kotlin
val references = ReferencesSearch.search(psiElement).findAll()
references.forEach { reference ->
    val element = reference.element
    val containingFile = element.containingFile
}
```

### Find Subclasses
```kotlin
val subclasses = ClassInheritorsSearch.search(psiClass).findAll()
```

### Find Method Overrides
```kotlin
val overridingMethods = OverridingMethodsSearch.search(psiMethod).findAll()
```

### Custom Search
```kotlin
// Search for all string literals
val scope = GlobalSearchScope.projectScope(project)
val processor = object : PsiElementProcessor<PsiLiteralExpression> {
    override fun execute(element: PsiLiteralExpression): Boolean {
        val value = element.value as? String
        if (value != null) {
            // Process string literal
        }
        return true // continue search
    }
}
PsiTreeUtil.processElements(psiFile, PsiLiteralExpression::class.java, processor)
```

## Common Patterns

### Safe Navigation
```kotlin
fun processMethod(element: PsiElement): String? {
    val method = PsiTreeUtil.getParentOfType(element, PsiMethod::class.java)
        ?: return null
    val containingClass = method.containingClass
        ?: return null
    return "${containingClass.qualifiedName}.${method.name}"
}
```

### Visitor Pattern
```kotlin
psiFile.accept(object : PsiRecursiveElementVisitor() {
    override fun visitElement(element: PsiElement) {
        if (element is PsiMethodCallExpression) {
            // Process method call
        }
        super.visitElement(element)
    }
})

// Kotlin specific
ktFile.accept(object : KtTreeVisitorVoid() {
    override fun visitNamedFunction(function: KtNamedFunction) {
        // Process function
        super.visitNamedFunction(function)
    }
})
```

### Document Synchronization
```kotlin
// Commit document changes to PSI
PsiDocumentManager.getInstance(project).commitDocument(document)

// Commit all documents
PsiDocumentManager.getInstance(project).commitAllDocuments()

// Check if document is committed
val isCommitted = PsiDocumentManager.getInstance(project).isCommitted(document)
```

## Performance Considerations

1. **Use `BGT` for action updates** - PSI reads should be on background thread
2. **Cache PSI elements carefully** - PSI can become invalid after modifications
3. **Check validity** - Use `element.isValid` before accessing stale references
4. **Use `runReadAction`** - Wrap reads in read action for thread safety
5. **Batch modifications** - Group changes in single write action

```kotlin
// Read action
ApplicationManager.getApplication().runReadAction {
    // PSI reads
}

// Write action
ApplicationManager.getApplication().runWriteAction {
    // PSI modifications
}
```
