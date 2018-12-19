---
layout: post
title: .NET Development with Jenkins
tags: [Jenkins, Automation Server]
comment: false
---

# .NET Development with Jenkins

[Jenkins](https://jenkins.io/) is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

## Context

The goal is to create an enviroment where we don't have to manage the build and release process for every single deployment.


## Installing Jenkins
Deploying jenkins in Windows Server was straight forward . I encountered a few issues when I tried to attach a windows slave.
* I installed Jenkins via the [Chocolatey Package Manager](https://chocolatey.org/). This would allow us to easily manage upgrade process and not have to deal with binary files.
```
choco install chocolatey
```

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