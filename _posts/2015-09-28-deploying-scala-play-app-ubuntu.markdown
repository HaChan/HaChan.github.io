---
layout: post
title:  "Deploying a scala play application on Ubuntu"
date:   2015-09-28
tags: [scala]
---

This blog post is my reference for deploying a scala-play (version 2.4.3) application on Ubuntu machine (14.04 LTS).

Since play 2.4.3 require JDK 1.8 (or later) installed on the system, let start with installing java 8 on Ubuntu machine

##Installing Java 8

First, lets check that if Java is already installed on the machine:

    java -version

If it returns _"The program java can be found in the following packages"_, that mean Java hasn't been installed yet.

Java 8 can be installed through the Oracle JDK (the official JDK). However, since it is no longer provided by Oracle as a default installation for Ubuntu, it needs some configuration to be able to installed:

```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
```

After that, executing the following command to install Java 8:

    sudo apt-get install oracle-java8-installer

###Optional: Managing Java

When there are multiple Java installation on a machine, the Java version to use as default can be chosen. Execute the following command:

    sudo update-alternatives --config java

If there are more than 2 installations, it'll usually return something like this:

```
There are 2 choices for the alternative java (providing /usr/bin/java).

Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-7-oracle/jre/bin/java          1072      auto mode
  1            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1071      manual mode
  2            /usr/lib/jvm/java-7-oracle/jre/bin/java          1072      manual mode

Press enter to keep the current choice[*], or type selection number:
```

Enter a number to chose the java version to be used. This can also be done for the Java compiler (`javac`):

    sudo update-alternatives --config javac

###Setting the JAVA_HOME environment variable

To set the `JAVA_HOME` environment variable, which is needed for a play app, first find out the path of the Java installation:

    sudo update-alternatives --display java

It returns something like:

```
java - manual mode
  link currently points to /usr/lib/jvm/java-8-oracle/jre/bin/java
usr/lib/jvm/java-8-oracle/jre/bin/java - priority 1072
  slave java.1.gz: /usr/lib/jvm/java-8-oracle/man/man1/java.1.gz
Current 'best' version is '/usr/lib/jvm/java-8-oracle/jre/bin/java'.
```

The `Current 'best' version is '/usr/lib/jvm/java-8-oracle/jre/bin/java'` is the JAVA_HOME value. To set it to JAVA_HOME, edit the file `/etc/environment`:

    sudo vi /ect/environment

Add the following line in it:

    JAVA_HOME=/usr/lib/jvm/java-8-oracle/jre/bin/java

After that, reload this file

    source /etc/environment

##Installing sbt

`sbt` is a powerful build definitions for developing a scala application. It is required to compile and package a scala-play application to be able to deploy.

On Ubuntu, `sbt` is provided officially with a DEB package. Run the following commands to install `sbt`:

```
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
sudo apt-get update
sudo apt-get install sbt
```

##Deploying

In development mode, a Play application can be run by calling the `run` command on `sbt` console or by running the command:

    sbt run

on the terminal. However, it should not be used to run the application in production mode. When using `run`, Play will check to see if any files is changed on every request. This have significant performance impacts on the application.

###Deploy using dist task

There are many ways to deploy a Play app in production but the most recommend way is creating a distribution artifact by using `dist` task from `sbt`.

`dist` task builds a binary version of the application that can be ran on a server without any dependency on `sbt` or `activator` (except Java installation).

To use `dist` task to builds the app, first `cd` to the application directory and then either run this command on the terminal:

    sbt dist

or entering the `sbt` console (entering `sbt` on the terminal) and run

    dist

This command will produces a ZIP file in the `target/universal` folder under the application folder, containing all JAR files needed to run the Play app.

To run the application, unzip the ZIP file generated previously and then run the script in the `bin` directory (with the same name as the Play application). It has two version: a bash shell script and a window `.bat` script.

Example: on the application folder name `my-play-app`, first build the binary version of the application:

    sbt dist

It will then create the ZIP file named `my-play-app-1.0-SNAPSHOT.zip` on the `target/universal` folder under the `my-play-app` folder.

Now, unzip the folder and then run the script under the `bin` folder in order to start the application:

    unzip my-play-app-1.0-SNAPSHOT.zip

    my-play-app-1.0-SNAPSHOT/bin/my-play-app

A different configuration file can be set while executing the previous script:

    my-play-app-1.0-SNAPSHOT/bin/my-play-app -Dconfig.file=/path/to/conf/application-prod.conf

**Note**

- The `appVersion` in the `build.scala` can be set with each time the `dist` task is called to build the application in order to specific the deployed version.

- Since the binary generated by `dist` task is independent from `stb` so in production server, it only need the scripts after unzipping the ZIP file generated by dist task and JAVA installation to be able to run. The following bash shell script can be used to deploy the app in production server:

```bash
# nohub will make the program ignore the HUP (hangup) signal, which is triggered when the user logout.
nohup bin/start -Dconfig.resource=application-prod.conf -Dhttp.port=9000 &
```

###Deploying in place

In order to run the application from the project's source directory without the need to create a distributed file, `sbt` provide the `stage` task:

    sbt clean stage

The above command will clean and compile the application, get the required dependencies and then add them into the `target/universal/stage` folder. Also, in this folder, there is a `bin/{projec-name}` script which will run the Play server on the production machine.

Example with the `my-play-app` above example:

    target/universal/stage/bin/my-play-app -Dconfig.file=/path/to/conf/application-prod.conf

**Note**

This approach require `sbt` installation in the production server.


[References]()

[Play Framework Tutorial](https://www.playframework.com/documentation/2.4.3/Production)

[Java installation on Ubuntu (Digital ocean)](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get)
