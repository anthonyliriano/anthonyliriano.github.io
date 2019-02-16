---
layout: post
title: .NET Development with Jenkins (Work In Progress)
tags: [Jenkins, Automation Server]
comment: false
---

# .NET Development with Jenkins

[Jenkins](https://jenkins.io/) is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software. You can easily install a Jenkins instance to experiment with via the [Chocolatey Package Manager](https://chocolatey.org/)

```
choco install jenkins
```

## Context

Deploy a continous integration server to automatate the build and release process for Microsoft .NET & Angular 2 applications.

## Jenkins Flow

 * Commit a change to a Github Repository.
 * Jenkins kicks of a job based on a new commit.
 * Unit Tests are ran.
 * Application is deployed.


## Adding A Jenkins Slave.
I encountered a few issues when I tried to add a windows slave/agent. Below you'll find the steps I followed to configure the slave.





* In the jenkins slave(Windows Server 2016) I installed Java SE Runtime Environment via _Chocolatey Package Manager_
```
choco install jre8
```

* In the Jenkins interface enable 'TCP port for JNLP agents' and set it to 'Random'
```
Jenkins UI > Manage Jenkins > Configure Global Security > Agents
```

* Now, we'll create a Windows Task in the jenkins slave. 
```
Windows Task Scheduler > Action > Create Basic Task
Name: 'Jenkins Slave'
When do you want the task to start: 'When the computer starts'
What action do you want the task to perform: 'Start a program'
Program/script: 'Java.exe'
Arguments: -jar "C:\jenkins\agent.jar" -jnlpUrl http://JENKINS-URL:8080/computer/SLAVE-NAME/slave-agent.jnlp -secret 43cc32a3f86670a13b0892ed743f83befe38df5c430d44f25b2f5e9988f6302c -workDir "c:\jenkins"
```

## Adding Micosoft visual Studios
choco install visualstudio2017buildtools .. I'm not sure this is really needed..

## Adding Nuget
choco install nuget.commandline

## Web Applications Folder Issue
https://stackoverflow.com/questions/3980909/microsoft-webapplication-targets-was-not-found-on-the-build-server-whats-your/44704273#44704273 


## Tools

### Catlight