# AGENTS.md

## Cursor Cloud specific instructions

This is a mobile CI/CD demo monorepo with two independent modules: **Android** (Kotlin/Gradle) and **iOS** (Swift Package Manager). There are no backend services, databases, or Docker Compose setups. All CI runs on GitHub Actions.

### Network restrictions (Cloud VM)

The Cloud VM environment blocks access to several key domains:
- `repo1.maven.org` (Maven Central)
- `plugins.gradle.org` (Gradle Plugin Portal)
- `services.gradle.org` (Gradle distributions)
- `download.swift.org` (Swift toolchain downloads)

Accessible domains include `github.com`, `dl.google.com`, and Docker Hub.

### Android (Kotlin)

- The `gradle-wrapper.jar` is **not committed** to the repo. The CI generates it with `gradle wrapper` before builds.
- Because Maven Central and Gradle Plugin Portal are blocked, the full `./gradlew lint assembleDebug test` pipeline **cannot run** in the Cloud VM.
- **Workaround**: compile and test the pure-JVM Kotlin code directly with `kotlinc` and `junit-4.13.2.jar`:
  ```
  /opt/kotlinc/bin/kotlinc -cp "/opt/libs/junit-4.13.2.jar:/opt/libs/hamcrest-core-1.3.jar" \
    android/app/src/main/kotlin/com/example/aviatordemo/Calculator.kt \
    android/app/src/test/kotlin/com/example/aviatordemo/CalculatorTest.kt \
    -d /tmp/android-build/
  java -cp "/tmp/android-build:/opt/libs/junit-4.13.2.jar:/opt/libs/hamcrest-core-1.3.jar:/opt/kotlinc/lib/kotlin-stdlib.jar" \
    org.junit.runner.JUnitCore com.example.aviatordemo.CalculatorTest
  ```
- `MainActivity.kt` cannot compile without Android SDK framework JARs; it is excluded from local compilation. The Calculator and its tests are pure JVM code.

### iOS (Swift)

- Swift is **not installed natively** on the VM. Use the `swift:5.10` Docker image instead.
- Build: `docker run --rm -v /workspace/ios:/workspace/ios -w /workspace/ios swift:5.10 swift build`
- Test: `docker run --rm -v /workspace/ios:/workspace/ios -w /workspace/ios swift:5.10 swift test`
- SwiftLint is not installed locally; the CI installs it inside the Docker container. To run SwiftLint locally, install it inside the swift container.

### Key paths

| Module | Source | Tests |
|--------|--------|-------|
| Android | `android/app/src/main/kotlin/com/example/aviatordemo/` | `android/app/src/test/kotlin/com/example/aviatordemo/` |
| iOS | `ios/Sources/AviatorDemo/` | `ios/Tests/AviatorDemoTests/` |

### CI Workflows

See `README.md` for workflow descriptions. Workflows are in `.github/workflows/`.
