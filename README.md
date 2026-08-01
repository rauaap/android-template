# Android container build template

A minimal Android app skeleton with a fully containerized, CLI-driven Gradle
build. No JDK, Android SDK, or Gradle needed on the host — everything runs
inside a podman container defined by the `Containerfile`. The only host
dependency is `podman` (and `adb`, if you want to install on a device).

## Layout

```
Containerfile          Fedora + JDK 21 + Android SDK + Gradle toolchain
Makefile               build / shell / install targets (podman wrapper)
build.gradle           root project — pins the Android Gradle Plugin version
settings.gradle        project name + module list
app/                   the application module (one-activity skeleton)
```

## Build

Build the toolchain image once:

```sh
make image
```

Build a debug APK (rebuilds the image if needed):

```sh
make debug
```

Output: `app/build/outputs/apk/debug/app-debug.apk`

Other targets:

```sh
make release            # assembleRelease
make clean              # gradle clean
make gradle ARGS="tasks"   # run any gradle task in the container
make shell              # interactive shell inside the build container
```

The Gradle cache is persisted in a named volume (`android-gradle-cache`) so
incremental builds and the debug keystore survive between runs.

## Install on a device

The build stays containerized; only `adb` runs on the host:

```sh
sudo dnf install android-tools     # Fedora
make install                       # adb install -r the debug APK
```

## Starting a new app from this template

Replace each placeholder value below. The package id (`com.example.app`) must be
changed in all four of its locations together — `namespace`, `applicationId`, the
source directory, and the `package` declaration.

| Placeholder | Value | Where |
|-------------|-------|-------|
| Package id | `com.example.app` | `app/build.gradle` (`namespace` + `applicationId`), source dir `app/src/main/java/com/example/app/`, `package` line in `MainActivity.java` |
| Project name | `AndroidApp` | `settings.gradle` (`rootProject.name`) |
| App label | `AndroidApp` | `app/src/main/res/values/strings.xml` (`app_name`) |
| Image / cache names | `android-builder`, `android-gradle-cache` | `Makefile` (`IMAGE`, `GRADLE_CACHE`) — optional |
| Launcher icon | placeholder "A" art on a `#78909C` backdrop | `app/src/main/res/drawable/ic_launcher_foreground.xml` (the mark) and `ic_launcher_background.xml` (the backdrop) — replace both; the adaptive-icon wrappers in `mipmap-anydpi-v26/` reference them by name and need no changes |

The app colors in `app/src/main/res/values/colors.xml` — `accent` `#D97757` and
`background` `#1A1820` — are **not** placeholders. They are the shared brand
palette and are meant to carry over to every app built from this template. Use
`@color/accent` and `@color/background` for new UI rather than introducing
another palette.

Bump SDK / build-tools / Gradle versions in the `Containerfile` `ARG`s if needed.

## Converting this README

This README describes the template, not the app you are building. Converting it
is the last step of starting a new app — leaving it in place is how a repo ends
up documenting a scaffold nobody is using anymore.

Rewrite the top of the file to describe the actual app: its title, what it does,
and anything a reader needs in order to use it. Delete the template-only
material — the `Layout` section, `Starting a new app from this template`, and
this section.

The build tooling does not change during conversion, so its documentation
survives the rewrite. Move `Build`, `Install on a device`, and `Notes` to the
**bottom** of the file, below the app's own documentation, collected under a
single heading such as `## Building`. They are reference material for whoever
works on the app, not the first thing a reader should meet.

## Notes

- **Container-only by design.** There is no Gradle wrapper (`gradlew`); the
  pinned Gradle version lives solely in the `Containerfile`. Build through
  `make`, not on the host.
- SDK level, build-tools, and Gradle versions are all `ARG`s at the top of the
  `Containerfile` — change them in one place.
