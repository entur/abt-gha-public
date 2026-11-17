# abt-gha-public

This repository contains workflows used to verify, build and deploy artifacts in projects used by  account based ticketing (ABT) team at Entur.

# Versioning strategies
These workflows support various versioning strategies:

 * release tag
   * extracts version from the tag name
   * passes version to build via a command-line parameter
   * does not modify any files in the repository
 * release branch: determine version from a previous tag
   * extract and increment version from latest tagged version
     * i.e. release-1.2.3 -> 1.2.3
   * increment controlled via commit message
     * __[patch]__: 1.2.3 -> 1.2.4
     * __[minor]__: 1.2.3 -> 1.3.0
     * __[major]__: 1.2.3 -> 2.0.0
   * passes version to build via a command-line parameter
   * does not commit any files, only creates a new tag

### Maven
Builder using Maven (i.e. not wrapper)

Workflows:
  * [maven-open-source-verify](.github/workflows/maven-open-source-verify.yml)
    * Builds project with Maven
  * [maven-open-source-increment-version-and-release-to-maven-central](.github/workflows/maven-open-source-increment-version-and-release-to-maven-central.yml) __Note: this job can leak secrets if run on untrusted code__
    * Increment version based on incrementing latest previous release
      * Add __[patch]__, __[minor]__ or __[major]__ to commit message to control increment (patch is the default)
    * Publish artifacts to Maven Central (Sonatype)
    * Adds tag
  * [maven-open-source-release-tag-to-maven-central](.github/workflows/maven-open-source-release-tag-to-maven-central.yml) __Note: this job can leak secrets if run on untrusted code__
    * Extracts version from the current tag, or from an input parameter.
    * Publish artifacts to Maven Central (Sonatype)

#### Maven dry-run
Upload to Maven central without close/releasing staging repo: Set `autoReleaseAfterClose` to `false` in `nexus-staging-maven-plugin` configuration.

### Gradle
Build using Gradle wrapper.

Workflows:
 * [gradle-open-source-verify](.github/workflows/gradle-open-source-verify.yml)
   * Builds project with gradle 
 * [gradle-open-source-increment-version-and-release-to-maven-central](.github/workflows/gradle-open-source-increment-version-and-release-to-maven-central.yml) __Note: this job can leak secrets if run on untrusted code__ 
   * Increment version based on incrementing latest previous release
     * Add __[patch]__, __[minor]__ or __[major]__ to commit message to control increment (patch is the default)
   * Publish artifacts to Maven Central (Sonatype)
   * Adds tag
 * [gradle-open-source-release-tag-to-maven-central](.github/workflows/gradle-open-source-release-tag-to-maven-central.yml) __Note: this job can leak secrets if run on untrusted code__
   * Extracts version from the current tag, or from an input parameter.
   * Publish artifacts to Maven Central (Sonatype)

Note that projects which use the release flows must propagate the command-line `version` property to artifact version in the Gradle publishing configuration, i.e. like this:

```groovy
publications {
    maven(MavenPublication) {
        from components.java

        groupId = "$group"
        artifactId = project.name

        // set version from command line version, otherwise use a snapshot version for local publishing
        version = project.hasProperty('version') ? "$version" : "0.0.0-SNAPSHOT"

        // ...
    }
}
```

#### Gradle dry-run
Upload to Maven central without close/releasing staging repo: Set `tasks` parameter `build publishToSonatype`.

# Safe Java version
The reusable workflows set up Java through the [`safe-java-version`](.github/actions/safe-java-version/action.yml) action before `actions/setup-java`. It translates the requested Java version and the distribution into a version that is safe to use (CVE-2026-91403, eMudhra CA compromise): a bare major becomes the patched build, a full version is passed through. Apps can keep `java-version: 25` and get the patched JDK.

| Distribution | 17 | 21 | 25 |
|---|---|---|---|
| `liberica` | `17.0.21+0.3.5` | `21.0.13+0.3.5` | `25.0.5+0.3.5` |
| `temurin` | no patched build | no patched build | no patched build |

Temurin has no patched build yet (the newest, 17.0.21+9 / 21.0.13+9.0.LTS / 25.0.5+8.0.LTS, were released 2026-10-22, before the compromise). The version is passed through and the step warns; `fail-if-unpatched: true` fails the step instead. Other distributions are passed through with a warning.

In your own workflow:

```yaml
- id: java
  uses: entur/abt-gha-public/.github/actions/safe-java-version@v2
  with:
    java-version: 25
    distribution: liberica
- uses: actions/setup-java@d1374f8ac74474200d16ef8ecee22c7ae7a51880 # v6.3.0, at least 6.2.0
  with:
    java-version: ${{ steps.java.outputs.java-version }}
    distribution: ${{ steps.java.outputs.distribution }}
```

`actions/setup-java` must be at least v6.2.0: it spells BellSoft's `25.0.5.0.3+5` as `25.0.5+0.3.5`, older releases do not. Outputs: `java-version`, `distribution`, `patched`. The table is the `pin_for` function in the action. The same action exists in `entur/abt-gha`; keep them identical.

# Workflow usage
Workflow, java version and runner will naturally change from time to time, so you might consider to

 * use the workflow full version / commit hash as version
 * specify `java-version` and `runs-on` parameters

Also consider to selectively propagate secrets if necessary.

## Pull-requests
Does not use secrets.

Maven:
 ```yaml
 name: PR verify maven
 on:
   pull_request:
     types:
       - synchronize
       - opened

 jobs:
   pr-verify-maven:
     uses: entur/abt-gha-public/.github/workflows/maven-open-source-verify.yml@vx.x.x
 ```

Gradle:

 ```yaml
 name: pr verify gradle
 on:
   pull_request:
     types:
       - synchronize
       - opened

 jobs:
   pr-verify-gradle:
     uses: entur/abt-gha-public/.github/workflows/gradle-open-source-verify.yml@vx.x.x
 ```
