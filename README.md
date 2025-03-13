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
  * maven-open-source-verify
    * Builds project with Maven
  * maven-open-source-increment-version-and-release-to-maven-central __Note: this job can leak secrets if run on untrusted code__
    * Increment version based on incrementing latest previous release
      * Add __[patch]__, __[minor]__ or __[major]__ to commit message to control increment (patch is the default)
    * Publish artifacts to Maven Central (Sonatype)
    * Creates and commits tag
  * maven-open-source-release-current-tag-to-maven-central __Note: this job can leak secrets if run on untrusted code__
    * Extracts version from the current tag
    * Publish artifacts to Maven Central (Sonatype)

#### Dry-run
Upload to Maven central without close/releasing staging repo: Set `autoReleaseAfterClose` to `false` in `nexus-staging-maven-plugin` configuration.

### Gradle
Build using Gradle wrapper.

Workflows:
 * gradle-open-source-verify
   * Builds project with gradle 
 * gradle-open-source-increment-version-and-release-to-maven-central __Note: this job can leak secrets if run on untrusted code__ 
   * Increment version based on incrementing latest previous release
     * Add __[patch]__, __[minor]__ or __[major]__ to commit message to control increment (patch is the default)
   * Publish artifacts to Maven Central (Sonatype)
   * Creates and commits tag
 * gradle-open-source-release-current-tag-to-maven-central __Note: this job can leak secrets if run on untrusted code__
   * Extracts version from the current tag
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

#### Dry-run
Upload to Maven central without close/releasing staging repo: Set `tasks` parameter `build publishToSonatype`.

