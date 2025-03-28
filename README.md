# scalatest-issue-2357

Reproduces nested tests behaviour when running ScalaTest via SBT Testing Interface.

A supplement to the ScalaTest [issue 2357](https://github.com/scalatest/scalatest/issues/2357).

Uses [Scala.js Gradle plugin](https://github.com/dubinsky/scalajs-gradle)
to run `ScalaTest` tests via `sbt.testing.Framework`.

Supplies `sbt.testing.TaskDef` that:
- asks to run `fullyQualifiedName="org.podval.tools.test.Nesting"`;
- as a suite (`selectors=[sbt.testing.SuiteSelector]`);
- with the class selected explicitly (`explicitlySpecified=true`);
- uses `ScalaTest`'s subclass fingerprint (`fingerprint=org.scalatest.Suite`).

To run the tests, execute the following command in the project's directory:
```shell
$ ./gradlew clean test --info
```

In the log, lines that mention `RunTestClassProcessor` show:
- what `sbt.testing.Task[Def]` is being executed;
- what `sbt.testing.Event`s are emitted by `ScalaTest`.

Log also contains `ScalaTest` output and summary.

Resulting test report can be found in `build/reports/tests/test/index.html`.

Tests can be run either on `Scala.js` (default) or on `JVM`;
to switch to running on `JVM` (disable `Scala.js`):
- in `gradle.properties`, uncomment `org.podval.tools.scalajs.disabled = true`;
- in `build.gradle`, comment out `node.version = '16.19.1'`;
- in `build.gradle`, comment out `implementation 'org.scalatest:scalatest_sjs1_3:3.2.19'`;
- in `build.gradle`, uncomment `implementation 'org.scalatest:scalatest_3:3.2.19'`.

When running on `Scala.js`, test cases of the `Nested` class ARE executed.

When running on `JVM`, test classes of the `Nested` class ARE NOT executed.

This inconsistency in behaviour is not really surprising,
since `ScalaTest` has two distinct implementations of the `sbt.testing.Framework` interface 
for [`Scala.js`](https://github.com/scalatest/scalatest/blob/main/js/core/src/main/scala/org/scalatest/tools/Framework.scala) and
for [`JVM`](https://github.com/scalatest/scalatest/blob/main/jvm/core/src/main/scala/org/scalatest/tools/Framework.scala).
