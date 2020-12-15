This branch shows the process of extending an existing Gradle build of a Java application to package it in a CICS bundle. To follow the tutorial, see the [main branch](https://github.com/cicsdev/cics-tutorial-bundle-maven-gradle/tree/main).

---

![](https://github.com/OpenLiberty/open-liberty/blob/master/logos/logo_horizontal_light_navy.png)

The sample application contains a system microservice to retrieve the system properties and uses MicroProfile Config to simulate the status of the microservice, MicroProfile Health to determine the health of the microservice, and MicroProfile Metrics to provide metrics for the microservice.

## Run Sample application (press [ENTER] key to run tests])
    mvn liberty:dev

### Open URL in browser
    http://localhost:9080

### Stop server (quit)
    q [ENTER]