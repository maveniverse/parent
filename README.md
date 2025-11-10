# Maveniverse parent POM

Common build configuration.

## Requirements

Projects using this parent receive following requirements:

Build time:
* Java 21+
* Maven 3.9+ (use of Maven Wrapper in project is recommended)

Runtime:
* Java 8+
* for Maven Plugins being built: Maven 3.6.3+

Naturally, these can be overridden in projects.

## Release procedure

Release uses [Maveniverse Njord](https://github.com/maveniverse/njord) to publish to [Maven Central](https://repo.maven.apache.org/maven2/)
using the [Sonatype Central Portal](https://central.sonatype.com/) service. There are two ways to release, each one
explained before.

Requirements:
* Sonatype Central Portal account
* Access to `eu.maveniverse` namespace
* Publishing tokens generated

### Release from (user) workstation

This is how releases were done so far. You need to have the following entry in your user `settings.xml`:

```xml
    <server>
      <id>sonatype-central-portal</id>
      <username>$TOKEN1</username>
      <password>$TOKEN2</password>
      <configuration>
        <njord.publisher>sonatype-cp</njord.publisher>
        <njord.releaseUrl>njord:template:release-sca</njord.releaseUrl>
        <njord.snapshotUrl>njord:template:snapshot-sca</njord.snapshotUrl>
      </configuration>
    </server>
```

If you have multiple GPG keys make sure the proper one is used. You can bind it for example to the `maveniverse-release` profile like this
in your `settings.xml`

```xml
    <profile>
        <id>maveniverse-release</id>
        <properties>
            <!-- which GPG key to take (the one for Apache), https://maven.apache.org/plugins/maven-gpg-plugin/sign-mojo.html#keyname -->
            <gpg.keyname><short-id></gpg.keyname>
        </properties>
    </profile>
```


To perform Maven release invoke:
* `mvn release:prepare`
* `mvn release:perform`
* project will be staged to [Sonatype Central Portal](https://central.sonatype.com/publishing); **manual step**: once validation passes, publish it
* **don't forget to push git changes** once all done (`maven-release-plugin` is configured to not push them): `git push origin main --tags`.

### Release from Action

This is opt-in option for projects under Maveniverse organization. This needs a few more preparation, but at the end
you end up with "push button" release that happens on GitHub.

Requirements:
* Add required secrets to given repository: `MAVEN_USER` , `MAVEN_PASSWORD` (Central Portal token1 and token2), `GPG_SIGNING_KEY` (gpg secret key with armor: `gpg --armor --export-secret-key me@domain.eu`), `GPG_PASSPHRASE` (gpg passphrase), optionally `GPG_KEY_FINGERPRINT` (if using subkey for signing).
* Copy [release-settings.xml](.github/release-settings.xml) file (as is) to `.github/release-settings.xml` path in given repository.
* Make sure your POM/scm/developerConnection is `https://`.
* Add `release.yml` workflow as this
```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      version:
        description: The version to release (will be inferred from the pom version by default).
        required: false
      next-version:
        description: The next development version (will be inferred from the pom version by default).
        required: false
      java-version:
        description: The java version to use
        required: false
        default: 21
      distribution:
        description: The java distribution to use.
        required: false
        default: temurin

jobs:
  build:
    name: Release
    uses: maveniverse/parent/.github/workflows/release.yml@release-34
    secrets: inherit
    with:
      version: ${{ github.event.inputs.version }}
      next-version: ${{ github.event.inputs.next-version }}
      java-version: ${{ github.event.inputs.java-version }}
      distribution: ${{ github.event.inputs.distribution }}
```

Note: POM parent version and `@release-XX` suffix of reused workflows **should match**.

Once all done, to perform automated release, just fire the `Release` workflow with required parameters. Once the
action finishes, project will be staged to [Sonatype Central Portal](https://central.sonatype.com/publishing); **manual step**: once validation passes, 
publish it.
