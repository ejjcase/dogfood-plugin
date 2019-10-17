# Plugin that eats its own dogfood cannot set group in gradle.properties

I believe this project demonstrates a bug in Gradle: a Gradle plugin project that applies an earlier version of itself must set its `group` property in `build.gradle`, not in `gradle.properties`.

Clone this project and run:

```bash
$ git checkout 1.0.0
$ ./gradlew publishToMavenLocal
```

This publishes the "dogfood" for later revisions to "eat".

It's okay for the plugin to apply an earlier version of itself:

```bash
$ git checkout works-okay
$ ./gradlew build
```

But if we start defining the project's `group` in `gradle.properties` instead of in `build.gradle`, we get an error:

```bash
$ git checkout broken
$ ./gradlew build

FAILURE: Build failed with an exception.

* Where:
Build file '/ssd/ejj/git/dogfood-plugin/build.gradle' line: 4

* What went wrong:
Could not apply requested plugin [id: 'dogfood.plugin.greeting', version: '1.0.0'] as it does not provide a plugin with id 'dogfood.plugin.greeting'. This is caused by an incorrect plugin implementation. Please contact the plugin author(s).
> Plugin with id 'dogfood.plugin.greeting' not found.
```

Setting the group in this way is fine if the plugin is not trying to use itself:

```bash
$ git checkout no-dogfood
$ ./gradlew publishToMavenLocal
```

This publishes a version of the plugin that looks just like 1.0.0 except for the version number.

Possibly Gradle is looking for the plugin in the project's `buildSrc/`, which doesn't exist.  But surely it shouldn't do that if the version number doesn't match.

