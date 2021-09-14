---
title: Expose spring boot microservice with ingress using helm
date: 2020-06-17 18:07:00 Z
categories: [Spring Boot]
tags: [minikube, helm, ingress, kubernetes]
cover: /images/fsqs-tips-tricks-notes.png
---

In k8s deployment, [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
In this article, I will take you through how to expose spring boot microservice for the outside world in k8s deployment using ingress.
<!-- more -->

This example needs `kubectl` `minikube` `helm` command-line tools in your machine.

## Simple Spring Boot Microservice

Let's create one simple spring boot microservice that just returns the given name. 

### Initialize Project

```shell
NAME='Expose spring boot microservice with ingress using helm' && PRJ=expose-spring-boot-microservice-with-ingress-using-helm && \
mkdir -p $PRJ && cd $PRJ && \
curl https://start.spring.io/starter.tgz \
    -d dependencies=actuator,webflux \
    -d groupId=io.github.bhuwanupadhyay -d artifactId=$PRJ -d packageName=io.github.bhuwanupadhyay.example \
    -d applicationName=Spring Boot -d name=$NAME -d description=$NAME \
    -d language=kotlin -d platformVersion=2.3.1.RELEASE -d javaVersion=11 \
    -o demo.tgz && \
    tar -xzvf demo.tgz && rm -rf demo.tgz
```

### Simple API to return given name

```kotlin
@Configuration
class NameRoutes(private val handler: NameHandler) {

    @Bean
    fun router() = router {
        accept(APPLICATION_JSON).nest {
            GET("/names/{given-name}", handler::findGivenName)
        }
    }

}
```

## Containerizing Spring Boot Application

From Spring Boot [2.3.0.RELEASE](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/maven-plugin/reference/html/#build-image)
the maven plugin of spring boot by default support `build-image` goal during execution which creates an [OCI image](https://github.com/opencontainers/image-spec) using [Cloud Native Buildpacks](https://buildpacks.io/).

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <executions>
   <execution>
     <goals>
      <goal>build-image</goal>
     </goals>
     <configuration>
      <imageName>docker.io/bhuwanupadhyay/${project.artifactId}:${project.version}</imageName>
     </configuration>
   </execution>
  </executions>
</plugin>
``` 

Run `mvn clean install` : spring boot maven plugin will create a docker image. The end part of the output log:

```shell
[INFO]
[INFO] Successfully built image 'docker.io/bhuwanupadhyay/expose-spring-boot-microservice-with-ingress-using-helm:0.0.1-SNAPSHOT'
[INFO]
```

To publish docker image in a registry run the following command

```shell
docker push docker.io/bhuwanupadhyay/expose-spring-boot-microservice-with-ingress-using-helm:0.0.1-SNAPSHOT
```

## Helm Chart

To create a helm chart from your project directory run the following command.

```shell
helm create src/microservice
```

Replace value image repository and tag with your published docker image name and tag in `src/microservice/values.yaml` inside a helm chart.

```yaml
image:
  repository: docker.io/bhuwanupadhyay/expose-spring-boot-microservice-with-ingress-using-helm
  pullPolicy: IfNotPresent
  tag: "0.0.1-SNAPSHOT"
```

In your helm chart under `src/microservice/templates/deployment.yaml` change `readinessProbe` and `livenessProbe` health check settings, also modify container port to `8080` is the default for spring boot application.

```yaml
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 60
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 5
            timeoutSeconds: 5
            failureThreshold: 5
```

## Enable Ingress in Helm

Simply modify file `src/microservice/values.yaml` accordingly.

```yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: microservice.minikube
      paths:
        - "/"
``` 

## Deployment

Get ready for the deployment!

#### Start Minikube and Enable ingress
```shell
minikube start
```

#### Minikube enable addons ingress and ingress-dns in 
```shell
minikube addons enable ingress
minikube addons enable ingress-dns
```

#### Helm deployment
```shell
helm upgrade \
    --install -f src/microservice/values.yaml \
    example-deployment src/microservice --force
```

#### Watch the deployment and ingress
```shell
watch kubectl get pods
```

#### Modify `/etc/hosts` to add your host
```shell
# Know your host and address -> Run the following command
kubectl get ingress

# Output
NAME                              CLASS    HOSTS                   ADDRESS      PORTS   AGE
example-deployment-microservice   <none>   microservice.minikube   172.17.0.2   80      63m

# Add your host -> Run the following command
sudo sed -i "$ a 172.17.0.2 microservice.minikube" /etc/hosts
```

#### Test Microservice APIS
Firstly, install `httpie` command line tool.
```shell
sudo apt install httpie
```

`Open New Terminal` - Call microservice APIS

```shell
# GET given name
http http://microservice.minikube/names/k8sname

# Output
{
    "givenName": "k8sname"
}
```

We are done ! Thanks for reading. [Github](https://github.com/BhuwanUpadhyay/expose-spring-boot-microservice-with-ingress-using-helm) 

## References
- Versions: `helm: v3.2.3` `minikube: v1.11.0` `kubectl: v1.17.0`
- https://docs.spring.io/initializr/docs/current/reference/html/
- https://buildpacks.io/
- https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/maven-plugin/reference/html/
