---
title: 'Deploying a Spring Boot Kotlin app on Kubernetes with Docker and Helm'
date: 2022-01-29 00:00:00
description: Steve Jackson - Portfolio Website
featured_image: '/images/demo/demo-square.jpg'
---
<h1>
<a id="user-content-h1" class="anchor" href="#h1" aria-hidden="true"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M7.775 3.275a.75.75 0 001.06 1.06l1.25-1.25a2 2 0 112.83 2.83l-2.5 2.5a2 2 0 01-2.83 0 .75.75 0 00-1.06 1.06 3.5 3.5 0 004.95 0l2.5-2.5a3.5 3.5 0 00-4.95-4.95l-1.25 1.25zm-4.69 9.64a2 2 0 010-2.83l2.5-2.5a2 2 0 012.83 0 .75.75 0 001.06-1.06 3.5 3.5 0 00-4.95 0l-2.5 2.5a3.5 3.5 0 004.95 4.95l1.25-1.25a.75.75 0 00-1.06-1.06l-1.25 1.25a2 2 0 01-2.83 0z"></path></svg></a>Deploying a Spring Boot Kotlin app on Kubernetes with Docker and Helm</h1>
The easiest hello-world application deployed on Kubernetes with Helm that you can put together.

[Code is hosted in Github](https://github.com/sjackson4430/demo-spring-boot)

Prerequisites (with the versions used in this tutorial):
Docker (Engine 19.03.13)
Minikube (1.15.1)
Helm (3.4.0)
Kubectl (matching kubernetes version)
Spring-boot app setup
On the spring initializr, create your app project. In this case we go with Gradle, Kotlin and Spring Boot 2.4.0 for Java 15 packaged in a JAR file. For the dependencies, let's select Spring Web so that we can initialize a Hello World REST API:
<br>
Alt Text
<br>
After generating it, you should have your project folder, which is the root to our repository.
<br>
Writing a hello-world REST API
for a simple REST API, just create a controller folder on the same folder as your DemoSpringBootApplication.kt, create an Application.kt file inside and add this content:
<br>
package com.gateixeira.demospringboot.controller
<br>
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController
<br>
@RestController
class Application {
    @GetMapping("/")
    fun home(): String {
        return "Hello World";
    }
}
Build and run app
The generated zip file comes with an embedded Gradle executable, so in order to build our app and make sure that everything is working just do a ./gradlew build:
<br>
~/code/demos/demo-spring-boot master > ./gradlew build
Starting a Gradle Daemon (subsequent builds will be faster)
<br>
Task :test
2020-11-29 20:44:55.227  INFO 43582 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
<br>
BUILD SUCCESSFUL in 8s
7 actionable tasks: 3 executed, 4 up-to-date
This process generates a jar file under build/libs. This is our executable. You can run it with java -jar build/libs/<your-app>. Your terminal will show the Spring banner and the following message:
<br>
DemoSpringBootApplicationKt : Started DemoSpringBootApplicationKt in 1.492 seconds (JVM running for 1.852)
<br>
Now if you go to localhost:8080 you see "Hello World".
<br>
Creating docker image
Now let's write a very simple dockerfile to generate our image. We are going to use Open JDK's alpine version as our base image, with Java 15 to match what we chose above.
<br>
FROM openjdk:15-jdk-alpine
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
It simply copies the generated JAR file to the container as app.jar and uses this as entrypoint for a Java application.
<br>
Writing Helm chart
Helm will be used to manage our application lifecycle inside Kubernetes. This will probably be the simplest chart possible but can easily be extended.
<br>
First, create a charts folder on your project's root. Then inside, create a Chart.yaml that specifies basic chart information:
<br>
apiVersion: v2
name: helm
description: "A Helm chart for our demo-spring-boot application"
type: application
version: 0.1.0
appVersion: latest
In the charts folder, create a templates folder and add a deployment.yaml inside:

apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Values.namespace }}
  name: {{ .Values.appName }}
  labels:
    app: {{ .Values.appName }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
        - name: "{{ .Values.appName }}"
          image: "{{ .Values.image.registry }}/{{ .Values.appName }}:{{ .Values.appVersion }}"
          imagePullPolicy: "{{ .Values.image.pullPolicy }}"
      restartPolicy: Always
Everything defined as {{ variable name }} is a placeholder and will be set in the values file.

This is a very simple deployment file that specifies some metadata, labels match and selectors as well as the name of the container, it's image repository address and pull policy.

Now in the same folder, create a service.yaml file:

apiVersion: v1
kind: Service
metadata:
  namespace: {{ .Values.namespace }}
  name: {{ .Values.appName }}
  labels:
    app: {{ .Values.appName }}
spec:
  type: LoadBalancer
  loadBalancerIP: 172.16.0.1
  sessionAffinity: None
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
  selector:
    app: {{ .Values.appName }}
This file defines the LoadBalancer configuration for the application, with an IP Address as entry point and the respective port in which the service is exposed (80) and the app is running (8080). Selector specifies the deployment to which this service relates.

Now in order to connect everything and replace the placeholders, let's create a values.yaml file in the charts folder:

appName: demo-spring-boot
namespace: default
appVersion: latest
replicaCount: 1
image:
  registry: gateixeira
  pullPolicy: Never
Our app will be on the default namespace, have a latest version and just 1 single replica. The container image will be composed of the tag that was added on the docker build command. In this case: gateixeira as the container registry, the app name as image name and concatenated with the version. pullPolicy is set to never since we are building a local image.

Deploying to Kubernetes
Starting Minikube
Our Kubernetes will be running locally with Minikube:

~/code/demos/demo-spring-boot master > minikube start --memory 2048 --cpus 2 --disk-size 10g --kubernetes-version v1.18.8
<br>
😄  minikube v1.15.1 on Darwin 10.15.6
<br>
❗  Both driver=docker and vm-driver=virtualbox have been set.

    Since vm-driver is deprecated, minikube will default to driver=docker.

    If vm-driver is set in the global config, please run "minikube config unset vm-driver" to resolve this warning.
<br>
✨  Using the docker driver based on user configuration
<br>
👍  Starting control plane node minikube in cluster minikube
<br>
🔥  Creating docker container (CPUs=2, Memory=2048MB) ...
<br>
🐳  Preparing Kubernetes v1.18.8 on Docker 19.03.13 ...
<br>
🔎  Verifying Kubernetes components...
<br>
🌟  Enabled addons: storage-provisioner, default-storageclass
<br>
❗  /usr/local/bin/kubectl is version 1.16.7, which may have incompatibilites with Kubernetes 1.18.8.
    ▪ Want kubectl v1.18.8? Try 'minikube kubectl -- get pods -A'
<br>
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
Now our app should be ready to be deployed on Kubernetes. First, let's build the docker image.

Start by switching the docker environment to use Minikube's daemon, otherwise Minikube can't find the image:
<br>
eval $(minikube docker-env)
<br>
For those running on Windows, the powershell equivalent is
<br>
minikube docker-env | Invoke-Expression
<br>
Then build the image with docker build -t <image_name>:<image_tag> .
<br>
~/code/demos/demo-spring-boot master > docker build -t gateixeira/demo-spring-boot:latest .  
<br>                                                                                   4s
Sending build context to Docker daemon  23.16MB
Step 1/4 : FROM openjdk:15-jdk-alpine
 ---> f02adfce91a2
Step 2/4 : ARG JAR_FILE=build/libs/*.jar
 ---> Using cache
 ---> 9bb20485adba
Step 3/4 : COPY ${JAR_FILE} app.jar
 ---> e651311f698a
Step 4/4 : ENTRYPOINT ["java", "-jar", "/app.jar"]
 ---> Running in 0ed363cc6d8b
Removing intermediate container 0ed363cc6d8b
 ---> 0033b968a8a9
<br>
Successfully built 0033b968a8a9
Successfully tagged gateixeira/demo-spring-boot:latest
<br>
To install your application on Minikube, run:
helm upgrade --install demo-spring-boot charts --values charts/values.yaml. Where charts is your charts folder.
<br>
~/code/demos/demo-spring-boot master > helm upgrade --install demo-spring-boot charts --values charts/values.yaml                                               
<br>
INT kube minikube

<br>
WARNING: "kubernetes-charts.storage.googleapis.com" is deprecated for "stable" and will be deleted Nov. 13, 2020.
WARNING: You should switch to "https://charts.helm.sh/stable"
Release "demo-spring-boot" does not exist. Installing it now.
NAME: demo-spring-boot
LAST DEPLOYED: Sun Nov 29 21:25:39 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
You can check your application is running with kubectl get pods:
<br>
~/code/demos/demo-spring-boot master !1 > kubectl get pods                                                                                                          kube minikube
NAME                               READY   STATUS    RESTARTS   AGE
demo-spring-boot-684cc98cc-zxgfv   1/1     Running   0          101s
With your app running, Minikube can tunnel your application and provide a URL to access it:

~/code/demos/demo-spring-boot master !1 > minikube service demo-spring-boot --url
<br>
🏃  Starting tunnel for service demo-spring-boot.


http://127.0.0.1:61460
Congratulations! 😄

If you open your browser on http://127.0.0.1:61460 you should see Hello World.