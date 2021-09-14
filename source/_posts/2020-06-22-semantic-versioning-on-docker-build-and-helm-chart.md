---
title: Semantic versioning on docker build and helm chart
date: 2020-06-22 16:10:00 Z
categories: [Semantic Versioning]
tags: [helm, docker, semantic-versioning]
cover: /images/fsqs-tips-tricks-notes.png
---

Helm best practice guide advocate semantic versioning for the helm chart that your release for deployment. Wherever possible, Helm uses [SemVer 2](https://semver.org/) to represent version numbers. Semantic versioning is a meaningful method for incrementing version numbers. So, today we will explore how to release helm charts and docker build by using semantic versioning convention.

<!-- more -->


In this example, I will release semantic versions for helm chart and docker image of spring boot microservice that build upon maven.

## Docker build and Helm chart for spring boot

Let's start with spring boot a simple microservice that exposes API to return a given name.

### Initialize Project

```shell
NAME='Semantic versioning on docker build and helm chart' && PRJ=semantic-versioning-on-docker-build-and-helm-chart && \
mkdir -p $PRJ && cd $PRJ && \
curl https://start.spring.io/starter.tgz \
    -d dependencies=actuator,webflux \
    -d groupId=io.github.bhuwanupadhyay -d artifactId=$PRJ -d packageName=io.github.bhuwanupadhyay.example \
    -d applicationName=Spring Boot -d name=$NAME -d description=$NAME \
    -d language=kotlin -d platformVersion=2.3.1.RELEASE -d javaVersion=11 \
    -o demo.tgz && \
    tar -xzvf demo.tgz && rm -rf demo.tgz
```

### Create API to return given name

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

### Dockerfile for spring boot `src/main/docker/Dockerfile`

From Spring Boot 2.3.0.RELEASE they introduced [layertools](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/htmlsingle/#layering-docker-images) to create optimized Docker images that can be built with a dockerfile.

```dockerfile
FROM adoptopenjdk:11.0.7_10-jre-hotspot as builder
WORKDIR /app
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM adoptopenjdk:11.0.7_10-jre-hotspot
WORKDIR /app
COPY --from=builder app/dependencies/ ./
COPY --from=builder app/spring-boot-loader/ ./
COPY --from=builder app/snapshot-dependencies/ ./
COPY --from=builder app/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

### Helm chart for spring boot `src/main/helm/my-service`

Simply run the create helm command.

```shell
mkdir -p src/main/helm && helm create src/main/helm/my-service
```

In your helm chart under `src/main/helm/my-service/templates/deployment.yaml` change `readinessProbe` and `livenessProbe` health check settings, also modify container port to `8080` is the default for spring boot application.

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

Also, replace value image repository with your published docker image name without a tag in `src/main/helm/my-service/values.yaml` inside a helm chart.

```yaml
image:
  repository: docker.io/bhuwanupadhyay/my-service
  pullPolicy: IfNotPresent
  tag: ""  
```

In `src/main/helm/my-service/Chart.yaml` there are two properties:
 - `version` [chart version] - Versions are expected to follow Semantic Versioning (https://semver.org/)
    
    This is the chart version. This version number should be incremented each time you make changes. 
 
 - `appVersion` [default value for image tag] - Versions are expected to follow Semantic Versioning (https://semver.org/)
    
    This is the version number of the application being deployed. This version number should be incremented each time you make changes to the application.

In maven [pom.xml](https://github.com/BhuwanUpadhyay/semantic-versioning-on-docker-build-and-helm-chart/blob/master/pom.xml), 
the `flatten-maven-plugin` to set revision number for project, 
which will use by `dockerfile-maven-plugin` to build the docker image `repository/image-name:<revision>` with revision
and `helm-maven-plugin` create package with that revision for chart version and appVersion.  

It's very important to provide consistent releases during the life cycle of the product. To achieve this we will use very popular
tool [Semantic Release](https://semantic-release.gitbook.io/) with [Conventional Commits](https://www.conventionalcommits.org/).


## Semantic Release Process

![](/images/semantic-release-process.png)

## Github Pipeline

Let's create [semantic-release configuration](https://semantic-release.gitbook.io/semantic-release/usage/configuration) `.releaserc` file in your project directory.
In this configuration, we have two commands given by [@semantic-release/exec](https://github.com/semantic-release/exec) plugin that we will use to a build the release and publish helm package on [GitHub Packages](https://github.com/BhuwanUpadhyay/semantic-versioning-on-docker-build-and-helm-chart/packages) and docker image on [docker.io](https://hub.docker.com/).

- `prepareCmd`: The shell command to execute during the prepare step.
- `publishCmd`: The shell command to execute during the publish step.

```json
{
  "branches": ["master"],
  "plugins": [
      "@semantic-release/commit-analyzer",
      "@semantic-release/release-notes-generator",
      "@semantic-release/github",
      "@semantic-release/git",
      "@semantic-release/changelog",
      ["@semantic-release/exec", {
            "prepareCmd" : "./bot.sh --prepare ${nextRelease.version}",
            "publishCmd" : "./bot.sh --publish ${nextRelease.version}"
            }]
    ]
}
```

Here `bot.sh` is a script file used to `build` and `publish` the docker image and helm chart package using the next version given by a semantic release process.

```bash
set -e
option="${1}"
default_version='0.0.0-SNAPSHOT'
next_version="${2:-$default_version}"
case ${option} in
   --prepare)
      ./mvnw \
        -B -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn -V \
        clean install -Drevision="$next_version"
      ;;
   --publish)
      # Publish Docker in Github Packages
      DOCKER_PKG=docker.pkg.github.com/bhuwanupadhyay/semantic-versioning-on-docker-build-and-helm-chart/my-service:"$next_version"
      docker login docker.pkg.github.com -u BhuwanUpadhyay -p "$GITHUB_TOKEN"
      docker tag docker.io/bhuwanupadhyay/my-service:"$next_version" "$DOCKER_PKG"
      docker push "$DOCKER_PKG"

      # Publish Helm chart in Github Releases
      HELM_CHART="my-service-$next_version.tgz"
      HELM_CHART_FILE_PATH="$(pwd)/target/helm/repo/$FILE_NAME"

      # TODO: yourself
      # Write a suitable script to upload your helm chart in your Artifactory or chart museum.

      echo "Mock publish: $HELM_CHART from $HELM_CHART_FILE_PATH"
      ;;
   *)
      echo "`basename ${0}`:usage: [--prepare] | [--publish]"
      exit 1 # Command to come out of the program with status 1
      ;;
esac
```

Finally, we need a workflow action YAML configuration to run the Github pipeline under `.github/workflows/build.yml`.

```yaml
name: Java CI

on:
  push:
    branches:
      - '*'
jobs:
  build:
    runs-on: ubuntu-latest
    name: Build
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Build
        run: ./bot.sh --prepare
      - uses: actions/setup-node@v1
        name: Install semantic-release
        if: github.ref == 'refs/heads/master' && github.event_name == 'push'
        with:
          node-version: "12.x"
      - name: Release
        if: github.ref == 'refs/heads/master' && github.event_name == 'push'
        run: |
          npm install -g semantic-release @semantic-release/{git,changelog,exec}
          semantic-release
        env:
          GITHUB_TOKEN: ${{ github.token }}
```

We are done ! Thanks for reading. [Github](https://github.com/BhuwanUpadhyay/semantic-versioning-on-docker-build-and-helm-chart)

## References
- https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/htmlsingle/#layering-docker-images
