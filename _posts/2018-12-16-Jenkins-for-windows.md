---
layout: post
title: .NET Development with Jenkins
tags: [Jenkins, Automation Server]
comment: false
---

# .NET Development with Jenkins

[Jenkins](https://jenkins.io/) a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

## Context

The goal is to create an enviroment where we don't have to depend on developers to manage the build and release process.


## Installing Jenkins
Deploying jenkins in Windows Server was straight forward . I encounted a few issues when I tried to attach a windows slave.
* I installed Jenkins via the Chocolatey package manager. This would allow us to easily manage upgrade process and not have to deal with binary files.
```
choco install chocolatey
```

* In the jenkins slave I installed Java SE Runtime Environment via chocolatey package manager
```
choco install jre8
```

* In the Jenkins interface I enabled 'TCP port for JNLP agents' and set it to 'Random'
```
Jenkins UI > Manage Jenkins > Configure Global Security > Agents
```