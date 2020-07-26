---
layout: post
title:  "How to run 'bosom' in Eclipse with Apache Tomcat"
date:   2017-04-05
categories: projects java spring-mvc notes bosom
---

A couple of years ago I created a web application called "BOSOM Calculator: A Breast Cancer Outcome - Survival Online Measurement Calculator using Data Mining and Predictive Modeling on SEER data" and basically it takes specific medical information of an individual (may or may not have breast cancer, but assumes they have been to a specialist to give data) and provides a ten-year prediction based from trained models. This is based from Agrawal et. al.'s "[
A lung cancer outcome calculator using ensemble data mining on SEER data](http://dl.acm.org/citation.cfm?id=2003356)" from 2011.

Originally it was hosted in my university's servers but I neglected to maintain it over the years after graduation so there is no deployed version as of this time.

Today, after many setbacks and procrastination, I finally managed to revive and run it successfully in my system. I was anxious because it has been a while since I last checked the code and expected a lot of compiler errors and misalignment of resources and fortunately it required only few modifications to make it work.

## Steps

1.  Clone the project.

        git clone https://giltroymeren@bitbucket.org/giltroymeren/bosom.git

2.  Import the project to your Eclipse environment.

    For environment comparison, here are my current specifications:

    *   Java

            > java -version
            java version "1.8.0_101"
            Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
            Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)

    *   Apache

            > /usr/sbin/httpd -v
            Server version: Apache/2.4.23 (Unix)
            Server built:   Aug  8 2016 18:10:45

    *   Eclipse

        Eclipse Java EE IDE for Web Developers.<br/>
        Version: Neon Release (4.6.0)<br/>
        Build id: 20160613-1800

3.  After import, an error should appear containing these messages:

    > Loading descriptor for bosom

    > An internal error occurred during: "Loading descriptor for bosom.".
    org.eclipse.emf.ecore.xmi.FeatureNotFoundException: Feature 'location' not found. (platform:/resource/bosom/WebContent/WEB-INF/web.xml, 36, 12)

    To fix this, open `WebContent/WEB-INF/web.xml` and in the "Error pages" section, the `<location>` for code `400` is outside the `<error-page>` (compared to the others below it):

    ~~~ xml
    <!-- Error pages -->
    <location>/WEB-INF/jsp/error/400.jsp</location>
    <error-page>
        <error-code>400</error-code>
    </error-page>
    ~~~

    Move it back inside the tag:

    ~~~ xml
    <!-- Error pages -->
    <error-page>
        <location>/WEB-INF/jsp/error/400.jsp</location>
        <error-code>400</error-code>
    </error-page>
    ~~~

4.  This time there could be errors in the JSP files and `/WebContent/resources/js/phantom.script.js`. We will solve the first and ignore the latter because it does not interfere with the overall activity of the project.

    The first lines of the JSPs will have an error:

        The superclass "javax.servlet.http.HttpServlet" was not found on the Java Build Path

    Unfortunately my project has no `pom.xml` to manage dependencies. If I recall correctly, I did not use one then because my reason is that I will only use a few libraries and using a POM would just complicate things. I regret that now.

    Moving on, just download the `servlet-api-2.5.jar` file from [MVN Repo](https://mvnrepository.com/artifact/javax.servlet/servlet-api/2.5) and add it as an External JAR file in the Properties to solve the issue.

5.  Next, try to run the project with the Apache Tomcat configured in your Eclipse.

    It should show the home page:

    ![BOSOM - Home page](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/pages/home.png){:class="img-responsive"}

    If any of the other tabs are visited (say "BOSOM Calculator"), this page should appear:

    ![BOSOM - Exception page](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/pages/exception.png){:class="img-responsive"}

    and the console should show the following error:

    ~~~ java
    SEVERE: Servlet.service() for servlet [spring] in context with path [/bosom] threw exception [Request processing failed;
        nested exception is org.springframework.beans.factory.CannotLoadBeanClassException:
        Cannot find class [ph.edu.upm.agila.gtmeren.bosom.controller.PdfController] for bean with name 'pdfView' defined in null;
        nested exception is java.lang.ClassNotFoundException: ph.edu.upm.agila.gtmeren.bosom.controller.PdfController] with root cause
    java.lang.ClassNotFoundException: ph.edu.upm.agila.gtmeren.bosom.controller.PdfController
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1284)
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1118)
        at org.springframework.util.ClassUtils.forName(ClassUtils.java:257)
        at org.springframework.beans.factory.support.AbstractBeanDefinition.resolveBeanClass(AbstractBeanDefinition.java:416)
        at org.springframework.beans.factory.support.AbstractBeanFactory.doResolveBeanClass(AbstractBeanFactory.java:1302)
        at org.springframework.beans.factory.support.AbstractBeanFactory.resolveBeanClass(AbstractBeanFactory.java:1273)
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.predictBeanType(AbstractAutowireCapableBeanFactory.java:575)
        at org.springframework.beans.factory.support.AbstractBeanFactory.isFactoryBean(AbstractBeanFactory.java:1350)
        at org.springframework.beans.factory.support.AbstractBeanFactory.isFactoryBean(AbstractBeanFactory.java:916)
        at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:609)
        at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:932)
        at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:479)
        at org.springframework.web.servlet.view.ResourceBundleViewResolver.initFactory(ResourceBundleViewResolver.java:251)
        at org.springframework.web.servlet.view.ResourceBundleViewResolver.loadView(ResourceBundleViewResolver.java:194)
        at org.springframework.web.servlet.view.AbstractCachingViewResolver.createView(AbstractCachingViewResolver.java:241)
        at org.springframework.web.servlet.view.AbstractCachingViewResolver.resolveViewName(AbstractCachingViewResolver.java:153)
        at org.springframework.web.servlet.DispatcherServlet.resolveViewName(DispatcherServlet.java:1239)
        at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1188)
        at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:992)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:939)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:856)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:936)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:827)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:622)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:812)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:230)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:192)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:198)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:108)
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:522)
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:140)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
        at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:620)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:349)
        at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:1110)
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66)
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:785)
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1425)
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)
    ~~~

    *   Set the Java version

        * Projects
        * Properties
        * Java Build Path
            * Libraries tab
                * should show "JRE System Library [jre7] (unbound)" with an error icon (depends on the system).

        !["JRE System Library [jre7] (unbound)" with an error icon](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/eclipse/libraries-jre-with-error.png){:class="img-responsive"}

        Edit this JRE and select your current Java SE version in the "Alternate JRE" part.

        ![Choose "Alternate JRE"](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/eclipse/jre-system-library-alternate-jre.png){:class="img-responsive"}

    *   Set the Apache Tomcat version

        * Project
        * Project Facets
            * Runtimes tab

        In my case, Tomcat 6 was already selected but Tomcat 8 is present so I added it as well.

        ![Project Facets - "Runtimes"](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/eclipse/project-facets-runtimes.png){:class="img-responsive"}


*   Run the application again and the other tabs should be accessible.

    ![BOSOM - "BOSOM Calculator" page](/assets/images/posts/2017-04-05-how-to-run-bosom-in-eclipse-with-apache-tomcat/pages/bosom-calculator.png){:class="img-responsive"}
