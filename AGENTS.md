# AGENTS.md

## Cursor Cloud specific instructions

This is a demo monorepo with two independent projects: an Android app (`android/`) and an iOS/Swift package (`ios/`). There are no backend services, databases, or running processes — it is purely build-and-test.

### Services

| Project | Lint | Build | Test |
|---------|------|-------|------|
| Android (`android/`) | `./gradlew lint --no-daemon` | `./gradlew assembleDebug --no-daemon` | `./gradlew test --no-daemon` |
| iOS (`ios/`) | `swiftlint lint --strict Sources/ Tests/` | `swift build` | `swift test` |

### Environment prerequisites

- **JDK 17** must be the active Java version. Set `JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64` (persisted in `~/.bashrc`).
- **Android SDK** is installed at `/opt/android-sdk`. `ANDROID_HOME` must be set (persisted in `~/.bashrc`).
- **Swift 5.10** is installed at `/usr/local/bin/swift`.
- **SwiftLint 0.57.0** is installed at `/usr/local/bin/swiftlint`.
- **Gradle 8.5** is installed at `/opt/gradle-8.5/bin/gradle` (also symlinked to `/usr/local/bin/gradle`).

### Gotchas

- The `gradle-wrapper.jar` is **not committed** to the repo. Before running any `./gradlew` command in `android/`, you must first run `gradle wrapper` (using the system Gradle) in the `android/` directory to generate it. The update script handles this automatically.
- All Gradle commands should use `--no-daemon` to avoid long-lived daemon processes in ephemeral environments.
- The iOS project uses Swift Package Manager on Linux (no Xcode required). CI runs on `ubuntu-latest` with a `swift:5.10` container.
- External services (Aviator, SonarCloud, Snyk) are optional and only relevant in CI. No secrets are needed for local development.
