---
layout: post
title: Host your Ktor application with Postgres DB on Heroku for free
date: 2020-08-04 
fig-caption: # Add figcaption (optional)
tags: [Ktor, Heroku, Postgres, Intellij]
---
##  Step - 1:

Generate a fat Jar, depending upon the gradle build file, please use as specified below,

- **build.gradle**

```
 buildscript {
     repositories {
        jcenter()
     }
     dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:2.0.4'
     }
 }

 apply plugin: 'com.github.johnrengelman.shadow'
 apply plugin: 'kotlin'
 apply plugin: 'application'

 mainClassName = 'io.ktor.server.netty.EngineMain' 

 // This task will generate your fat JAR and put it in the ./build/libs/ directory
 shadowJar {
    manifest {
        attributes 'Main-Class': mainClassName
    }
 }
```

- **build.gradle.kts**

```
plugins {
    application
    kotlin("jvm") version "1.3.21"
    id("com.github.johnrengelman.shadow") version "5.0.0"
}

application {
    mainClassName = "io.ktor.server.netty.EngineMain"
}

tasks.withType<Jar> {
    manifest {
        attributes(
            mapOf(
                "Main-Class" to application.mainClassName
            )
        )
    }
}
```

you can read more from [Fat Jar](https://ktor.io/servers/deploy/packing/fatjar.html)

Then, if all goes well, proceed to generation of Jar file, as specified in the image below,

![Jar Generation with gradle]({{site.baseurl}}/assets/img/jar_generation_with_gradle.png)

First clean the project and then build it, After build, a jar file is generated in root build section.

##  Step - 2:
Make sure __heroku__ is installed on your system, with command,
>heroku --version.

if it's not installed, please install it with **homebrew** or with **Heroku Installer** from their official website

##  Step - 3:
Create an app.json file in the root directory of your project, with following like structure,

```json
{
   "name": "Start on Heroku: Kotlin",
   "description": "A barebones Kotlin app, which can easily be deployed to Heroku.",
   "image": "heroku/java",
   "addons": [ "heroku-postgresql" ]
 }
```

I am not sure whether, it's a essential step or not, but just for smoother process, i had added it

##  Step - 4:
Create a file named __Procfile__ without any extension in the root folder or project, with contents like same below,
>web:    java -jar build/name_of_project.jar

in my case it's as follows
>web: java -jar build/libs/khatabook-0-0-1-with-dependencies.jar

##  Step - 5:
Create a file named __system.properties__ in root folder or project, describing your java version,
 with contents like below,
>java.runtime.version=1.8

##  Step - 6:
### 6.A(Create .env file) 
Create a file named __.env__ in root folder or project,
 with contents like below,
 
```
PORT=8080
JDBC_DATABASE_URL=jdbc:postgresql://localhost:5432/java_database_name
```

If your local installation of postgresql has a user/password, you have to change the jdbc url too:
```
JDBC_DATABASE_URL=jdbc:postgresql://localhost:5432/java_database_name?user=user&password=password
```

### 6.B(Database connection establishment)
```
private val hikariConfig = HikariConfig().apply {
        jdbcUrl = System.getenv("JDBC_DATABASE_URL")
    }

    private val dataSource = if (hikariConfig.jdbcUrl != null)
        HikariDataSource(hikariConfig)
    else
        configureHikariCP()

    private fun configureHikariCP(): HikariDataSource {
        val config = HikariConfig("/hikari.properties") 
```

### Local Deployment on Heroku
After completion, till step-6, you should be able to run or deploy the app, locally on Heroku
>heroku local:start

##  Step - 7:
This is the final step to deploy app on Heroku server, but first make sure, you have an account on Heroku, if not,
please signup and have it.

### 7.A
 You first have to create an app or set the git remote with heroku create
 >heroku create

 With above command you should see, the output somewhat like this,
 ```
 Creating app... done, ⬢ demo-demo-12345
 https://demo-demo-12345.herokuapp.com/ | https://git.heroku.com/demo-demo-12345.git
 ```
 
### 7.B
  > cat .git/config
  
  It will adds a heroku remote to your git clone:

### 7.C
  > git push heroku master
  
  It will push the git changes to the heroku remote. generate a build on push
  
  If everything goes well, then you should see some url like below, at the end of log,
  
  ```
  remote: Verifying deploy... done.
  To https://git.heroku.com/demo-demo-12345.git
   * [new branch]      master -> master
  ```
  And if push get rejected or your log is somewhat like below,
  
  ```
    remote:        If you're stilling having trouble, please submit a ticket so we can help:
    remote:        https://help.heroku.com
    remote:        
    remote:        Thanks,
    remote:        Heroku
    remote: 
    remote:  !     Push rejected, failed to compile Gradle app.
  ```

  Then, please run the following command from terminal,
  >heroku config:set GRADLE_TASK="build"
  
  Then again push the changes to heroku server, with below command,
  >git push heroku master

  From now, you should be able to setup everyting well, without getting any error, You can veryify it by running the following 
  command,
  >heroku open

### 7.D

  Till this step, your app is ready, but you hadn't setup your **postgres db** till now on heroku server
  So, to do that, we have to run the following command, on your system terminal
  

  >``heroku addons:create heroku-postgresql:<PLAN_NAME>``


  For __free plan__, i am using hobby-dev, so at my end my db query is like below,
  >heroku addons:create heroku-postgresql:hobby-dev

  To know more about postgres on heroku, you can go for this [link](https://devcenter.heroku.com/articles/heroku-postgresql), 

  So, after this, you should be able to access or create db on heroku.
  
  Please feel free, to reach out to me, in case of any issue regarding this article.
  
  
  
### References:

  1. https://ktor.io/servers/deploy/hosting/heroku.html