# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

ArcSeekBar is an Android library providing a single custom `View`: an arc-shaped, draggable seek bar (like `SeekBar` but drawn along a circular arc instead of a straight line). It supports gradient coloring, configurable arc width/open angle/rotation, a customizable thumb (fill/stroke/fill+stroke, with shadow), border stroke, and outer glow shadow.

This is a two-module Gradle project:
- `arc-seekbar/` — the library module (`com.android.library`), published as `com.gcssloop.widget:arc-seekbar`. All library logic lives here.
- `app/` — a demo/test app (`com.android.application`) that depends on `:arc-seekbar` and exercises it via `MainActivity`.

The project uses the legacy Android Support Library (not AndroidX) and an old Gradle/AGP toolchain (AGP 3.1.0, no Kotlin, no Gradle wrapper distribution URL check needed beyond what's in `gradle/wrapper`).

## Build commands

Run from the repository root using the Gradle wrapper:

```bash
./gradlew assembleDebug          # build the demo app + library
./gradlew :arc-seekbar:assemble  # build only the library module
./gradlew clean
./gradlew installDebug           # install demo app on a connected device/emulator
```

There are no unit or instrumented test source sets in either module currently — `app/build.gradle` declares `testImplementation`/`androidTestImplementation` (JUnit, Espresso) but no test classes exist. If tests are added, run them with `./gradlew test` (JVM unit tests) or `./gradlew connectedAndroidTest` (instrumented tests).

There is no lint/format tooling configured beyond the default Android Gradle plugin lint (`./gradlew lint`).

## Architecture

Everything of substance is in one file: `arc-seekbar/src/main/java/com/gcssloop/widget/ArcSeekBar.java`. Custom attributes are declared in `arc-seekbar/src/main/res/values/attrs.xml` under the `ArcSeekBar` styleable, prefixed `arc_*`.

Key mechanics to understand before modifying `ArcSeekBar`:

- **Single path, canvas rotation for orientation.** The entire arc is one `Path` built with `Path.addArc()` inside `onSizeChanged`. To make the arc's "opening" point in any direction, the `Canvas` itself is rotated by `mRotateAngle` in `onDraw` — the path's own coordinate system always starts the arc at angle `mOpenAngle / 2`. Any change to angle math must account for this: touch coordinates are in *screen* space, but the arc path is defined in *unrotated* space, so touch handling inverts the rotation via `mInvertMatrix` before comparing against path-space geometry.
- **Progress ↔ angle conversion.** Touch position → angle from center (`getAngle`) → angle relative to the rotated/opened arc (`getDiffAngle`) → progress fraction in `[0,1]` (`getCurrentProgress`) → actual value via `mMinValue`/`mMaxValue` (`getProgress`/`setProgress`). Thumb screen position is derived from the progress fraction via `PathMeasure.getPosTan()` (`computeThumbPos`), not recomputed from the angle directly.
- **Drag vs. click vs. anti-jump guard.** `onTouchEvent` only allows dragging (`mCanDrag`) if `ACTION_DOWN` lands within `1.5 * mThumbRadius` of the current thumb position (`judgeCanDrag`). A separate `GestureDetector` (`OnClickListener`) handles single taps that jump progress directly, but only if the tap lands inside `mArcRegion` (a `Region` derived from the paint's fill path, computed in `onSizeChanged`) — this is `isInArcProgress`. During drag, `mAllowTouchSkip` (currently always `false`, no public setter) prevents the progress from jumping by more than 0.5 in one move event, guarding against the 0.0/1.0 wraparound discontinuity inherent to a circular progress value.
- **Gradient coloring via `SweepGradient`.** `resetShaderColor()` maps the user-supplied `mArcColors` array onto gradient stop positions scaled to the arc's actual open angle (not the full circle), so colors always span exactly the visible arc regardless of `arc_open_angle`. `getColor(float)` independently recomputes the interpolated color at a given progress fraction for external callers (`getColor()`), by linearly interpolating RGB between the two bounding entries in `mArcColors` — keep this in sync with `resetShaderColor()`'s stop math if the gradient logic changes.
- **Colors can come from three places**: XML `app:arc_colors="@array/..."`, `setArcColors(int[])`, or `setArcColors(int colorArrayRes)` — all funnel through `getColorsByArrayResId`/`resetShaderColor`.
- **State restoration** only persists `mProgressPresent` (a float fraction, not the raw progress value) via `onSaveInstanceState`/`onRestoreInstanceState`, layered on top of `super` state via a `Bundle`.
- **Dimensions from attrs are in px, not dp**, converted once in `initAttrs`/`dp2px`; if adding new dimension attrs, follow the existing `ta.getDimensionPixelSize(..., dp2px(DEFAULT_...))` pattern.

## Making changes

- `minSdkVersion` is 14 — avoid APIs newer than that without a version check.
- The library targets plain `android.*`/`com.android.support:*` APIs, no AndroidX; don't introduce AndroidX dependencies into `arc-seekbar` without also migrating `app` and updating `build.gradle`/`gradle.properties` (no `android.useAndroidX` flag is currently set).
- Version/artifact coordinates for publishing live in `arc-seekbar/build.gradle` under the `ext { GROUP_ID / ARTIFACT_ID / VERSION }` block — bump `VERSION` there when cutting a release, and mirror it in `README.md`'s changelog section and the Gradle dependency snippet.
- The README (`README.md`) is written in Chinese and is the primary user-facing documentation for attributes and usage examples — keep it in sync with any new/changed/removed `arc_*` attributes or public methods on `ArcSeekBar`.
