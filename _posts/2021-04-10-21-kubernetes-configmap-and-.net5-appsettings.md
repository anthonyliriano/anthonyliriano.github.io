---
layout: post
type: article
title: Kubernetes ConfigMap & .NET 5 AppSettings.Json
tags: [Kubernetes, .NET5 ]
comment: false
---

## Overview

A Kubernetes ConfigMap is used to store non-confidential data in key-value pairs. The ConfigMap can then be consumed as environment variables, command-line arguments, or as configuration files in a volume. ConfigMaps can also be referenced by multiple pods.

Appsettings.Json is the default configuration file used in .NET to store all of the application settings. 

As we're starting to adopt Kubernetes, we want to start using ConfigMaps to store application configuration. By using ConfigMaps, we hope  we'll be able to reuse our application configuration files and reduce dupplication. 

Currently, any changes made to ConfigMap would require the pod to be restarted in order for the application to read the new changes.
There's an enhancement being considered to allow reloadOnChange option to work in Linux enviroments, feature being tracked here:  [Github Issue #36091](https://github.com/dotnet/runtime/issues/36091)

To view or reference the .NET Project created for this demonstration please go to  [Github Repository](https://github.com/anthonyliriano/.NET5-AppSettings-ConfigMap-Demo)

## Configuring The .NET 5 Project
For this demonstration, I created an empty ASP.NET CORE Web API Project targetting the .NET 5 Framework.

I modified the Program.cs file so that the application is not using the default web host builder and recognizes the appsettings.k8s.json file. Here's what my HostBuilder method looks like like

```c#
public static IHostBuilder CreateHostBuilder(string[] args)
{
    return new HostBuilder()
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.UseKestrel();
        }).ConfigureLogging(logging =>
        {
            logging.ClearProviders();
            logging.SetMinimumLevel(LogLevel.Trace);
        }).ConfigureAppConfiguration((context, config) =>
        {
            config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: false);
            config.AddJsonFile("appsettings.Development.json", optional: true, reloadOnChange: false);
            config.AddJsonFile("appsettings.k8s.json", optional: true, reloadOnChange: false);
        });
}
```

To validate the changes have worked, I'm exposing the Appsettings file via a controller. I created the AppSettingsController.cs with a simple HTTP GET method. 

```c#
[Route("api/settings")]
[ApiController]
public class AppSettingsController : ControllerBase
{
    private readonly IConfiguration _configuration;
    public AppSettingsController(IConfiguration config)
    {
        _configuration = config;
    }

    [HttpGet]
    public IActionResult GetConfigurationSettings()
    {
        return Ok(_configuration.GetSection("AppConfiguration").GetChildren());
    }
}
```


## Dockerfile
Containarize the application
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0-focal AS base 

ENV ASPNETCORE_URLS http://+:83
EXPOSE 83

# Set TimeZone
ENV TZ America/New_York

FROM mcr.microsoft.com/dotnet/sdk:5.0-focal AS build 
WORKDIR /src
COPY NET5-AppSettings-ConfigMap-Demo.csproj NET5-AppSettings-ConfigMap-Demo/
RUN dotnet restore NET5-AppSettings-ConfigMap-Demo/NET5-AppSettings-ConfigMap-Demo.csproj --source https://api.nuget.org/v3/index.json
WORKDIR /src/NET5-AppSettings-ConfigMap-Demo
COPY . .
RUN dotnet build NET5-AppSettings-ConfigMap-Demo.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish NET5-AppSettings-ConfigMap-Demo.csproj -c Release -o /app


FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "NET5-AppSettings-ConfigMap-Demo.dll"]
```

## ConfigMap
There are several ways we can consume ConfigMaps inside pods. In this scenario we'll be adding the ConfigMap as a readonly file. In order for this to work, we'll want to structure the Configmap so that the data propery contains both the name of the file and the contents?
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: appsettings-demo
data:
  appsettings.k8s.json: |-
    {
      "Logging": {
        "LogLevel": {
          "Default": "Error",
          "System": "Error",
          "Microsoft": "Error"
        }
      },
      "AppConfiguration": {
        "DatabaseHost": "localhost",
        "ThirdPartyAPIBaseURL" :  "https://local.somewhere.com/"
      }
    }
```
## Deployment
After the ConfigMap is created we need a way to reference the ConfigMap so that the container can read its contents. In the Kubernetes Deployment, we'll want to include two additional properties volumeMounts and volumes.

Volume - In short words, this is can be thought of a directory that contains files. There are different types of volumes, here we're using the configMap type.  In a deployment, the volume is specified in `.spec.volumes`
```yaml
volumes:
  - name: appsettings-demo-volume
    configMap:
      name: appsettings-demo
```

Volume Mounts - Now that we specified the volume, we'll want to indicate where we want to mount this volume ("directory").. In a deployment, the volume is specified in `.spec.containers[*].volumeMounts`

```yaml
volumeMounts:
  - name: appsettings-demo-volume
    mountPath: /app/appsettings.k8s.json
    subPath: appsettings.k8s.json
```

Here's is what the Deployment should look like..
```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: appsettings-demo
  namespace: default
  labels:
    k8s-app: appsettings-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: appsettings-demo
  template:
    metadata:
      name: appsettings-demo
      labels:
        k8s-app: appsettings-demo
    spec:
      volumes:
        - name: appsettings-demo-volume
          configMap:
            name: appsettings-demo
      containers:
        - name: appsettings-demo
          image: 'anthonylir/dotnet-appsettings-configmap-demo:1.0.0'
          volumeMounts:
            - name: appsettings-demo-volume
              mountPath: /app/appsettings.k8s.json
              subPath: appsettings.k8s.json
          imagePullPolicy: Always
      imagePullSecrets:
        - name: regcred


```

## Verifying
To keep things simple, I am testing by executing within the container and running the curl commands below to verify the response contains the contents of the ConfigMap.

Execute into the container
```shell 
kubectl exec --stdin --tty appsettings-demo-69cf9684dc-6w28h -- /bin/bash
````

Run the curl command inside the container
```shell 
curl -X GET localhost:83/api/settings
````
Output
```json
[
   {
      "key":"DatabaseHost",
      "path":"AppConfiguration:DatabaseHost",
      "value":"prod.somedb.com"
   },
   {
      "key":"ThirdPartyAPIBaseURL",
      "path":"AppConfiguration:ThirdPartyAPIBaseURL",
      "value":"https://prod.somewhere.com/"
   }
]
```
