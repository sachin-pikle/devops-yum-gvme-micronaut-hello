# Using Oracle GraalVM in OCI DevOps to build a Micronaut REST App

This sample shows how to use Oracle GraalVM in OCI DevOps build pipelines to build a simple Micronaut hello world REST application. You can use this approach to build any high performance Java application with Micronaut, Oracle GraalVM and OCI DevOps.

## What is GraalVM?

GraalVM is a high performance JDK distribution that can accelerate any Java workload running on the HotSpot JVM.

GraalVM Native Image ahead-of-time compilation enables you to build lightweight Java applications that are smaller, faster, and use less memory and CPU. At build time, GraalVM Native Image analyzes a Java application and its dependencies to identify just what classes, methods, and fields are absolutely necessary and generates optimized machine code for just those elements.

Oracle GraalVM is available for use on Oracle Cloud Infrastructure (OCI) at no additional cost.

## What is Micronaut?

Micronaut is a modern, JVM-based framework to build modular, easily testable microservice and serverless applications. By avoiding runtime reflection in favor of annotation processing, Micronaut improves the Java-based development experience by detecting errors at compile time instead of runtime, and improves Java-based application start time and memory footprint. Micronaut includes a persistence framework called Micronaut Data that precomputes your SQL queries at compilation time making it a great fit for working with databases like MySQL, Oracle Autonomous Database, etc.

Micronaut uses GraalVM Native Image to build lightweight Java applications that use less memory and CPUs, are smaller and faster because of an advanced ahead-of-time compilation technology.

## What is OCI DevOps?

OCI DevOps services provide a continuous integration and deployment (CI/CD) platform for developers to automate the build, test, and deployment of software applications on Oracle Cloud.

OCI DevOps build and deployment pipelines reduce change-driven errors and decrease the time customers spend on building and deploying software. The service also provides private Git repositories to store your code and supports connections to external code repositories. 


## Updating the Build Specification File

To install and use Oracle GraalVM in the DevOps build pipeline, update your build specification file as follows:

1. Add the following command to install the required Oracle GraalVM components. For example, this command installs Native Image along with the Java Development Kit (JDK) and other necessary dependencies.

    ```shell
    steps:
      - type: Command
        name: "Install the latest Oracle GraalVM for JDK 17 - JDK and Native Image"
        command: |
          yum -y install graalvm-17-native-image
    ```

2. Set the JAVA_HOME environment variable.

    ```shell
    env:
      variables:
        "JAVA_HOME" : "/usr/lib64/graalvm/graalvm-java17"
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
        name: "Example 2: Build a native executable with Oracle GraalVM for JDK 17 - Native Image"
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
    EXEC:   graalvm-17-native-image.x86_64 0:17.0.7-2.el7                                    
    EXEC:    
    EXEC: Dependency Installed:   
    EXEC:   glibc-static.x86_64 0:2.17-326.0.5.el7_9                                         
    EXEC:   graalvm-17-jdk.x86_64 0:17.0.7-2.el7                                             
    EXEC:   libstdc++-static.x86_64 0:4.8.5-44.0.3.el7                                       
    EXEC:   zlib-static.x86_64 0:1.2.7-21.el7_9                                              
    EXEC:    
    EXEC: Dependency Updated:   
    EXEC:   glibc.x86_64 0:2.17-326.0.5.el7_9                                                
    EXEC:   glibc-common.x86_64 0:2.17-326.0.5.el7_9                                         
    EXEC:   glibc-devel.x86_64 0:2.17-326.0.5.el7_9                                          
    EXEC:   glibc-headers.x86_64 0:2.17-326.0.5.el7_9                                        
    EXEC:    
    EXEC: Complete! 
    ...
    ```

2. The native executable build log statements should be similar to:

    ```shell
    ...
    EXEC: ========================================================================================================================   
    EXEC: GraalVM Native Image: Generating 'MnHelloRest' (static executable)...   
    EXEC: ========================================================================================================================   
    EXEC: [1/8] Initializing...                                                                                    (9.1s @ 0.14GB)   
    EXEC:  Java version: 17.0.7+8-LTS, vendor version: Oracle GraalVM 17.0.7+8.1   
    EXEC:  Graal compiler: optimization level: 2, target machine: x86-64-v3, PGO: ML-inferred   
    EXEC:  C compiler: gcc (redhat, x86_64, 4.8.5)   
    EXEC:  Garbage collector: Serial GC (max heap size: 80% of RAM)   
    EXEC:  4 user-specific feature(s)   
    EXEC:  - io.micronaut.buffer.netty.NettyFeature   
    EXEC:  - io.micronaut.core.graal.ServiceLoaderFeature   
    EXEC:  - io.micronaut.http.netty.graal.HttpNettyFeature   
    EXEC:  - io.micronaut.jackson.JacksonDatabindFeature   
    EXEC: [2/8] Performing analysis...  [*******]                                                                 (93.5s @ 1.62GB)   
    EXEC:   15,049 (90.10%) of 16,702 types reachable   
    EXEC:   22,193 (58.56%) of 37,896 fields reachable   
    EXEC:   83,299 (64.28%) of 129,589 methods reachable   
    EXEC:    4,924 types,   354 fields, and 4,053 methods registered for reflection   
    EXEC:       63 types,    68 fields, and    55 methods registered for JNI access   
    EXEC:        4 native libraries: dl, pthread, rt, z   
    EXEC: [3/8] Building universe...                                                                              (13.4s @ 1.60GB)   
    EXEC: [4/8] Parsing methods...      [******]                                                                  (34.0s @ 1.44GB)   
    EXEC: [5/8] Inlining methods...     [***]                                                                      (3.9s @ 1.70GB)   
    EXEC: [6/8] Compiling methods...    [**************]                                                         (212.6s @ 2.31GB)   
    EXEC: [7/8] Layouting methods...    [****]                                                                    (12.3s @ 1.83GB)   
    EXEC: [8/8] Creating image...       [***]                                                                      (8.4s @ 2.58GB)   
    EXEC:   44.56MB (60.14%) for code area:    48,635 compilation units   
    EXEC:   28.98MB (39.12%) for image heap:  396,992 objects and 292 resources   
    EXEC:  562.93kB ( 0.74%) for other data   
    EXEC:   74.09MB in total   
    EXEC: ------------------------------------------------------------------------------------------------------------------------   
    EXEC: Top 10 origins of code area:                                Top 10 object types in image heap:   
    EXEC:   16.60MB java.base                                            9.59MB byte[] for code metadata   
    EXEC:    4.34MB svm.jar                                              3.51MB byte[] for java.lang.String   
    EXEC:    3.40MB java.xml                                             3.21MB java.lang.Class   
    EXEC:    2.41MB jackson-databind-2.14.2.jar                          2.63MB java.lang.String   
    EXEC:    1.27MB netty-buffer-4.1.92.Final.jar                        2.43MB byte[] for general heap data   
    EXEC:    1.13MB micronaut-inject-3.9.3.jar                         950.06kB byte[] for reflection metadata   
    EXEC:    1.03MB reactor-core-3.5.0.jar                             705.42kB com.oracle.svm.core.hub.DynamicHubCompanion   
    EXEC:  915.53kB jdk.proxy4                                         437.38kB c.o.svm.core.hub.DynamicHub$ReflectionMetadata   
    EXEC:  897.38kB micronaut-core-3.9.3.jar                           435.69kB java.util.concurrent.ConcurrentHashMap$Node   
    EXEC:  814.28kB netty-transport-4.1.92.Final.jar                   424.75kB java.util.HashMap$Node   
    EXEC:   11.48MB for 61 more packages                                 4.42MB for 3336 more object types   
    EXEC: ------------------------------------------------------------------------------------------------------------------------   
    EXEC: Recommendations:   
    EXEC:  G1GC: Use the G1 GC ('--gc=G1') for improved latency and throughput.   
    EXEC:  PGO:  Use Profile-Guided Optimizations ('--pgo') for improved throughput.   
    EXEC:  HEAP: Set max heap for improved and more predictable memory usage.   
    EXEC:  CPU:  Enable more CPU features with '-march=native' for improved performance.   
    EXEC:  QBM:  Use the quick build mode ('-Ob') to speed up builds during development.   
    EXEC: ------------------------------------------------------------------------------------------------------------------------   
    EXEC:                        35.5s (9.1% of total time) in 214 GCs | Peak RSS: 3.97GB | CPU load: 3.39   
    EXEC: ------------------------------------------------------------------------------------------------------------------------   
    EXEC: Produced artifacts:   
    EXEC:  /workspace/mn-hello/target/MnHelloRest (executable)   
    EXEC: ========================================================================================================================   
    EXEC: Finished generating 'MnHelloRest' in 6m 28s.
    ...
    ```
