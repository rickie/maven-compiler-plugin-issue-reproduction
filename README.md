## Unexpected dependency resolution of `annotationProcessorPaths`

This repository contains a minimal reproduction case for the following issue.

This issue relates to
[MCOMPILER-272](https://issues.apache.org/jira/browse/MCOMPILER-272).
That ticket improved annotation processor classpath construction, but as of
version 3.10.1 the constructed classpath is still highly unintuitive.

In a nutshell, the generated annotation processor classpath does not match
Maven's "general" dependency resolution logic, leading to (at least) the
following issues:
- The classpath may contain multiple versions of the same dependency.
- Indirect dependencies may take precedence over explicitly declared
  dependencies.

### Reproduction case

Consider the `pom.xml` with a `maven-compiler-plugin` configuration as
follows:
```xml
<annotationProcessorPaths>
    <path>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>1.0</version>
    </path>
    <path>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>31.0.1-jre</version>
    </path>
</annotationProcessorPaths>
```

Note that `com.google.auto.service:auto-service` 1.0 depends on Guava
30.1.1-jre, which is a different version of Guava than the one explicitly
specified.

To see the issue, checkout the repository and run the following command:
`mvn clean install -X | grep -oP '(?<=\-processorpath )\S+' | xargs -d: -l1`

Prior to `maven-compiler-plugin` version 3.9.0 this would output:
```
/home/user/.m2/repository/com/google/auto/service/auto-service/1.0/auto-service-1.0.jar
/home/user/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar
/home/user/.m2/repository/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
/home/user/.m2/repository/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
/home/user/.m2/repository/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar
/home/user/.m2/repository/org/checkerframework/checker-qual/3.12.0/checker-qual-3.12.0.jar
/home/user/.m2/repository/com/google/errorprone/error_prone_annotations/2.7.1/error_prone_annotations-2.7.1.jar
/home/user/.m2/repository/com/google/j2objc/j2objc-annotations/1.3/j2objc-annotations-1.3.jar
/home/user/.m2/repository/com/google/auto/service/auto-service-annotations/1.0/auto-service-annotations-1.0.jar
/home/user/.m2/repository/com/google/auto/auto-common/1.0/auto-common-1.0.jar
```

As of version 3.9.0+ this outputs:
```
/home/user/.m2/repository/com/google/auto/service/auto-service/1.0/auto-service-1.0.jar
/home/user/.m2/repository/com/google/auto/service/auto-service-annotations/1.0/auto-service-annotations-1.0.jar
/home/user/.m2/repository/com/google/auto/auto-common/1.0/auto-common-1.0.jar
/home/user/.m2/repository/com/google/guava/guava/30.1.1-jre/guava-30.1.1-jre.jar
/home/user/.m2/repository/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
/home/user/.m2/repository/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
/home/user/.m2/repository/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar
/home/user/.m2/repository/org/checkerframework/checker-qual/3.8.0/checker-qual-3.8.0.jar
/home/user/.m2/repository/com/google/errorprone/error_prone_annotations/2.5.1/error_prone_annotations-2.5.1.jar
/home/user/.m2/repository/com/google/j2objc/j2objc-annotations/1.3/j2objc-annotations-1.3.jar
/home/user/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar
/home/user/.m2/repository/org/checkerframework/checker-qual/3.12.0/checker-qual-3.12.0.jar
/home/user/.m2/repository/com/google/errorprone/error_prone_annotations/2.7.1/error_prone_annotations-2.7.1.jar
```

Note the following:
- Some dependencies are duplicated, with _different versions_ listed
  (`guava`, `checker-qual`, and `error_prone_annotations`).
- Even though the `pom.xml` explicitly declares a dependency on Guava
  31.0.1-jre, the classpath lists version 30.1.1-jre first, meaning that de
  facto the latter will be used.

In practice this means that:
1. One cannot rely on "common Maven dependency resolution knowledge".
2. One must be very careful with the order in which dependencies are listed.
3. The need to carefully order the dependencies may mean that a certain
   configuration cannot be factored out to a Maven profile.

Ideally `annotationProcessorPaths` follows the same dependency resolution logic
as "regular" project dependencies. That is, ideally the classpath would be
constructed as follows:
```
/home/user/.m2/repository/com/google/auto/service/auto-service/1.0/auto-service-1.0.jar
/home/user/.m2/repository/com/google/auto/service/auto-service-annotations/1.0/auto-service-annotations-1.0.jar
/home/user/.m2/repository/com/google/auto/auto-common/1.0/auto-common-1.0.jar
/home/user/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar
/home/user/.m2/repository/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
/home/user/.m2/repository/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
/home/user/.m2/repository/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar
/home/user/.m2/repository/org/checkerframework/checker-qual/3.12.0/checker-qual-3.12.0.jar
/home/user/.m2/repository/com/google/errorprone/error_prone_annotations/2.7.1/error_prone_annotations-2.7.1.jar
/home/user/.m2/repository/com/google/j2objc/j2objc-annotations/1.3/j2objc-annotations-1.3.jar
```
