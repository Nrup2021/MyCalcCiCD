# MyCalculator

MyCalculator is an Android calculator built with Kotlin and Jetpack Compose. It supports
live expression previews, the four basic arithmetic operations, decimal values, and
parenthesized expressions. Its calculator logic is separated from the UI so that input
handling and expression evaluation can be tested independently.

## Features

- Addition, subtraction, multiplication, and division.
- Parentheses and unary minus.
- Live result previews while entering an expression.
- Clear and delete controls, with input rules for incomplete expressions.
- Expression state restoration through `SavedStateHandle`.
- A Compose UI with light and dark theme previews.

## Code overview

The Android application is in the `app` module, under
`app/src/main/java/com/plcoding/mycalculator`.

- **`MainActivity`** enables edge-to-edge display and hosts the calculator inside the app
  theme and a Material 3 scaffold.
- **`calculator/presentation`** contains the Compose screen, UI state, user actions, and
  `CalculatorViewModel`. The ViewModel validates keypad input, saves the current
  expression, and requests a live result preview.
- **`calculator/presentation/components`** contains the calculator display and keypad
  composables.
- **`calculator/domain`** contains the calculator operators and expression-evaluation
  interface. `DefaultExpressionEvaluator` parses expressions with standard operator
  precedence and parentheses; incomplete or undefined expressions have no result.
- **`ui/theme`** contains the Compose color, typography, and theme definitions.

## Build and test

Use JDK 17 and an Android SDK installation that includes the project's compile SDK. The
CI workflow uses JDK 17.

```bash
# Build the debug app
./gradlew assembleDebug

# Run local JVM unit tests
./gradlew testDebugUnitTest

# Run Kotlin formatting and style checks
./gradlew ktlintCheck

# Run Android instrumentation tests on a connected device or emulator
./gradlew connectedDebugAndroidTest
```

On Windows, use `gradlew.bat` instead of `./gradlew`.

Unit tests cover the expression evaluator and ViewModel behavior. Instrumentation tests
run in an Android runtime on a device or emulator.

## Android CI with GitHub Actions

This project demonstrates a YAML-based Android continuous integration (CI) pipeline
using **GitHub Actions, Gradle, Kotlin, and ktlint**. It automates code style checks,
Android builds, local unit tests, and instrumentation tests on an Android emulator.
Test result files are uploaded as workflow artifacts, including when a test job fails.

The workflow is defined in
[`.github/workflows/android-ci.yml`](.github/workflows/android-ci.yml). It runs when
code is pushed to the `ci-prep` or `main` branch, or when a pull request targets `main`.
The jobs are chained so each stage only runs after the preceding stage succeeds:

```text
Lint → Build APK → Unit Tests → Instrumentation Tests → Publish APK
```

1. **Lint (`ktlintCheck`)** checks Kotlin formatting and coding standards.
2. **Build APK (`assembleDebug`)** compiles the debug application.
3. **Unit Tests (`testDebugUnitTest`)** runs JVM tests on the CI host without an Android
   device.
4. **Instrumentation Tests (`connectedDebugAndroidTest`)** runs Android tests on an
   API 34 x86_64 emulator.
5. **Publish APK** uploads the debug APK as a workflow artifact.

If any required check fails before the final upload step, the APK publish step is skipped
and the build is treated as unsuccessful.

### Why run lint first?

The pipeline follows the **fail-fast principle**: inexpensive checks run early so
formatting problems are found before the workflow spends time building and testing.
This gives faster feedback, avoids unnecessary CI work after a failure, and provides a
clear quality gate before changes are reviewed or merged.

## Test-Driven Development for agentic Android development

Test-Driven Development (TDD) defines expected behavior through tests before production
code is implemented. In agentic Android development, AI coding agents can help analyze
requirements, create tests, implement features, debug failures, and refactor. Tests make
the expected behavior executable and give developers and agents a way to validate
changes incrementally. Developers remain responsible for reviewing the implementation
and test quality.

### The Red-Green-Refactor cycle

1. **Red:** Write a test for the required behavior and confirm that it fails for the
   expected reason.
2. **Green:** Implement the minimum production code needed to make the test pass.
3. **Refactor:** Improve the code while keeping the tests passing.

Repeat this cycle for each behavior or feature.

### Agentic Android development workflow

1. **Plan:** Analyze the requirement, existing architecture, expected behavior, and edge
   cases.
2. **Write tests:** Add tests for the behavior before implementing the business logic.
3. **Verify Red:** Run the tests and confirm that they fail as expected.
4. **Implement:** Make the smallest change that satisfies the tests.
5. **Verify Green:** Rerun the tests and investigate any failures.
6. **Refactor:** Improve readability and structure without changing behavior.
7. **Validate locally:** Run relevant tests, `ktlintCheck`, and the Android build.
8. **Push changes:** Commit and push to the appropriate Git branch.
9. **Run CI:** Let GitHub Actions validate the pushed changes independently.
10. **Review and merge:** Inspect the implementation, assertions, CI results, and
    remaining risks before merging.

An AI coding agent prompt for this workflow could be:

> Implement the requested Android feature using Test-Driven Development.
>
> 1. Inspect the existing project architecture and coding conventions.
> 2. Identify expected behavior and edge cases.
> 3. Write meaningful unit tests before implementing production code.
> 4. Run the tests and confirm that they fail for the expected reason.
> 5. Implement the minimum code required to make the tests pass.
> 6. Refactor while keeping the tests passing.
> 7. Run the relevant unit tests, ktlint checks, and Android build.
> 8. Add or update instrumentation tests when Android runtime behavior requires
>    validation.
> 9. Do not remove, weaken, or change test assertions merely to make tests pass.
> 10. Summarize the implementation, test results, and remaining limitations.

This approach encourages small, verifiable changes instead of implementing a feature
all at once without incremental validation.

## Local unit tests and instrumentation tests

| Aspect | Local unit tests | Instrumentation tests |
| --- | --- | --- |
| Gradle task | `testDebugUnitTest` | `connectedDebugAndroidTest` |
| Execution environment | JVM on the development or CI host | Android device or emulator |
| Android runtime | Not required | Required |
| Typical use | Business logic, ViewModels, utilities, and validation | Android components, UI interactions, and framework-dependent behavior |
| Execution cost | Usually faster | Usually slower due to emulator startup and execution |

Local unit tests are useful for rapid TDD feedback. Instrumentation tests complement
them by checking behavior that needs Android APIs, UI components, or an Android runtime;
they do not replace the faster JVM tests.

## How TDD and CI work together

TDD guides how a feature is developed, while CI independently checks committed changes.
Local tests provide rapid feedback during implementation, GitHub Actions runs the
configured quality gates on pushes and pull requests, and code review provides human
oversight. CI alone does not establish that tests were written before implementation.

```text
Requirement → Test → Red → Implementation → Green → Refactor
            → CI Validation → Code Review
```

## Key technologies and benefits

- **Kotlin and Jetpack Compose:** Android application and declarative UI development.
- **Gradle:** Build, lint, and test automation.
- **GitHub Actions:** Repeatable CI workflows for pushes and pull requests.
- **ktlint:** Kotlin style and formatting checks.
- **JUnit and Android testing libraries:** Automated local and instrumentation tests.
- **Android Emulator:** Runs instrumentation tests in an Android environment.
- **TDD and AI coding agents:** Support test-first, incremental implementation and
  validation.

## Image showing workflow/pipeline executed with specified jobs and APK is generated

<img width="1889" height="913" alt="Screenshot 2026-10-19 at 2 31 43 PM" src="https://github.com/user-attachments/assets/ef854685-3c7b-47f4-8fc4-26d5dc6dc25a" />

Together, these practices provide automated Kotlin quality checks, early detection of
build and test failures, repeatable Android validation, and visible test results. They
offer a foundation for using AI coding agents while keeping correctness and code quality
subject to developer review.
