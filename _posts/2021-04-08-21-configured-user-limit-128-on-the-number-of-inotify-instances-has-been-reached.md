---
layout: post
title: Inotify Instances Limit Has Been Reached
tags: [Kubernetes, .NET 5, EKS, Linux]
comment: false
---

## Overview
While modifying one of our Kubernetes Deployments that caused a pod restart, we started receiving the System.IO.IOException below. The pod would not go into the ready state and would continue to restart. This was spread through our namespaces and would resufarce everytime we restarted a pod or scaled a deployment. 

```
Unhandled exception. System.IO.IOException: The configured user limit (128) on the number of inotify instances has been reached, or the per-process limit on the number of open file descriptors has been reached.
   at System.IO.FileSystemWatcher.StartRaisingEvents()
   at System.IO.FileSystemWatcher.StartRaisingEventsIfNotDisposed()
   at System.IO.FileSystemWatcher.set_EnableRaisingEvents(Boolean value)
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.TryEnableFileSystemWatcher()
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.CreateFileChangeToken(String filter)
   at Microsoft.Extensions.FileProviders.PhysicalFileProvider.Watch(String filter)
   at Microsoft.Extensions.Configuration.FileConfigurationProvider.<.ctor>b__1_0()
   at Microsoft.Extensions.Primitives.ChangeToken.OnChange(Func`1 changeTokenProducer, Action changeTokenConsumer)
   at Microsoft.Extensions.Configuration.FileConfigurationProvider..ctor(FileConfigurationSource source)
   at Microsoft.Extensions.Configuration.Json.JsonConfigurationSource.Build(IConfigurationBuilder builder)
   at Microsoft.Extensions.Configuration.ConfigurationBuilder.Build()
   at Microsoft.AspNetCore.Hosting.WebHostBuilder.BuildCommonServices(AggregateException& hostingStartupErrors)
   at Microsoft.AspNetCore.Hosting.WebHostBuilder.Build()
```

## Finding the Solution
After looking around, everything was pointing to the root issue being that the Kubernetes cluster was running out of inotify resources at the OS level. The OS Resource limits are defined by fs.inotify.max_user_watches and fs.inotify.max_user_instances environment variables. 

to view the inotify resource limits, execute into a pod in the cluster and run sysctl fs.inotify. The values may vary depending on the OS.
```
root@sdsswe-5bcb94bbf4-9jtr5:/app#  sysctl fs.inotify
fs.inotify.max_queued_events = 16384
fs.inotify.max_user_instances = 128
fs.inotify.max_user_watches = 524288
```

### Temporary Fix - Short Term
Increase the allocated inotify resources. From what I understand this can't be modified via Dockerfile so it must be done directly on the Virtual Machine. 
```
echo fs.inotify.max_user_instances=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

### What Worked For Us - Long Term
After some searching, it seems that this was being caused by our use of the CreateDefaultBuilder in Program.cs, there's setting within the CreateDefaultBuilder that we cannot override that is setting our AppSettings.Json files to reloadOnChange. https://github.com/dotnet/runtime/blob/release/5.0/src/libraries/Microsoft.Extensions.Hosting/src/Host.cs#L74-L77 

```
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            new HostBuilder()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseKestrel();
                }).ConfigureLogging(logging =>
                {
                    logging.ClearProviders();
                    logging.SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Trace);
                }).ConfigureAppConfiguration((context, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: false);
                    config.AddJsonFile("appsettings.Development.json", optional: true, reloadOnChange: false);
                })
                .UseNLog();
```
## Why?
I still have a few questions, that I couldn't find the answer to. Let me know if you have answers?
- If we're not changing the AppSettings.Json file within the Docker container, why is realoded on change causing this issue?