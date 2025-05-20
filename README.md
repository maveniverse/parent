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
using the [Sonatype Central Portal](https://central.sonatype.com/) service.

You need to have the following entry in your user `settings.xml`:

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

To perform Maven release invoke:
* `mvn release:prepare`
* `mvn release:perform`
* project will be staged to [Sonatype Central Portal](https://central.sonatype.com/publishing) (manual step: once validation passes, publish it)
* **don't forget to push git changes** once all done (`maven-release-plugin` is configured to not push them): `git push origin main --tags`.
