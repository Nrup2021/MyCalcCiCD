# MyCalculator

MyCalculator is an Android calculator built with Kotlin and Jetpack Compose. It supports
live expression previews, the four basic arithmetic operations, decimal values, and
parenthesized expressions. Its calculator logic is separated from the UI so that input
handling and expression evaluation can be tested independently.

## Objective

This project demonstrates how to build, test, and deliver an Android app with GitHub
Actions:

- Pull requests targeting `main` run Kotlin lint, build the debug APK, run JVM unit tests,
  and run Android instrumentation tests on an emulator. The debug APK and test results
  are available as workflow artifacts.
- The `main` branch is protected by a ruleset requiring pull requests and successful
  **Lint / Format Check** and **Build and Test** checks before merge.
- After a successful push to `main` (such as a PR merge), CI uses signing credentials
  stored as GitHub Actions repository secrets to build a signed, versioned release APK
  and publish it as a GitHub Release.

This is continuous integration plus continuous delivery: the APK is published for
people to download, but is not automatically installed on devices or deployed through
the Play Store. See [Android CI/CD with GitHub Actions](#android-cicd-with-github-actions)
for the workflow and signing setup.

### Example: adding negative-number support

Suppose a contributor changes the calculator to accept a negative number such as `-5+2`.
They create a feature branch from the latest `main`, push their changes, and open a pull
request targeting `main`.

1. GitHub starts **Lint / Format Check** and **Build and Test** for the pull request.
2. If the new Kotlin code has a formatting issue, lint fails and the `main` ruleset
   blocks merging. The contributor fixes the formatting and pushes another commit.
3. The build and tests then check that the app compiles and that the expression
   evaluator and UI behave correctly. For example, a unit test can catch if `-5+2`
   incorrectly returns `-7` instead of `-3`.
4. The contributor fixes any code or test failure and pushes again. GitHub reruns the
   checks for the updated pull request.
5. Once all required checks pass, GitHub allows the pull request to be merged. The push
   to `main` then triggers the signed, versioned APK release workflow.

This catches style, build, and behavior problems before they enter `main`. Opening a
pull request is still allowed when a check fails; the branch ruleset prevents merging
until the required checks pass.

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

## Android CI/CD with GitHub Actions

This project demonstrates Android continuous integration and continuous delivery
(CI/CD) using **GitHub Actions, Gradle, Kotlin, and ktlint**. The workflow runs code
style checks, builds the app, and runs unit and instrumentation tests. Test result files
are uploaded as workflow artifacts, including when a test job fails. After all checks
succeed, pushes to `main` publish a versioned GitHub Release containing the signed APK.
Other workflow runs upload the debug APK as a workflow artifact. This is automated
delivery for download; it does not install or deploy the app to users' devices.

The **Android CI/CD** workflow is defined in
[`.github/workflows/android-ci.yml`](.github/workflows/android-ci.yml). It runs when
code is pushed to the `ci-prep` or `main` branch, or when a pull request targets `main`.
The checks are chained; the final delivery step depends on the event branch:

```text
Lint → Build Debug APK → Unit Tests → Instrumentation Tests
    → [main push: Build Signed Release APK → Publish versioned GitHub Release]
    → [other events: Publish Debug APK]
```

1. **Lint (`ktlintCheck`)** checks Kotlin formatting and coding standards.
2. **Build Debug APK (`assembleDebug`)** compiles the debug application.
3. **Unit Tests (`testDebugUnitTest`)** runs JVM tests on the CI host without an Android
   device.
4. **Instrumentation Tests (`connectedDebugAndroidTest`)** runs Android tests on an
   API 34 x86_64 emulator.
5. After all checks pass, a push to `main` builds and publishes the signed release APK.
   Each release is tagged `v1.0.<run-number>`; the Android `versionName` is
   `1.0.<run-number>` and `versionCode` is the workflow run number. Pull requests and
   pushes to other branches upload the debug APK as a workflow artifact instead.

If any required check fails, APK publishing/upload is skipped.

### Require CI checks before merging pull requests

Protect the `main` branch with a GitHub branch ruleset so pull requests cannot be
merged until CI succeeds:

1. In the repository, open **Settings → Rules → Rulesets** and create or edit a branch
   ruleset.
2. Set the ruleset to **Active** and target the `main` branch (for example, with
   **Include by pattern** → `main`).
3. Enable **Require a pull request before merging**.
4. Enable **Require status checks to pass** and select **Lint / Format Check** and
   **Build and Test**. If they are not listed yet, first run the workflow on a pull
   request targeting `main`, then return to the ruleset and select them.
5. Do not require **Publish GitHub Release** as a pull request check. That job runs
   only after a successful push to `main`.

To verify the protection, create a temporary test branch from the latest `main`, make
a harmless change, push the branch, and open a pull request with `main` as the base
branch. Confirm that **Lint / Format Check** and **Build and Test** appear and complete
successfully. Before they complete, GitHub should show that merging is blocked by
pending checks; after they pass, the merge becomes available, subject to any other
rules you configured. Close the test pull request without merging if the change is only
for verification. For a stronger check, use a disposable test PR with a known failing
check and confirm merging remains blocked, then close it without merging.

After a PR is merged, the push to `main` starts the signed release build and
**Publish GitHub Release** job. Check the run in the repository's **Actions** tab; after
it succeeds, find the versioned APK under **Releases**.

GitHub Releases provide a free way to distribute APK downloads without a Play Store
account. Anyone can download a release from a public repository; releases in a private
repository are only available to people with repository access. Each successful push to
`main` creates or updates the corresponding versioned release and attaches the APK.

### Configure release signing

Create the release keystore once on a trusted local machine. You need a JDK installed
for the `keytool` command:

```bash
keytool -genkeypair -v \
  -keystore release-key.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias mycalculator
```

Choose and securely save the keystore and key passwords. Keep a secure backup of
`release-key.jks`: future app updates must use the same signing key. Do not commit the
keystore or passwords to the repository.

Encode the keystore for GitHub Actions. On macOS, from the directory containing the
file, run:

```bash
base64 -i release-key.jks | pbcopy
```

This copies the Base64-encoded keystore to the clipboard. In the GitHub repository, go
to **Settings → Secrets and variables → Actions → Repository secrets → New repository
secret**. Add each of the following as a separate **repository secret** (not a
repository variable or an environment secret):

| Secret name | Value |
| --- | --- |
| `ANDROID_KEYSTORE_BASE64` | Paste the Base64 contents copied above. |
| `ANDROID_KEYSTORE_PASSWORD` | The keystore password chosen when creating the keystore. |
| `ANDROID_KEY_ALIAS` | The alias used when creating the key (in this example, `mycalculator`). |
| `ANDROID_KEY_PASSWORD` | The password for that key. |

The workflow reads these secrets only for pushes to `main`. It decodes the keystore
into the runner's temporary directory, uses the credentials to build the signed release
APK, and removes the temporary keystore afterward. Other workflow runs upload the debug
APK and do not need the signing secrets. The separate publishing job uses the workflow's
`GITHUB_TOKEN` with `contents: write` permission to create the release; the build job
only has read permission.

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

<img width="1884" height="1769" alt="screencapture-github-Nrup2021-MyCalcCiCD-actions-runs-39234083019-2026-10-19-15_40_10" src="https://github.com/user-attachments/assets/0bf1c438-bfcf-4e94-8300-460ed38c46d4" />


Together, these practices provide automated Kotlin quality checks, early detection of
build and test failures, repeatable Android validation, and visible test results. They
offer a foundation for using AI coding agents while keeping correctness and code quality
subject to developer review.
