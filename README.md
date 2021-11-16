# Deploy DotNet 6.0 Application to Openshift 
The objective of this repo is to deploy a sample dotnet 6.0 application that meet the following requirements: 
1. BuildConfig + ImageStream: 
    * binary based build
    * Dockerfile Build Strategy
    * Using the RedHat .NetCore 6.0 UBI 8 Image: [.NET 6.0 SDK and Runtime](https://catalog.redhat.com/software/containers/ubi8/dotnet-60/6182efb9be25a74c00923849)
2. Deployment, Service, Route Openshift Resources to run the microservice in openshift 

This code base for this application was originally developed by the RH Consulting Team: [redhat-developer/s2i-dotnetcore-ex](https://github.com/redhat-developer/s2i-dotnetcore-ex)

## Install Latest DotNet Image Streams on Openshift 
Please validate and run the following command if necessary: 
1. Check the image streams that are present in the ocp internal registry: `$ oc describe is dotnet -n openshift`
2. Import the addtional image streams if the dotnet image stream version is missing: `oc replace -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json`


## Building the Application 
The BuildConfig is designed to use: 
* binary input - allows developers to upload source or artifacts directly to a build instead of having the build pull source from a Git repository 
* dockerfile build strategy - allows developers to use a dockerfile to build the application layer image 

1. Create the BuildConfig and Image Stream Openshift Resources from the parent directory: 
```
oc apply -f openshift/bc.yaml 
oc apply -f openshift/is.yaml 
```

2. To start the build run the following command in the parent directoy: 
   * `oc start-build dotnetcore-ex --from-dir=./`

Validate a successful build: 
1. check build summary: 
    * `oc get builds`
    * `oc describe build <build-name>`
2. check build  logs: `oc logs -f <pod-name>`

## Deploying and Exposing the Application 
Once the application has been built successfully deploy and expose the microservice on the ocp cluster: 
```
oc apply -f openshift/deploy.yaml
oc apply -f openshift/svc.yaml
oc apply -f openshift/route.yaml
```

Validate the you can reach the route url in the browser, `oc get route`, and navigate to the url on https, disable edge termination if necessary.

## Modifying the BuildConfig 
In the buildconfig provided, the builder pod will look for a imagestream **dotnet** with the tag **6.0** in the openshift namespace, and output build to a second image stream **dotnetcore-ex**  with the tag **latest**. Depending on the needs of the CI/CD pipeline the outputs and source images can point to alternative image repositories, such as Nexus, by using **DockerImage** in place of **ImageStreamTag**, see the example below. 
```
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: dotnetcore-ex
spec:
  output:
    to:
      kind: "DockerImage"
      name: "some-external-registry:5000/netcore-sample/dotnetcore-ex:latest"
  source:
    binary: {}
    type: Binary
  strategy:
    dockerStrategy:
      dockerfilePath: openshift/Dockerfile
      from:
        kind: "ImageStreamTag"
        name: "dotnet:6.0"
        namespace: openshift
```

## Additional Documentation 
Link to Redhat Documentation on DotNet on Openshift: [Getting started with .NET on RHEL 8](https://access.redhat.com/documentation/en-us/net/6.0/html/getting_started_with_.net_on_rhel_8/using_net_6_0_on_openshift_container_platform)

