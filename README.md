# Using GraalVM Enterprise in OCI DevOps to build a Micronaut REST App

This sample shows how to use GraalVM Enterprise Edition in OCI DevOps build pipelines to build a simple Micronaut hello world REST application. You can use this approach to build any high-performance Java application with Micronaut, GraalVM Enterprise and OCI DevOps.

## What is GraalVM?

GraalVM is a high-performance JDK distribution that can accelerate any Java workload running on the HotSpot JVM.

GraalVM Native Image ahead-of-time compilation enables you to build lightweight Java applications that are smaller, faster, and use less memory and CPU. At build time, GraalVM Native Image analyzes a Java application and its dependencies to identify just what classes, methods, and fields are absolutely necessary and generates optimized machine code for just those elements.

GraalVM Enterprise Edition is available for use on Oracle Cloud Infrastructure (OCI) at no additional cost.

## What is Micronaut?

Micronaut is a modern, JVM-based framework to build modular, easily testable microservice and serverless applications. By avoiding runtime reflection in favor of annotation processing, Micronaut improves the Java-based development experience by detecting errors at compile time instead of runtime, and improves Java-based application start time and memory footprint. Micronaut includes a persistence framework called Micronaut Data that precomputes your SQL queries at compilation time making it a great fit for working with databases like MySQL, Oracle Autonomous Database, etc.

Micronaut uses GraalVM Native Image to build lightweight Java applications that use less memory and CPUs, are smaller and faster because of an advanced ahead-of-time compilation technology.

## What is OCI DevOps?

OCI DevOps services provide a continuous integration and deployment (CI/CD) platform for developers to automate the build, test, and deployment of software applications on Oracle Cloud.

OCI DevOps build and deployment pipelines reduce change-driven errors and decrease the time customers spend on building and deploying software. The service also provides private Git repositories to store your code and supports connections to external code repositories. 


## Updating the Build Specification File

To install and use GraalVM Enterprise in the DevOps build pipeline, update your build specification file as follows:

1. Add the following command to install the one or more required GraalVM Enterprise components. For example, this command installs Native Image along with the Java Development Kit (JDK) and other necessary dependencies.

    ```shell
    steps:
      - type: Command
        name: "Install the latest GraalVM Enterprise 22.x for Java 17 - JDK and Native Image"
        command: |
          yum -y install graalvm22-ee-17-native-image
    ```

2. Set the JAVA_HOME environment variable.

    ```shell
    env:
      variables:
        "JAVA_HOME" : "/usr/lib64/graalvm/graalvm22-ee-java17"
    ```

3. Set the PATH environment variable.

    ```shell
    env:
      variables:
        # PATH is a reserved variable and cannot be defined as a variable.
        # PATH can be changed in a build step and the change is visible in subsequent steps.
    
    steps:
      - type: Command
        name: "Set the PATH here"
        command: |
          export PATH=$JAVA_HOME/bin:$PATH
    ```

4. Build a native executable for your Micronaut application.

    ```shell
    steps:
      - type: Command
        name: "Example 2: Build a native executable with the installed GraalVM Enterprise 22.x for Java 17 - Native Image"
        command: |
          ./mvnw --no-transfer-progress package -Dpackaging=native-image
    ```

5. The native executable file is available under `target/MnHelloRest`.

    ```shell
    outputArtifacts:
      - name: app_native_executable
        type: BINARY
        location: target/MnHelloRest
    ```

6. Package the native executable in a lightweight distroless runtime container image.

    ```shell
    steps:
      - type: Command
        name: "Package the native executable in a runtime container image"
        command: |
          docker build -f ./Dockerfile \
                    --build-arg APP_FILE=${APP_FILE} \
                    -t ${TAG} .
    ```

7. The runtime container image is available as a build output artifact.

    ```shell
    outputArtifacts:
      - name: runtime_image
        type: DOCKER_IMAGE
        location: ${TAG}
    ```

Here's the complete [build specification](build_spec.yaml) file.


## Build Logs

1. The `yum install` build log statements should be similar to:

    ```shell
    ...
    EXEC: Installed:   
    EXEC:   graalvm22-ee-17-native-image.x86_64 0:22.1.0.1-1.el7                             
    EXEC:    
    EXEC: Dependency Installed:   
    EXEC:   glibc-static.x86_64 0:2.17-326.0.1.el7_9                                         
    EXEC:   graalvm22-ee-17-jdk.x86_64 0:22.1.0.1-1.el7                                      
    EXEC:   libstdc++-static.x86_64 0:4.8.5-44.0.3.el7                                       
    EXEC:   zlib-static.x86_64 0:1.2.7-20.el7_9                                              
    EXEC:    
    EXEC: Complete!
    ...
    ```

2. The native executable build log statements should be similar to:

    ```shell
    ...
    EXEC: ==================================================================
    EXEC: GraalVM Native Image: Generating 'MnHelloRest' (static executable)...
    EXEC: ==================================================================
    EXEC: [1/7] Initializing...                         (6.8s @ 0.27GB)   
    EXEC:  Version info: 'GraalVM 22.1.0.1 Java 17 EE'   
    EXEC:  C compiler: gcc (redhat, x86_64, 4.8.5)   
    EXEC:  Garbage collector: Serial GC   
    EXEC:  4 user-provided feature(s)   
    EXEC:   - io.micronaut.buffer.netty.NettyFeature   
    EXEC:   - io.micronaut.core.graal.ServiceLoaderFeature   
    EXEC:   - io.micronaut.http.netty.graal.HttpNettyFeature   
    EXEC:   - io.micronaut.jackson.JacksonDatabindFeature   
    ...
    EXEC: [2/7] Performing analysis...  [**************] (63.9s @ 1.98GB)   
    EXEC:   13,612 (91.86%) of 14,818 classes reachable   
    EXEC:   18,692 (57.10%) of 32,734 fields reachable   
    EXEC:   74,699 (64.21%) of 116,327 methods reachable   
    EXEC:      750 classes,   341 fields, and 2,746 methods registered for reflection   
    EXEC:       62 classes,    68 fields, and    54 methods registered for JNI access   
    EXEC: [3/7] Building universe...                    (5.0s @ 1.94GB)   
    EXEC: [4/7] Parsing methods...      [***]           (5.1s @ 2.60GB)   
    EXEC: [5/7] Inlining methods...     [****]          (8.6s @ 1.77GB)   
    EXEC: [6/7] Compiling methods...    [***********]   (135.4s @ 1.80GB)   
    EXEC: [7/7] Creating image...                       (6.2s @ 2.01GB)   
    EXEC:   34.67MB (51.40%) for code area:   43,533 compilation units   
    EXEC:   24.50MB (36.31%) for image heap:   9,698 classes and 352,409 objects   
    EXEC:    8.29MB (12.29%) for other data   
    EXEC:   67.46MB in total   
    EXEC: ------------------------------------------------------------------
    ...
    EXEC: ------------------------------------------------------------------
    EXEC:  16.5s (6.9% of total time) in 111 GCs | Peak RSS: 4.46GB | CPU load: 3.38   
    EXEC: ------------------------------------------------------------------
    EXEC: Produced artifacts:   
    EXEC:  /workspace/mn-hello/target/MnHelloRest (executable)   
    EXEC:  /workspace/mn-hello/target/MnHelloRest.build_artifacts.txt   
    EXEC: ==================================================================
    EXEC: Finished generating 'MnHelloRest' in 3m 57s.   
    ...
    ```
