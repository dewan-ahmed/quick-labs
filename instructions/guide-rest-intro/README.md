## Building and deploying a RESTful web service on OpenShift 4.3

### What you will learn

Red Hat OpenShift is a leading hybrid cloud, enterprise Kubernetes application platform. In this lab, you will learn how to build and deploy a simple REST service with JAX-RS and JSON-B on OpenShift 4.3. The REST service will respond to **GET** requests made to the /LibertyProject/System/properties REST endpoint.

The service responds to a **GET** request with a JSON representation of the system properties, where each property is a field  in a JSON object like this:
```JSON
{
  
  "user.timezone": "Etc/UTC",
  "java.vm.specification.version": "11",
  "os.name": "Linux",
  "server.output.dir": "/opt/ol/wlp/output/defaultServer/",
  
}
```
### Introduction

When you create a new REST application, the design of the API is important. The JAX-RS APIs can be used to create JSON-RPC, or XML-RPC APIs, but it wouldn't be a RESTful service. A good RESTful service is designed around the resources that are exposed, and on how to create, read, update, and delete the resources. The service responds to **GET** requests to the **/System/properties** path. The **GET** request should return a **200 OK** response that contains all of the JVM's system properties.

The platform where your application is deployed to is equally important as the design of your application/API. OpenShift provides a secure, scalable and universal way to build and deploy your application. Regardless of the infrastructure, OpenShift can run your application on private cloud, public cloud or physical machines. Although OpenShift offers multiple ways to build your application, you'll be building from your local files using binary build that matches close to a typical developer workflow. To learn more about OpenShift 4.3 build processes, refer to [this link](https://docs.openshift.com/container-platform/4.3/builds/understanding-image-builds.html). 

### Getting Started

You should see a terminal running. In case a terminal window does not open, navigate:

`Terminal -> New Terminal`

Check you are in the **home/project** folder:

`pwd`

The fastest way to work through this guide is to clone [this Git repository](https://github.com/dewan-ahmed/guide-rest-intro.git) and use the maven projects that are provided inside.

```
git clone https://github.com/dewan-ahmed/guide-rest-intro.git
```

The **finish** directory in the root of this guide contains the finished application. Due to limitation of time, we suggest using **finish** directory for this exercise. The **start** directory provides a skeleton of the finished project and you can follow the steps [in this guide](https://openliberty.io/guides/rest-intro.html) to add the missing pieces of the application yourself (at a later time and on your local environment).

### Understanding a JAX-RS application

JAX-RS has two key concepts for creating REST APIs. The most obvious one is the resource itself, which is modelled as a class. The second is a JAX-RS application, which groups all exposed resources under a common path. You can think of the JAX-RS application as a wrapper for all of your resources.

Click on File-->Open-->**guide-rest-intro**-->**finish**-->**src**/**main**/**java**/**io**/**openliberty**/**guides**/**rest**/**SystemApplication.java**

The **SystemApplication** class extends the **Application** class, which in turn associates all JAX-RS resource classes in the WAR file with this JAX-RS application, making them available under the common path specified in the **SystemApplication** class. The **@ApplicationPath** annotation has a value that indicates the path within the WAR that the JAX-RS application accepts requests from.

The expectation is that when you repeat this lab yourself using the **start** directory, you'll be creating the SystemApplication class yourself. 

### Understanding the JAX-RS resource

In JAX-RS, a single class should represent a single resource, or a group of resources of the same type. In this application, a resource might be a system property, or a set of system properties. It is easy to have a single class handle multiple different resources, but keeping a clean separation between types of resources helps with maintainability in the long run.

Click on File-->Open-->**guide-rest-intro**-->**finish**-->**src**/**main**/**java**/**io**/**openliberty**/**guides**/**rest**/**PropertiesResource.java**

This resource class has quite a bit of code in it, so let's break it down into manageable chunks.

The **@Path** annotation on the class indicates that this resource responds to the **properties** path in the JAX-RS application. The **@ApplicationPath** annotation in the **SystemApplication** class together with the **@Path** annotation in this class indicates that the resource is available at the **System/properties** path.

JAX-RS maps the HTTP methods on the URL to the methods on the class. The method to call is determined by the annotations that are specified on the methods. In the application you are building, an HTTP **GET** request to the **System/properties** path results in the system properties being returned.

The **@GET** annotation on the method indicates that this method is to be called for the HTTP **GET** method. The **@Produces** annotation indicates the format of the content that will be returned. The value of the **@Produces** annotation will be specified in the HTTP **Content-Type** response header. For this application, a JSON structure is to be returned. The desired **Content-Type** for a JSON response is **application/json** with **MediaType.APPLICATION_JSON** instead of the **String** content type. Using a constant such as **MediaType.APPLICATION_JSON** is better because if there's a spelling error, a compile failure occurs.

JAX-RS supports a number of ways to marshal JSON. The JAX-RS 2.1 specification mandates JSON-Binding
(JSON-B) and JAX-B. 

The method body returns the result of **System.getProperties()** that is of type **java.util.Properties**. Since the method 
is annotated with **@Produces(MediaType.APPLICATION_JSON)**, JAX-RS uses JSON-B to automatically convert the returned object
to JSON data in the HTTP response.

The expectation is that when you repeat this lab yourself using the **start** directory, you'll be creating the PropertiesResource class yourself. 


### Understanding the server configuration

To get the service running, the Liberty server needs to be correctly configured. 

Click on File-->Open-->**guide-rest-intro**-->**finish**-->**src**/**main**/**liberty**/**config**/**server.xml**

The configuration does the following actions:

. Configures the server to enable JAX-RS. This is specified in the **featureManager** element.
. Configures the server to resolve the HTTP port numbers from variables, which are then specified in the Maven **pom.xml** file. This is specified in the **<httpEndpoint/>** element. Variables use the **${variableName}** syntax. 
. Configures the server to run the produced web application on a context root specified in the **pom.xml** file. This is specified in the **<webApplication/>** element.

Take a look at the pom.xml file. Click on File-->Open-->**guide-rest-intro**-->**finish**-->**pom.xml**

The variables that are being used in the **server.xml** file are provided by the properties set in the Maven **pom.xml** file. The properties must be formatted as **liberty.var.variableName**.

### Building and running the application locally

(Assuming you have already cloned the repositoty and are under **guide-rest-intro** folder)

To try out the application locally, first go to the **finish** directory and run the following Maven goal to build the application and deploy it to Open Liberty:

`pwd` (this should show /home/project/guide-rest-intro/finish)

`mvn liberty:run`

Click on the **Launch Application** tab at the top and enter "9080" for the port. This will take you to the OpenLiberty landing page (some images might not load properly due to the page being loaded via proxy). To view the system properties, append "/LibertyProject/System/properties" after the URL and you should be seeing a long list of parameters like below:

{
  ...
  "user.timezone": "Etc/UTC",
  "java.vm.specification.version": "11",
  "os.name": "Linux",
  "server.output.dir": "/opt/ol/wlp/output/defaultServer/",
  ...
}

Remember to hit **ctrl+c** to stop the server when you're done.

### Deploying the application on OpenShift

(Assuming you have already cloned the repositoty and are under **guide-rest-intro/finish** folder)

Step 1: Creating an OpenShift project
```
oc new-project think-quicklabs
```

Step 2: Create a new binary build
```
oc new-build --name=rest-quicklab --binary --strategy=docker
```

Step 3: Start the binary build using current directory as binary input for the build
```
oc start-build rest-quicklab --from-dir=.
```

Step 4: Observe the build via OpenShift Console

Click on the OpenShift Console tab at the top. From the **Administrator** perspective, esnure that you're on the right ptoject - **think-quciklabs**. Click on **Builds**-->**Builds** tab and then **rest-quicklab** build. Click on the **Logs** tab and ensure a "Push successful" message is displayed at the end. This means that your build image has been pushed to OpenShift internal image registry.

Step 5: Create a new OpenShift app from the build
```
oc new-app rest-quicklab
```
This command creates OpenShift deploymentconfigs and services for your application.

Step 6: Exposing the app by creating a route to your application
```
oc expose svc/rest-quicklab
```
This command ensures that your app is accessible from the internet by a public URL.

Step 7: 

Click on the OpenShift Console tab at the top. From the **Developer** perspective, esnure that you're on the right ptoject - **think-quciklabs** and click on **Topology** tab. This shows your deployed application on OpenShift. To access the application, click on the **Launch Application** tab at the top and enter "9080" for the port. This will take you to the OpenLiberty landing page. Append **/LibertyProject/System/properties** at the end of the address bar and you should see the system properties of the server this OpenLiberty application is running on. 

Step 8:

Let's clean up the resources we just created. Click on the OpenShift Console tab at the top. From the **Administrator** perspective, click on **Home**-->**Projects** and find **think-quicklabs** project. Click on three vertical dots at the last column beside this project name and click "Delete Project". You will need to type the name of the project which you just created and the name has to match. If you followed the same naming convention, type "think-quicklab" and click "Delete".
On your own OpenShift cluster, if you'd like to delete the app only and not the project, you can execute the following command:
```
oc delete all -l app=<name-of-app>
```

### Summary

**Well Done**

Nice work! You have laerned how to develop a REST service in Open Liberty by using JAX-RS and JSON-B and then deployed the application on OpenShift 4.3.
