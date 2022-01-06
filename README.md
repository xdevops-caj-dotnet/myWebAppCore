[TOC]

# Run .NET Core 3.1 WebApp on OpenShift

# Pre-req

- Download and install [.NET 6 SDK](https://dotnet.microsoft.com/en-us/download/dotnet)
- Install VS Code

## Create .Net Core 3.1 WebApp project 

If don't want to create project, you can directly clone this project.

```bash
# check .net sdks
dotnet --list-sdks

# create `global.json` file to switch .net sdk
mkdir core
cd core
dotnet new `global.json` --sdk-version 3.1.416
```
The generated `global.json` file:
```json
{
  "sdk": {
    "version": "3.1.416"
  }
}
```

Create .NET Core 3.1 WebApp project:
```bash
dotnet new webApp -o myWebAppCore --no-https
```

Check `*.csproj` file to verify the sdk version is .NET Core 3.1

Launch the WebApp:
```bash
cd myWebAppCore
dotnet run
```

Access the webapp by browser.

## Deploy .NET Core 3.1 WebApp to OpenShift 

```bash

export GUID=will

oc new-project ${GUID}-dotnet-demo

# Ensure 'dotnet:3.1-ubi8' builder image stream tag exist
oc get imagestreamtag  -n openshift | grep 'dotnet:3.1-ubi8'

# Create application by 'dotnet:3.1-ubi8' builder image stream tag
oc new-app dotnet:3.1-ubi8~https://github.com/xdevops-caj-dotnet/myWebAppCore.git --name my-web-app-core

# Check build logs
oc logs -f bc/my-web-app-core

# Expose service
oc get svc
oc expose svc my-web-app-core

# Check route of the WebApp
oc get route

```

Access the WebApp by browser, Example: <http://my-web-app-core-will-dotnet-demo.apps.${CLUSTER_DOMAIN}>

## Configure GitHub webhook


Open the build config of `my-web-app-core`, and copy the GitHub webhook URL with secret.

https://api.ocpdemo.sandbox1398.opentlc.com:6443/apis/build.openshift.io/v1/namespaces/will-dotnet-demo/buildconfigs/my-web-app-core/webhooks/ijb4UHA2cH1ye7CREXqn/github

In GitHub, open the `myWebAppCore` project settings, and open Webhooks.

Add webhook:
- Pyaload URL: the above GitHub webhook url
- Content type: `application/json`
- Secret: leave empty
- SSL verification: Disable for sel-signed cert
- Check "Just the push event"
- Check "Active"

Try to push something to the GitHub repository to verify if webhook will auto trigger re-build and re-deployment.

