---
layout: post
title:  "How to deploy 'bosom2' WAR file in Heroku"
date:   2017-04-09
categories: projects bosom notes deployment heroku
---

Today after successfully [converting `bosom` to a Maven project]({{site_url}}{% post_url 2017-04-09-how-to-convert-bosom-to-a-pom-based-maven-project %}) I decided to deploy it in Heroku. I know two options:

* [regular deployment of Java applications](https://devcenter.heroku.com/articles/deploying-java)
* [deployment using a WAR file](https://devcenter.heroku.com/articles/war-deployment)

This post followed the steps in the Heroku "WAR Deployment" article.

You can visit the deployed Heroku site in [`bosom2-war`](https://bosom2-war.herokuapp.com/).

## Steps

### Prepare `WAR` file

Make sure that a `WAR` file is generated from our project. Generally it should be found in `target/classes/<name-of-project>.war` (`bosom2-2.0.war` in my case).

If there is none, then just run:

    mvn clean install

### Deploy the `WAR` file

~~~ console
> heroku war:deploy target/bosom2-2.0.war -a bosom2
Uploading bosom2-2.0.war
-----> Packaging application...
       - app: bosom2
       - including: webapp-runner.jar
       - including: target/bosom2-2.0.war
Exception in thread "main" org.apache.http.client.HttpResponseException: Not Found
    at com.heroku.sdk.deploy.utils.RestClient.handleResponse(RestClient.java:172)
    at com.heroku.sdk.deploy.utils.RestClient.get(RestClient.java:66)
    at com.heroku.sdk.deploy.ConfigVars.getConfigVars(ConfigVars.java:41)
    at com.heroku.sdk.deploy.ConfigVars.merge(ConfigVars.java:24)
    at com.heroku.sdk.deploy.Deployer.mergeConfigVars(Deployer.java:106)
    at com.heroku.sdk.deploy.Deployer.deploy(Deployer.java:68)
    at com.heroku.sdk.deploy.App.deploy(App.java:57)
    at com.heroku.sdk.deploy.App.deploy(App.java:61)
    at com.heroku.sdk.deploy.WarApp.deploy(WarApp.java:30)
    at com.heroku.sdk.deploy.DeployWar.main(DeployWar.java:109)
 ▸    There was a problem deploying to bosom2.
 ▸    Make sure you have permission to deploy by running: heroku apps:info -a bosom2
~~~

~~~ console
> heroku apps:info -a bosom2
 ▸    Couldn't find that app.
~~~

This happend because Heroku will not create the applicatiom named "bosom2" like in the usual way. Instead, "bosom2" should be made manually in the website and try deploying again.

~~~ console
> heroku war:deploy target/bosom2-2.0.war -a bosom2
Uploading bosom2-2.0.war
-----> Packaging application...
       - app: bosom2
       - including: webapp-runner.jar
       - including: target/bosom2-2.0.war
-----> Creating build...
       - file: slug.tgz
       - size: 33MB
-----> Uploading build...
       - success
-----> Deploying...
remote:
remote: -----> heroku-deploy app detected
remote: -----> Installing OpenJDK 1.8... done
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 81.8M
remote: -----> Launching...
remote:        Released v4
remote:        https://bosom2.herokuapp.com/ deployed to Heroku
remote:
-----> Done
~~~

Remember that there was no `Procfile` mentioned in the official guide. It is because it is generated automatically and checking it in the website will show:

    web java $JAVA_OPTS -jar webapp-runner.jar ${WEBAPP_RUNNER_OPTS} --port $PORT ./target/bosom2-2.0.war

Finally run the application:

    heroku open -a bosom2
