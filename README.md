# Extending an existing Java application's build to produce a CICS bundle

Java is well-served in its two most popular build systems - Gradle and Maven. Their power and their ubiquity mean that most Java applications get built by either one or the other of them, and that can include Java applications that are intended to be installed into CICS Transaction Server.

Deploying Java applications to CICS can be achieved in a number of ways, but using CICS bundles to package those applications gives the advantage of being able to control that application in 'standard' CICS ways.

You can easily extend an existing Gradle or Maven build of a Java application to also build a CICS bundle. Let's take a look at how to do that.

## Prerequisites

Let's assume you already have your Java application, being built using Gradle or Maven. I'm going to start from the [Sample Getting Started application](https://github.com/OpenLiberty/sample-getting-started) supplied by the Open Liberty team and extend that, as it's set up with a build for both Gradle and Maven.

There's a branch in my fork of the sample for each of the build systems, [Maven](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/tree/maven) and [Gradle](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/tree/gradle), showing how I changed the project compared to the [original](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/tree/original). Each step is a commit, so you can follow along with the workflow. As I go through each step, I'll link to a comparison between each commit and its prior state.

## Estimated time

 It should take about 20 minutes to complete this tutorial.

Let's start with Maven.

## Steps for extending a Maven build

1. IBM supplies the [CICS bundle Maven plugin](https://github.com/IBM/cics-bundle-maven) to build CICS bundles using Maven. It's deployed to [Maven Central](https://search.maven.org/artifact/com.ibm.cics/cics-bundle-maven-plugin) so you can easily refer to it from a build without setting up repositories, or get it mirrored to your internal artifact repository.

    Extending the build is simply a matter of:

     - enabling the plugin, using group ID `com.ibm.cics` and artifact ID `cics-bundle-maven-plugin`.
     - telling it to run a goal (the Maven term for a task).
     - telling it what the name of the target JVM server in CICS is.
 
     We do this by [adding the following](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/d331910...d4c56e9) to the `<plugins>` section of the Maven `pom.xml`:

    ```diff
              </systemPropertyVariables>
            </configuration>
          </plugin>
    +      <plugin>
    +        <groupId>com.ibm.cics</groupId>
    +        <artifactId>cics-bundle-maven-plugin</artifactId>
    +        <version>1.0.2</version>
    +        <executions>
    +          <execution>
    +            <goals>
    +              <goal>bundle-war</goal>
    +            </goals>
    +            <configuration>
    +              <jvmserver>MYJVMS</jvmserver>
    +            </configuration>
    +          </execution>
    +        </executions>
    +      </plugin>
        </plugins>
      </build>
    </project>
    ```

    In this case our application is deployed as a WAR file, but pick the goal that's relevant to your application - `bundle-ear`, `bundle-eba` and `bundle-osgi` also exist.
 
1. Now, run a Maven build that includes the `verify` phase, such as `mvn verify` or `mvn install`. Towards the end of the build, you'll see the plugin logging its progress:

    ```
    [INFO] --- cics-bundle-maven-plugin:1.0.2:bundle-war (default) @ io.openliberty.sample.getting.started ---
    [INFO] Building zip: /path/to/my/project/extend-build-cics-bundle/target/io.openliberty.sample.getting.started-1.0-SNAPSHOT-cics-bundle.zip
    ```

    That was easy! Take a look at the zip mentioned in the log. It's a fully-fledged CICS bundle, with the Java application as a bundle part inside it, ready to be deployed to CICS or stored in an artifact repository for later deployment.

You can see the entire change [in the repository](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/before...maven).

But is Maven not your thing? Let's take a look at how to do the same, in Gradle.

## Steps for extending a Gradle build

The sample supplied by Open Liberty only has a Maven build, so the first thing I need to do to it is to [convert the build from Maven to Gradle](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/1103cf4..88a77ec). The CICS bundle Gradle plugin requires you to use at least Gradle version 5. I've upgraded to the latest at the time of writing, Gradle 6.7. Of course, if you're using Gradle already, you won't need to do this conversion, you'll already have a working Gradle build.

Now down to the real business of extending the build to generate the CICS bundle. 

1. IBM supplies the [CICS bundle Gradle plugin](https://github.com/IBM/cics-bundle-gradle) to build CICS bundles using Gradle. It's easiest to get it from the [Gradle Plugin Portal](https://plugins.gradle.org/plugin/com.ibm.cics.bundle), though if you prefer to get it from Maven Central it's [also available there](https://search.maven.org/artifact/com.ibm.cics.bundle/com.ibm.cics.bundle.gradle.plugin).

    I [enable the CICS bundle plugin](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/88a77ec...2947a42):

    ```diff
    plugins {
        id 'war'
        id 'io.openliberty.tools.gradle.Liberty' version '3.1'
    +    id 'com.ibm.cics.bundle' version '1.0.1'
    }
    
    version = "1.0-SNAPSHOT"
    ```

1. What would happen if you ran a build now? Well, you'd get a CICS bundle, but it wouldn't have the Java application added as a bundle part - it would be empty apart from the `cics.xml` file. We need to let the CICS bundle plugin know about the bundle part, by adding the file that's output as part of the `war` task as a dependency using the `cicsBundlePart` configuration, and configuring the name of the target JVM server:

    ```diff
        intTestImplementation 'org.glassfish:javax.json:1.1.4'
        // Support for JDK 9 and above
        intTestImplementation 'javax.xml.bind:jaxb-api:2.3.1'
    +    cicsBundlePart files(war)
    +}
    +
    +cicsBundle {
    +    build {
    +        defaultJVMServer = 'MYJVMS'
    +    }
    }
    
    liberty {
    ```

1. With that [latest change](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/2947a42...gradle) in place, try running a build:

    ```bash
    ./gradlew build
    ```

    You'll see two extra tasks being run - `buildCICSBundle` and `packageCICSBundle`. These are doing the work of creating the `cics.xml` file, pulling in the bundle parts, and packaging that final CICS bundle as a zip. You can find the CICS bundle under `./build/distributions/io.openliberty.sample.getting.started-1.0-SNAPSHOT.zip`, ready to be deployed to CICS or stored in an artifact repository for later deployment.

You can see the entire change [in the repository](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/compare/before...gradle).

## Summary

That's it - converting an existing Java application's build to generate a CICS bundle is very easy, and only takes a couple of steps, whether you use Maven or Gradle.



## Next steps

The plugins are capable of much more, if you need it. You can also do things like:

 - Keep your Java application agnostic of CICS, or build it at a different time from the CICS bundle, by pulling the Java application and the CICS bundle into two different modules.
 - Add more function to your CICS bundle by adding more than one bundle part, including non-Java-based bundle parts such as transactions, programs, or URI maps.

To find out how to do these, look at the [Maven plugin](https://github.com/IBM/cics-bundle-maven) and [Gradle plugin](https://github.com/IBM/cics-bundle-gradle) READMEs.

And one more thing...

Using these build systems make it easy to deploy the output of builds to artifact repositories like JFrog Artifactory or Sonatype Nexus, ready to be deployed by automation when they're needed.

However, during development you will want to be deploying to CICS on a regular basis, and going through CI builds and deployments will slow you down. The [CICS bundle deployment API](https://www.ibm.com/support/knowledgecenter/SSGMCP_5.6.0/fundamentals/cpsm/cics-bundle-api.html) was introduced at CICS TS 5.6 to help deploy CICS bundles to CICS, and it really shines when you deploy straight to CICS from the CICS bundle Maven or Gradle plugins. In a matter of seconds, your application is ready to try out in a real CICS region.

With the CICS bundle deployment API set up, you just need to add a few more lines to your build to get your CICS bundle deploying to CICS at the end of the build - the [Maven plugin](https://github.com/IBM/cics-bundle-maven) and [Gradle plugin](https://github.com/IBM/cics-bundle-gradle) READMEs show you exactly how.

## Related links

- [The CICS bundle Maven plugin](https://github.com/IBM/cics-bundle-maven)
- [The CICS bundle Gradle plugin](https://github.com/IBM/cics-bundle-gradle)
- [Java support in CICS](https://www.ibm.com/support/knowledgecenter/SSGMCP_5.6.0/fundamentals/java/JVMsupport.html)