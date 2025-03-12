# abt-gha-public

This repository contains workflows used to verify, build and deploy artifacts in projects used by
account based ticketing (ABT) team at Entur.

## Maven
Builder using Maven (i.e. not wrapper)

Workflows:
  * validate-jar-maven-sona.yml
    * Builds project with Maven
  * maven-release-sona.yml
    * Increment version based on incrementing latest previous release
    * Publish artifacts to Maven Central (Sonatype)
    * Creates tag

## Gradle
Build using Gradle wrapper.

Workflows: 
 * validate-jar-gradle-sona.yml
   * Builds project with gradle 
 * gradle-release-sona.yml
   * Increment version based on incrementing latest previous release
     * Add [patch], [minor] or [major] to commit message to control increment (patch is the default)
   * Publish artifacts to Maven Central (Sonatype)
   * Creates tag 
 * gradle-release-tag-sona.yml
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

