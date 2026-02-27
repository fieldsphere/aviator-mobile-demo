## Cursor Cloud specific instructions

This is a CI/CD demo monorepo with two independent toy calculator libraries (Android + iOS) and GitHub Actions workflows. There are no runtime services, databases, or web apps.

### Architecture overview

| Directory | Stack | Purpose |
|-----------|-------|---------|
| `android/` | Kotlin + Gradle + JUnit 4 | Calculator class with unit tests |
| `ios/` | Swift 5.9+ Package + XCTest | Calculator struct with unit tests |
| `.github/workflows/` | GitHub Actions | CI/CD pipelines (PR checks, Aviator integration) |

### Running Android tests

The full Gradle build (`./gradlew test`) requires network access to Maven Central and Gradle Plugin Portal to download the Kotlin Gradle Plugin and dependencies. In restricted network environments, compile and test directly:

```sh
KOTLIN_HOME=/opt/kotlinc
JUNIT_JAR=/usr/share/java/junit4.jar
HAMCREST_JAR=/usr/share/java/hamcrest-core.jar

# Compile
$KOTLIN_HOME/bin/kotlinc \
  -cp "$JUNIT_JAR:$HAMCREST_JAR" \
  android/app/src/main/kotlin/com/example/aviatordemo/Calculator.kt \
  android/app/src/test/kotlin/com/example/aviatordemo/CalculatorTest.kt \
  -d /tmp/android-build

# Run tests
java -cp "/tmp/android-build:$JUNIT_JAR:$HAMCREST_JAR:$KOTLIN_HOME/lib/kotlin-stdlib.jar" \
  org.junit.runner.JUnitCore com.example.aviatordemo.CalculatorTest
```

**Gotcha**: The `gradlew` script in the repo has a quoting bug in `DEFAULT_JVM_OPTS` that causes `Error: Could not find or load main class "-Xmx64m"`. The CI workflow regenerates the wrapper via `gradle wrapper` each run, so this doesn't affect CI. If you need to use `./gradlew`, invoke the wrapper jar directly: `java -Xmx64m -Xms64m -classpath gradle/wrapper/gradle-wrapper.jar org.gradle.wrapper.GradleWrapperMain <args>`.

### Running iOS tests

Swift is not installed natively. Use the `swift:5.10` Docker image (matches CI):

```sh
docker run --rm -v /workspace/ios:/workspace/ios -w /workspace/ios swift:5.10 swift build
docker run --rm -v /workspace/ios:/workspace/ios -w /workspace/ios swift:5.10 swift test
```

### Prerequisites installed in the snapshot

- JDK 21 (system) — backward-compatible with JDK 17 target
- Android SDK at `/opt/android-sdk` (API 34, build-tools 34.0.0)
- Kotlin compiler 1.9.22 at `/opt/kotlinc`
- JUnit 4.13.2 at `/usr/share/java/junit4.jar`
- Docker CE with `swift:5.10` and `gradle:8.5-jdk17` images pulled
- Docker requires `sudo dockerd` to start (see below)

### Starting Docker (required for iOS tests)

```sh
sudo dockerd &>/tmp/dockerd.log &
sleep 3
sudo chmod 666 /var/run/docker.sock
```

### Lint

- **Android lint** (`./gradlew lint`): Requires full Gradle build with network; not available in restricted environments.
- **iOS lint** (SwiftLint): Not installed locally. CI runs it inside the swift:5.10 container. Not critical for local dev.
