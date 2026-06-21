# BnPP_Index

Fresh Gradle 9.5.1 / Java project (Kotlin DSL). Single `org.example` package, one scaffold `Main.java` that does not compile as-is (`static void main()` lacks `String[] args` and uses a non-standard `IO` class). No tests yet.

## Commands

| Action | Command |
|--------|---------|
| Build | `gradlew build` |
| Test | `gradlew test` |
| Clean | `gradlew clean` |

## Key facts

- Gradle wrapper is present; use `gradlew.bat` on Windows, `gradlew` elsewhere.
- JUnit Jupiter 5 (BOM 6.0.0) is pre-configured as test dependency — just add test sources under `src/test/java/`.
- No application plugin or main class declared in build — `Main.java` has no `main(String[])` entry point and references an unknown `IO` class.
- No runtime dependencies beyond JUnit for testing.
- Single flat package `org.example` — no sub-packages.
- `build/` and `.gradle/` are gitignored.
