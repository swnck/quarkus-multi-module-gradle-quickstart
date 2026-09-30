# Quarkus Multi-Module Gradle Quickstart

An example project demonstrating how to structure a [Quarkus](https://quarkus.io/) application as a
**Gradle multi-module build** — with a plain Java library module consumed by a Quarkus application module.

## What this project shows

- How to split a Quarkus application into multiple Gradle modules
- How to apply the Quarkus Gradle plugin to only one module (the runnable application)
- How to share code between modules via `project(...)` dependencies
- How the common Quarkus commands (`quarkusDev`, `build`, native build) behave in a multi-module setup

## Project structure

```
quarkus-multi-module-gradle-quickstart
├── quickstart-module-common    Plain Java library module (no Quarkus, no CDI)
│   └── src/main/java/com/swnck/multimodule/RestUtil.java
├── quickstart-module-rest      Quarkus application module (depends on quickstart-module-common)
│   └── src/main/java/com/swnck/multimodule/ExampleResource.java
├── settings.gradle             Declares the modules and the Quarkus plugin version
├── build.gradle                Shared configuration via allprojects/subprojects
└── gradle.properties           Quarkus platform and plugin versions
```

| Module                    | Description                                                                 |
| ------------------------- | --------------------------------------------------------------------------- |
| `quickstart-module-common` | Shared library with plain Java code, usable by any other module.            |
| `quickstart-module-rest`   | The Quarkus application exposing a REST endpoint at `GET /hello`.           |

## How the multi-module setup works

**1. Declare the modules in `settings.gradle`**

```gradle
rootProject.name = 'quarkus-multi-module-gradle-quickstart'

include 'quickstart-module-rest'
include 'quickstart-module-common'
```

**2. Share common configuration in the root `build.gradle`**

Group, version and repositories are defined once for all modules, Java settings for all subprojects:

```gradle
allprojects {
    group = 'com.swnck.multimodule'
    version = '1.0-SNAPSHOT'

    repositories {
        mavenCentral()
        mavenLocal()
    }
}
```

**3. Apply the Quarkus plugin only to the application module**

`quickstart-module-common/build.gradle` is a plain Java module:

```gradle
plugins {
    id 'java'
}
```

`quickstart-module-rest/build.gradle` applies the Quarkus plugin and declares the dependency on the common module:

```gradle
plugins {
    id 'java'
    id 'io.quarkus'
}

dependencies {
    implementation enforcedPlatform("${quarkusPlatformGroupId}:${quarkusPlatformArtifactId}:${quarkusPlatformVersion}")
    implementation 'io.quarkus:quarkus-rest'
    implementation 'io.quarkus:quarkus-arc'

    implementation project(':quickstart-module-common')
}
```

The REST resource in `quickstart-module-rest` can then simply use classes from `quickstart-module-common`:

```java
@Path("/hello")
public class ExampleResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return RestUtil.error(); // from quickstart-module-common
    }
}
```

## Prerequisites

- JDK 26 (configured in the root `build.gradle`, adjust `sourceCompatibility`/`targetCompatibility` if you use an older JDK)
- No local Gradle installation required — use the included Gradle Wrapper (`./gradlew`)

## Running the application in dev mode

Run the application in dev mode with live coding:

```shell
./gradlew quarkusDev
```

Or target the application module explicitly:

```shell
./gradlew :quickstart-module-rest:quarkusDev
```

The endpoint is available at <http://localhost:8080/hello>.

> **_NOTE:_** Quarkus ships with a Dev UI, available in dev mode only at <http://localhost:8080/q/dev/>.

## Running the tests

```shell
./gradlew test
```

## Packaging and running the application

Package the application with:

```shell
./gradlew build
```

In a multi-module build the output is created in the **application module's** build directory, not in the
project root. The build produces `quickstart-module-rest/build/quarkus-app/quarkus-run.jar`.

Be aware that it's not an _über-jar_ — the dependencies are copied into the
`quickstart-module-rest/build/quarkus-app/lib/` directory.

Run the packaged application with:

```shell
java -jar quickstart-module-rest/build/quarkus-app/quarkus-run.jar
```

If you want to build an _über-jar_, execute:

```shell
./gradlew build -Dquarkus.package.jar.type=uber-jar
```

The application, packaged as an _über-jar_, is then runnable with
`java -jar quickstart-module-rest/build/*-runner.jar`.

## Creating a native executable

Create a native executable with:

```shell
./gradlew build -Dquarkus.native.enabled=true
```

Or, if you don't have GraalVM installed, run the native build in a container:

```shell
./gradlew build -Dquarkus.native.enabled=true -Dquarkus.native.container-build=true
```

You can then execute the native binary from `quickstart-module-rest/build/`.

For more details on building native executables, see <https://quarkus.io/guides/gradle-tooling>.

## Tech stack

- [Quarkus](https://quarkus.io/) 3.x — Supersonic Subatomic Java Framework
- Gradle (Kotlin-free `build.gradle` Groovy DSL) with the Gradle Wrapper
- Java 26 (configurable in the root `build.gradle`)

## License

This project is provided as an example — feel free to use it as a starting point for your own applications.
