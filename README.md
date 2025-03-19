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


## Manual release from main with version increment
Uses secrets.

Maven:

```yaml
name: deploy main maven manual
on:
  workflow_dispatch:
    inputs:
      version-increment:
        description: 'Version-increment'
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  deploy-tag-maven:
    uses: entur/abt-gha-public/.github/workflows/maven-open-source-increment-version-and-release-to-maven-central.yml@vx.x.x
    secrets: inherit
    with:
      version-increment: ${{ inputs.version-increment }}

```

Gradle:

```yaml
name: deploy tag gradle manual
on:
  workflow_dispatch:
    inputs:
      version-increment:
        description: 'Version-increment'
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  deploy-tag-gradle:
    uses: entur/abt-gha-public/.github/workflows/gradle-open-source-increment-version-and-release-to-maven-central.yml@vx.x.x
    secrets: inherit
    with: 
      version-increment: ${{ inputs.version-increment }}
```

## Release on tag
Uses secrets.

Maven:

 ```yaml
 name: deploy tag maven
 on:
   push:
     tags:
       - '*'

 jobs:
   deploy-tag-maven:
     uses: entur/abt-gha-public/.github/workflows/maven-open-source-release-tag-to-maven-central.yml@vx.x.x
     secrets: inherit
 ```

Gradle:

 ```yaml
 name: deploy tag gradle
 on:
   push:
     tags:
       - '*'

 jobs:
   deploy-tag-gradle:
     uses: entur/abt-gha-public/.github/workflows/gradle-open-source-release-tag-to-maven-central.yml@vx.x.x
     secrets: inherit
 ```

## Release on main
Uses secrets.

Maven:

 ```yaml
name: deploy main maven
on:
  push:
    branches:
      - main

jobs:
  deploy-main-maven:
    if: "!contains(github.event.head_commit.message, '[skip-release]')"
    uses: entur/abt-gha-public/.github/workflows/maven-open-source-increment-version-and-release-to-maven-central.yml@vx.x.x
    secrets: inherit
 ```

Gradle:

 ```yaml
name: deploy main gradle
on:
  push:
    branches:
      - main

jobs:
  deploy-main-gradle:
    if: "!contains(github.event.head_commit.message, '[skip-release]')"
    uses: entur/abt-gha-public/.github/workflows/gradle-open-source-increment-version-and-release-to-maven-central.yml@vx.x.x
    secrets: inherit
 ```