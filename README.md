# maven-compiler-plugin-issue-reproduction
Simple reproduction case for an issue with the `annotationProcessorPaths` and Maven profiles.

To see the issue, run the following command:
`mvn clean install -Pguava -X | grep -oP '\-processorpath \S+' | xargs -d: -l1`

To see how this behaved before `maven-compiler-plugin` version `3.9.0`, go to `example/pom.xml` and change the version from `3.9.0` to `3.8.1`.
Re-run the command and compare the output.

### Output

This is the output of the previously mentioned command when using `maven-compiler-plugin` version `3.9.0`:

>  -processorpath /home/rick/.m2/repository/com/google/auto/service/auto-service/1.0/auto-service-1.0.jar
/home/rick/.m2/repository/com/google/auto/service/auto-service-annotations/1.0/auto-service-annotations-1.0.jar
/home/rick/.m2/repository/com/google/auto/auto-common/1.0/auto-common-1.0.jar
/home/rick/.m2/repository/com/google/guava/guava/30.1.1-jre/guava-30.1.1-jre.jar
/home/rick/.m2/repository/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
/home/rick/.m2/repository/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
/home/rick/.m2/repository/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar
/home/rick/.m2/repository/org/checkerframework/checker-qual/3.8.0/checker-qual-3.8.0.jar
/home/rick/.m2/repository/com/google/errorprone/error_prone_annotations/2.5.1/error_prone_annotations-2.5.1.jar
/home/rick/.m2/repository/com/google/j2objc/j2objc-annotations/1.3/j2objc-annotations-1.3.jar
/home/rick/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar
/home/rick/.m2/repository/org/checkerframework/checker-qual/3.12.0/checker-qual-3.12.0.jar
/home/rick/.m2/repository/com/google/errorprone/error_prone_annotations/2.7.1/error_prone_annotations-2.7.1.jar


This is the output when using `maven-compiler-plugin` version `3.8.1`:

> -processorpath /home/rick/.m2/repository/com/google/auto/service/auto-service/1.0/auto-service-1.0.jar
/home/rick/.m2/repository/com/google/guava/guava/31.0.1-jre/guava-31.0.1-jre.jar
/home/rick/.m2/repository/com/google/guava/failureaccess/1.0.1/failureaccess-1.0.1.jar
/home/rick/.m2/repository/com/google/guava/listenablefuture/9999.0-empty-to-avoid-conflict-with-guava/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
/home/rick/.m2/repository/com/google/code/findbugs/jsr305/3.0.2/jsr305-3.0.2.jar
/home/rick/.m2/repository/org/checkerframework/checker-qual/3.12.0/checker-qual-3.12.0.jar
/home/rick/.m2/repository/com/google/errorprone/error_prone_annotations/2.7.1/error_prone_annotations-2.7.1.jar
/home/rick/.m2/repository/com/google/j2objc/j2objc-annotations/1.3/j2objc-annotations-1.3.jar
/home/rick/.m2/repository/com/google/auto/service/auto-service-annotations/1.0/auto-service-annotations-1.0.jar
/home/rick/.m2/repository/com/google/auto/auto-common/1.0/auto-common-1.0.jar

