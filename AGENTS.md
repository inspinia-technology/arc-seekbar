# Repository Guidelines

## Project Structure & Module Organization

This is a Gradle-based Android project with two modules:

- `arc-seekbar/` contains the reusable library. Its main implementation is `src/main/java/com/gcssloop/widget/ArcSeekBar.java`; custom XML attributes and default colors live under `src/main/res/values/`.
- `app/` is the demonstration application. Use `MainActivity.java` and `res/layout/activity_main.xml` to exercise library behavior.
- `pics/` stores screenshots and animations referenced by `README.md`.
- Root Gradle files configure shared repositories, Android tooling, and the wrapper.

Keep reusable behavior and public resources in `arc-seekbar`; demo-only code and assets belong in `app`.

## Build, Test, and Development Commands

Use the checked-in Gradle wrapper from the repository root:

- `./gradlew assembleDebug` builds debug variants for both modules.
- `./gradlew :app:installDebug` installs the demo on a connected device or emulator.
- `./gradlew :arc-seekbar:assembleRelease` produces the library AAR.
- `./gradlew test` runs local JVM unit tests.
- `./gradlew connectedAndroidTest` runs instrumented tests on a device or emulator.
- `./gradlew lint` runs Android lint; `./gradlew clean` removes generated build output.

Configure the Android SDK through `local.properties`; never commit that file.

## Coding Style & Naming Conventions

Follow the existing Java and Android XML style: four-space indentation, braces on the same line, and one top-level class per file. Use `PascalCase` for classes, `camelCase` for methods and locals, `UPPER_SNAKE_CASE` for constants, and the existing `mFieldName` convention for instance fields. Resource and custom attribute names use lowercase `snake_case` (for example, `arc_thumb_radius`). Preserve the `com.gcssloop` package hierarchy. No formatter is configured, so keep changes consistent with adjacent code and run lint before submitting.

## Testing Guidelines

JUnit 4, AndroidJUnitRunner, and Espresso dependencies are configured in `app/build.gradle`, although no tests are currently checked in. Add JVM tests under `app/src/test/java/` and device tests under `app/src/androidTest/java/`; name test classes after the subject, such as `ArcSeekBarTest`. Cover progress bounds, XML attributes, saved state, and touch callbacks when changing widget behavior. Manually verify visual or gesture changes in the demo app.

## Commit & Pull Request Guidelines

History uses short release messages (`v1.7.0`) and concise scoped maintenance messages such as `build(macos): ...`. Prefer an imperative `<type>(<scope>): summary` for normal changes and reserve version-only subjects for releases. Keep commits focused. Pull requests should explain behavior changes, list validation commands, link related issues, and include screenshots or recordings for UI changes. Call out public API or custom-attribute compatibility impacts.
