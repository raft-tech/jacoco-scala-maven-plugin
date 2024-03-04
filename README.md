# jacoco-scala-maven-plugin 

## Background

Scala projects at Raft were in need of code coverage tools. The best of these may be [scoverage](https://github.com/scoverage/scalac-scoverage-plugin), but scoverage has compatibility issues with [Quarkus](https://quarkus.io) applications. Quarkus has a history using [Jacoco](https://www.eclemma.org/jacoco/index.html) for code coverage in Java applications. The numerous methods auto-generated by Scala, however, make for messy and misleading reports. Jacoco needs an enhancement to filter out compiler-generated methods and produce reports based solely on user code.

This was the goal of the [jacoco-scala-maven-plugin](https://github.com/timezra/jacoco-scala-maven-plugin) created by Tim Myer. At this time, it appears the last activity in that git repo was in 2013. The repo README includes a reference to Tim's [blog post](http://timezra.blogspot.com/2013/10/jacoco-and-scala.html) where he describes his Scala solution focusing on Scala traits and case objects, and these are realized with two pom.xml configuration filters: **SCALAC.MIXIN** and **SCALAC.CASE**.

## Present State

The jacoco-scala-maven-plugin code (based on version **0.6.3.2-SNAPSHOT**) has been refactored and upgraded for use with jacoco version **0.8.11**. The **jacoco-scala-raft-example** module provides an example build producing a coverage report (jacoco-better-report) for a Quarkus application. The code should be effective, but it is still in an interim state. Moreover, it depends on access still allowed to jacoco internal objects. That access as well as the interfaces could change in a future release if this work is not contributed back to the jacoco code base. 

Other features to note:

* Handles traits and case objects natively, with no filter configuration required. (The original SCALAC.MIXIN and SCALAC.CASE filters remain in the example pom.xml as a historical reference.)
* Still relies on the **\*\*/\*$.class** exclude configuration setting (native to the jacoco base library) to filter out auto-generated companion object methods.
* Should be extensible to filter out additional auto-generated methods for Scala and potentially other languages.

## Logic

Most of this work and all the logic for Scala method filtering appears in ReportMojo.java. Iterative development and re-testing have effectively sidelined the original filter code in favor of new functionality. Jacoco uses the term **synthetic** to describe methods automatically generated by compilers which should be excluded from coverage reports. The enhanced jacoco-scala-maven-plugin module applies three criteria to identify Scala synthetic methods:

1.  They appear in a Scala source file (.scala).
2.  They are named with a known Scala synthetic method name.
3.  The first and last lines of the method are both less than or equal to the first and last lines of all class content.

## Usage

This work was developed and tested with openjdk v17.0.9 and maven v3.9.6.

The jacoco-scalatest-maven-plugin-example module ported from Tim Myer's jacoco-scala-maven-plugin repo is presently non-functional. To build the other modules:

```
mvn clean package
```

To locally install the plugin:

```
mvn -pl jacoco-scala-maven-plugin install
```

To build the sample project and see both the base jacoco and enhanced coverage reports:

```
mvn -pl jacoco-scala-raft-example -Dorg.slf4j.simpleLogger.log.timezra.maven.jacoco.scala.ReportMojo=debug verify
```

## Sample Configuration

This is the essential portion of the jacoco-scala-raft-example pom.xml configuration.

```
      <plugin>
        <groupId>timezra.maven</groupId>
        <artifactId>jacoco-scala-maven-plugin</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <configuration>
          <excludes>
            <exclude>**/*$.class</exclude>
          </excludes>
        </configuration>
        <executions>
          <execution>
            <id>post-integration-test</id>
            <phase>post-integration-test</phase>
            <goals>
              <goal>report</goal>
            </goals>
            <configuration>
              <dataFile>${project.build.directory}/jacoco-quarkus.exec</dataFile>
              <outputDirectory>${project.build.directory}/jacoco-better-report</outputDirectory>
              <filters>
                <filter>SCALAC.MIXIN</filter>
                <filter>SCALAC.CASE</filter>
              </filters>
            </configuration>
          </execution>
        </executions>
      </plugin>
```
