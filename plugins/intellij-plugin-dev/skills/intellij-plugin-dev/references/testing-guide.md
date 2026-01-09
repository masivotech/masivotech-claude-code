# Testing IntelliJ Platform Plugins

This guide covers testing strategies for IntelliJ Platform plugins including unit tests, functional tests, UI tests, and compatibility verification.

## Test Dependencies

### build.gradle.kts
```kotlin
dependencies {
    testImplementation("junit:junit:4.13.2")

    intellijPlatform {
        testFramework(TestFrameworkType.Platform)
        // For UI testing:
        // testFramework(TestFrameworkType.Plugin.Java)
    }
}
```

## Base Test Classes

### BasePlatformTestCase
For tests requiring a project context:

```kotlin
class MyServiceTest : BasePlatformTestCase() {

    override fun getTestDataPath(): String {
        return "src/test/testData"
    }

    fun testServiceInitialization() {
        val service = MyProjectService.getInstance(project)
        assertNotNull(service)
    }

    fun testSomeFunctionality() {
        val result = myService.doSomething()
        assertEquals("expected", result)
    }
}
```

### LightPlatformCodeInsightFixture4TestCase
For code insight tests (completion, highlighting, etc.):

```kotlin
class MyCompletionTest : LightPlatformCodeInsightFixture4TestCase() {

    override fun getTestDataPath(): String {
        return "src/test/testData"
    }

    fun testCompletion() {
        myFixture.configureByText("Test.kt", """
            class Test {
                fun test() {
                    println(<caret>)
                }
            }
        """.trimIndent())

        myFixture.completeBasic()
        val lookupElements = myFixture.lookupElementStrings
        assertNotNull(lookupElements)
        assertTrue(lookupElements!!.contains("expectedItem"))
    }
}
```

### LightJavaCodeInsightFixtureTestCase
For Java-specific tests:

```kotlin
class MyJavaInspectionTest : LightJavaCodeInsightFixtureTestCase() {

    override fun getTestDataPath(): String {
        return "src/test/testData/inspections"
    }

    fun testInspection() {
        myFixture.enableInspections(MyInspection::class.java)
        myFixture.testHighlighting("TestFile.java")
    }
}
```

## Test Fixtures

### Configuring Test Files

```kotlin
// From string
myFixture.configureByText("Test.kt", "fun main() {}")

// From test data file
myFixture.configureByFile("TestFile.kt")

// Multiple files
myFixture.configureByFiles("Main.kt", "Helper.kt", "Data.kt")

// With caret position
myFixture.configureByText("Test.kt", """
    fun test() {
        val x = <caret>
    }
""".trimIndent())

// With selection
myFixture.configureByText("Test.kt", """
    fun test() {
        <selection>val x = 1</selection>
    }
""".trimIndent())
```

### Performing Actions

```kotlin
// Type text
myFixture.type("newText")

// Perform editor action
myFixture.performEditorAction(IdeActions.ACTION_EDITOR_COMPLETE_STATEMENT)

// Custom action
myFixture.performEditorAction("MyPlugin.MyAction")

// Invoke intention
myFixture.launchAction(myFixture.findSingleIntention("My Intention"))
```

### Checking Results

```kotlin
// Check file content
myFixture.checkResult("""
    expected content here
""".trimIndent())

// Check highlighting
myFixture.checkHighlighting()

// Check against golden file
myFixture.checkResultByFile("expected/TestFile.kt")
```

## Unit Testing Services

### Application Service Test
```kotlin
class MyAppServiceTest : BasePlatformTestCase() {

    fun testServiceOperation() {
        val service = ApplicationManager.getApplication()
            .getService(MyAppService::class.java)

        val result = service.compute()
        assertEquals(42, result)
    }
}
```

### Project Service Test
```kotlin
class MyProjectServiceTest : BasePlatformTestCase() {

    fun testProjectOperation() {
        val service = project.getService(MyProjectService::class.java)

        val result = service.getProjectName()
        assertEquals(project.name, result)
    }
}
```

## Testing Actions

```kotlin
class MyActionTest : BasePlatformTestCase() {

    fun testActionEnabled() {
        myFixture.configureByText("Test.kt", "fun main() {}")

        val action = ActionManager.getInstance().getAction("MyPlugin.MyAction")
        val event = TestActionEvent.createTestEvent(action, dataContext)

        action.update(event)

        assertTrue(event.presentation.isEnabled)
    }

    fun testActionPerformed() {
        myFixture.configureByText("Test.kt", "fun <caret>main() {}")

        myFixture.performEditorAction("MyPlugin.MyAction")

        myFixture.checkResult("expected result")
    }

    private val dataContext: DataContext
        get() = MapDataContext().apply {
            put(CommonDataKeys.PROJECT, project)
            put(CommonDataKeys.EDITOR, myFixture.editor)
            put(CommonDataKeys.PSI_FILE, myFixture.file)
        }
}
```

## Testing Inspections

### Highlighting Test
```kotlin
class MyInspectionTest : LightJavaCodeInsightFixtureTestCase() {

    override fun setUp() {
        super.setUp()
        myFixture.enableInspections(MyInspection::class.java)
    }

    fun testWarning() {
        myFixture.configureByText("Test.java", """
            class Test {
                void test() {
                    <warning descr="My warning message">problemCode</warning>();
                }
            }
        """.trimIndent())

        myFixture.checkHighlighting()
    }
}
```

### Quick Fix Test
```kotlin
fun testQuickFix() {
    myFixture.configureByText("Test.java", """
        class Test {
            void test() {
                <caret>problemCode();
            }
        }
    """.trimIndent())

    val intention = myFixture.findSingleIntention("Fix problem")
    myFixture.launchAction(intention)

    myFixture.checkResult("""
        class Test {
            void test() {
                fixedCode();
            }
        }
    """.trimIndent())
}
```

## Testing Completion

```kotlin
class MyCompletionTest : LightPlatformCodeInsightFixture4TestCase() {

    fun testBasicCompletion() {
        myFixture.configureByText("Test.kt", """
            fun main() {
                val list = listOf(1, 2, 3)
                list.<caret>
            }
        """.trimIndent())

        myFixture.completeBasic()

        val lookupStrings = myFixture.lookupElementStrings
        assertNotNull(lookupStrings)
        assertTrue(lookupStrings!!.contains("filter"))
        assertTrue(lookupStrings.contains("map"))
    }

    fun testCompletionSelection() {
        myFixture.configureByText("Test.kt", "fun main() { prin<caret> }")

        myFixture.completeBasic()
        myFixture.type("\n")

        myFixture.checkResult("fun main() { println(<caret>) }")
    }
}
```

## Testing with Test Data Files

### Directory Structure
```
src/
├── main/
└── test/
    ├── kotlin/
    │   └── com/example/
    │       └── MyTest.kt
    └── testData/
        ├── inspections/
        │   ├── TestFile.java
        │   └── TestFile_after.java
        └── completion/
            └── TestFile.kt
```

### Using Test Data
```kotlin
class MyInspectionTest : LightJavaCodeInsightFixtureTestCase() {

    override fun getTestDataPath(): String {
        return "src/test/testData/inspections"
    }

    fun testInspectionHighlighting() {
        myFixture.testHighlighting("TestFile.java")
    }

    fun testQuickFix() {
        myFixture.configureByFile("TestFile.java")
        val intention = myFixture.findSingleIntention("Fix this")
        myFixture.launchAction(intention)
        myFixture.checkResultByFile("TestFile_after.java")
    }
}
```

## UI Testing

### RemoteRobot (Gradle Plugin)

For UI/integration tests, use the RemoteRobot framework:

```kotlin
dependencies {
    testImplementation("com.intellij.remoterobot:remote-robot:0.11.22")
    testImplementation("com.intellij.remoterobot:remote-fixtures:0.11.22")
}
```

```kotlin
class MyUITest {
    @JvmField
    @Rule
    val remoteRobot = RemoteRobotRule()

    @Test
    fun testToolWindow() {
        with(remoteRobot.find(IdeaFrame::class.java)) {
            // Find and click tool window button
            find<JButtonFixture>(byXpath("//div[@text='My Tool']")).click()

            // Verify content
            val content = find<JLabelFixture>(byXpath("//div[@class='JBLabel']"))
            assertEquals("Expected Text", content.text)
        }
    }
}
```

## Plugin Verifier

### Configuration
```kotlin
intellijPlatform {
    pluginVerification {
        ides {
            recommended()
            // Or specify versions:
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.1")
            // ide(IntelliJPlatformType.IntellijIdeaCommunity, "2024.2")
            // ide(IntelliJPlatformType.AndroidStudio, "2024.1.1")
        }

        // Mute specific problems
        freeArgs.addAll(
            "-mute", "TemplateWordInPluginId"
        )
    }
}
```

### Running Verifier
```bash
./gradlew runPluginVerifier
```

### Common Issues

| Issue | Solution |
|-------|----------|
| `Deprecated API usage` | Update to new API or suppress with `@Suppress` |
| `Incompatible changes` | Check API compatibility for target versions |
| `Missing dependencies` | Add missing `<depends>` in plugin.xml |
| `Internal API usage` | Replace with public API alternatives |

## Mocking

### MockK for Kotlin
```kotlin
class MyServiceTest : BasePlatformTestCase() {

    @MockK
    lateinit var mockDependency: SomeDependency

    override fun setUp() {
        super.setUp()
        MockKAnnotations.init(this)
    }

    fun testWithMock() {
        every { mockDependency.getValue() } returns "mocked"

        val service = MyService(mockDependency)
        val result = service.process()

        assertEquals("mocked-processed", result)
        verify { mockDependency.getValue() }
    }
}
```

## Test Utilities

### WriteCommandAction in Tests
```kotlin
fun testModification() {
    myFixture.configureByText("Test.kt", "fun main() {}")

    WriteCommandAction.runWriteCommandAction(project) {
        val psiFile = myFixture.file
        val document = PsiDocumentManager.getInstance(project)
            .getDocument(psiFile)
        document?.insertString(0, "// Comment\n")
        PsiDocumentManager.getInstance(project).commitDocument(document!!)
    }

    myFixture.checkResult("// Comment\nfun main() {}")
}
```

### Waiting for Background Tasks
```kotlin
fun testBackgroundTask() {
    // Start async operation
    myService.startBackgroundTask()

    // Wait for completion
    PlatformTestUtil.dispatchAllEventsInIdeEventQueue()

    // Or with timeout
    PlatformTestUtil.waitWhileBusy { myService.isBusy }

    // Assert results
    assertTrue(myService.isCompleted)
}
```

## Best Practices

1. **Use appropriate base class** - Choose the right test base for your scenario
2. **Isolate tests** - Each test should be independent
3. **Use test data files** - For complex test scenarios
4. **Test edge cases** - Empty files, invalid input, etc.
5. **Run verifier in CI** - Catch compatibility issues early
6. **Mock external dependencies** - Keep tests fast and reliable
7. **Test on multiple IDE versions** - Use plugin verifier
