---
layout: post
title:  "How to convert 'bosom' to a POM-based Maven project"
date:   2017-04-09
updated: 2017-04-09
categories: projects java spring-mvc notes bosom maven
---

In my [previous post]({{site_url}}{% post_url 2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat %}), I revisited how to run my old project [`bosom`](https://giltroymeren@bitbucket.org/giltroymeren/bosom) locally in my Eclipse environemnt. Unfortunately it is not POM-based (explained in the said post), which made it inconvenient to setup and I believe I do not have a straightforward way to deploy it as an application in [Heroku](https://www.heroku.com/) when I tried to (it was not recognized as any application type during my initial `push`).

Fortunately I was able to convert the entire project to use Maven POM and it is available as [`bosom2`](https://bitbucket.org/giltroymeren/bosom2).

## Table of Contents
* TOC
{:toc}

## Steps

The following are the steps I performed in order to setup and successfully run `bosom2` as a Maven project.

### Checkout `bosom`

I prepared a copy of the original `bosom` repository because we need its files later.

    git clone https://giltroymeren@bitbucket.org/giltroymeren/bosom.git

### Create new Maven project

I followed App Shah's [*Simplest Spring MVC Hello World Example / Tutorial – Spring Model – View – Controller Tips*](https://crunchify.com/simplest-spring-mvc-hello-world-example-tutorial-spring-model-view-controller-tips/) to create the initial project.

*   Create a new "Dynamic Web Project" with "bosom2" as "Project name".

![Dynamic Web Project](/assets/images/posts/2017-04-09-how-to-convert-bosom-to-a-pom-based-maven-project/dynamic-web-project.png){:class="img-responsive"}

*   Right click the project

    *   "Configure"
    *   "Convert to Maven Project"
    *   Provide the following details (you can use other values)

        * Group ID: `ph.edu.upm.agila.gtmeren`
        * Artifact ID: `bosom2`
        * Name: `bosom2`

### Set dependencies

I added the following to the `pom.xml`, in the same level as the `<build>` block.

~~~ xml
<properties>
    <version.springframework>3.2.9.RELEASE</version.springframework>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${version.springframework}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${version.springframework}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${version.springframework}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${version.springframework}</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <dependency>
        <groupId>nz.ac.waikato.cms.weka</groupId>
        <artifactId>weka-dev</artifactId>
        <version>3.7.10</version>
    </dependency>
    <dependency>
        <groupId>nz.ac.waikato.cms.weka</groupId>
        <artifactId>alternatingDecisionTrees</artifactId>
        <version>1.0.5</version>
    </dependency>

    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itextpdf</artifactId>
        <version>5.5.0</version>
    </dependency>
    <dependency>
        <groupId>org.jfree</groupId>
        <artifactId>jfreechart</artifactId>
        <version>1.0.17</version>
    </dependency>

    <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>1.1.0.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator-annotation-processor</artifactId>
        <version>4.3.1.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.2.1.Final</version>
    </dependency>

    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.2</version>
    </dependency>

    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.3</version>
    </dependency>
</dependencies>
~~~

These were based from the original `.jar` file list in `bosom`:

![bosom - jar list](/assets/images/posts/2017-04-09-how-to-convert-bosom-to-a-pom-based-maven-project/bosom-jar-list.png){:class="img-responsive"}

### Copy files from `bosom` to `bosom2`

Here is the list of files and directories that should be copied to their respective locations in `bosom2`.

> **NOTE**: I recommend copying them first in your OS's default file manager and pasting within the Eclipse environment.

* `src/ph/`
* `src/file.location.properties`
* `src/messages.validation.properties`
* `src/pdf.properties`

* `WebContent/WEB-INF/index.jsp`
* `WebContent/resources`
* `WebContent/WEB-INF/jsp/`
* `WebContent/WEB-INF/models/`
* `WebContent/WEB-INF/reports/`
* `WebContent/WEB-INF/application-context.xml`
* `WebContent/WEB-INF/spring-servlet.xml`
* `WebContent/WEB-INF/web.xml`

#### Register `.properties` files

In order for the application to find our `.properties` files I also added the following within the `<build>` block in our `pom.xml`:

~~~ xml
<resources>
    <resource>
        <directory>${project.basedir}/src</directory>
        <filtering>true</filtering>
        <includes>
            <include>**/*.properties</include>
        </includes>
    </resource>
</resources>
~~~

The `<directory>` value will look for `<include>` file types and copy them to the `target/classes` folder of a WAR file.

![bosom2 - target/classes](/assets/images/posts/2017-04-09-how-to-convert-bosom-to-a-pom-based-maven-project/bosom2-target-classes.png){:class="img-responsive"}

If this was not set correctly, then there will be an error looking for those files similar to these logs:

~~~ java
SEVERE: Allocate exception for servlet spring
 java.io.FileNotFoundException: class path resource [file.locations.properties] cannot be opened because it does not exist
    at org.springframework.core.io.ClassPathResource.getInputStream(ClassPathResource.java:171)
    at org.springframework.core.io.support.EncodedResource.getInputStream(EncodedResource.java:143)
    at org.springframework.core.io.support.PropertiesLoaderUtils.fillProperties(PropertiesLoaderUtils.java:98)
    at org.springframework.core.io.support.PropertiesLoaderSupport.loadProperties(PropertiesLoaderSupport.java:175)
    at org.springframework.core.io.support.PropertiesLoaderSupport.mergeProperties(PropertiesLoaderSupport.java:156)
    at org.springframework.beans.factory.config.PropertyResourceConfigurer.postProcessBeanFactory(PropertyResourceConfigurer.java:78)
    at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:694)
    at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:669)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:461)
    at org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:651)
    at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:602)
    at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:665)
    at org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:521)
    at org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:462)
    at org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
    at javax.servlet.GenericServlet.init(GenericServlet.java:158)
    at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1238)
    at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1151)
    at org.apache.catalina.core.StandardWrapper.allocate(StandardWrapper.java:828)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:135)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:106)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:141)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:88)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:522)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1095)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:672)
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1502)
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1458)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)
~~~

### Run the application

#### Java Build Path

I went to "Properties" &rarr; "Java Build Path" &rarr; "Libraries" tab and checked "JRE System Library (version)" and "Maven Dependencies". Next I checked "Allow output folders for source folders" in the "Sources" tab. The build paths can cause many problems if they were not set correctly so I made sure to set them correctly.

### Maven

Right click the project &rarr; "Run As" &rarr; "Maven build..." and set `clean install` as "Goals".

~~~ java
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building bosom 2.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ bosom ---
[INFO] Deleting /bosom2/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ bosom ---
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.5.1:compile (default-compile) @ bosom ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 15 source files to /bosom2/target/classes
[INFO] /bosom2/src/ph/edu/upm/agila/gtmeren/bosom/service/impl/CalcArffServiceImpl.java: /bosom2/src/ph/edu/upm/agila/gtmeren/bosom/service/impl/CalcArffServiceImpl.java uses or overrides a deprecated API.
[INFO] /bosom2/src/ph/edu/upm/agila/gtmeren/bosom/service/impl/CalcArffServiceImpl.java: Recompile with -Xlint:deprecation for details.
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ bosom ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /bosom2/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.5.1:testCompile (default-testCompile) @ bosom ---
[INFO] No sources to compile
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ bosom ---
[INFO] No tests to run.
[INFO]
[INFO] --- maven-war-plugin:2.6:war (default-war) @ bosom ---
[INFO] Packaging webapp
[INFO] Assembling webapp [bosom] in [/bosom2/target/bosom-2.0]
[INFO] Processing war project
[INFO] Copying webapp resources [/bosom2/WebContent]
[INFO] Webapp assembled in [575 msecs]
[INFO] Building war: /bosom2/target/bosom-2.0.war
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ bosom ---
[INFO] Installing /bosom2/target/bosom-2.0.war to /Users/username/.m2/repository/ph/edu/upm/agila/gtmeren/bosom/2.0/bosom-2.0.war
[INFO] Installing /bosom2/pom.xml to /Users/username/.m2/repository/ph/edu/upm/agila/gtmeren/bosom/2.0/bosom-2.0.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.905 s
[INFO] Finished at: 2017-04-09T12:32:12+08:00
[INFO] Final Memory: 24M/284M
[INFO] ------------------------------------------------------------------------
~~~

Finally, "Run on Server".

## Problems

I encountered several problems during this process and their respective solutions:

### `MANIFEST.MF` is missing

After I run the application in the server, I got a dialog box containing the following message:

    Publishing failed with multiple errors
    File not found: /bosom2/target/m2e-wtp/web-resources/META-INF/MANIFEST.MF.
    File not found: /bosom2/target/m2e-wtp/web-resources/META-INF/maven/ph.edu.upm.agila.gtmeren/bosom/pom.properties.
    File not found: /bosom2/target/m2e-wtp/web-resources/META-INF/maven/ph.edu.upm.agila.gtmeren/bosom/pom.xml.

The following error also appears in `web.xml`:

    /bosom2/target/m2e-wtp/web-resources/META-INF/MANIFEST.MF (No such file or directory)

To solve this, go to "Preferences" &rarr; "Maven" &rarr; Java EE Integration and uncheck "Maven Archiver generates files under the build directory".

### `resources` are not found in browser

I once had the mistake of misplacing the `resources` folder from `WebContent/` and the following happened to the home page after running:

![Resources not found in browser](/assets/images/posts/2017-04-09-how-to-convert-bosom-to-a-pom-based-maven-project/resources-not-found-in-browser.png){:class="img-responsive"}

Make sure this folder is strictly in `/bosom2/WebContent/resources` because it is set in `spring-servlet.xml`:

    <mvc:resources mapping="/resources/**" location="/, /resources/**, classpath:/WEB-INF/resources" />

## References

* Cosjav. "M2e-wtp error: /target/m2e-wtp/web-resources/META-INF/MANIFEST.MF (No such file or directory)." *Eclipse - m2e-wtp error: /target/m2e-wtp/web-resources/META-INF/MANIFEST.MF (No such file or directory) - Stack Overflow*. N.p., 3 June 2013. Web. 09 Apr. 2017. \<[`http://stackoverflow.com/a/16891764`](http://stackoverflow.com/a/16891764)\>.
* Ppuskar. "Reading Properties file in Maven Java Project created by maven-archetype-quickstart." *Eclipse - Reading Properties file in Maven Java Project created by maven-archetype-quickstart - Stack Overflow*. N.p., 16 July 2014. Web. 09 Apr. 2017. \<[`http://stackoverflow.com/a/24779741`](http://stackoverflow.com/a/24779741)\>.
* Shah, App. "Simplest Spring MVC Hello World Example / Tutorial - Spring Model - View - Controller Tips • Crunchify." *Crunchify*. N.p., 28 Mar. 2017. Web. 09 Apr. 2017. \<[`https://crunchify.com/simplest-spring-mvc-hello-world-example-tutorial-spring-model-view-controller-tips/`](https://crunchify.com/simplest-spring-mvc-hello-world-example-tutorial-spring-model-view-controller-tips/)\>.


## Changes

* `2017-04-09`
    * Changed the path of `.properties` files in [Register `.properties` files](#register-properties-files) section.
    * Added supplementary explanation in [Register `.properties` files](#register-properties-files) section.