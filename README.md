
# CICD-DEMO

This project aims to be the basic skeleton to apply continuous integration and continuous delivery.

## Topology

CICD Demo uses some kubernetes primitives to deploy:

* Deployment
* Services
* Ingress ( with TLS )

```bash
     internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
   --|-----|--
   [   Pods   ]

```

This project includes:

* Spring Boot java app
* Jenkinsfile integration to run pipelines
* Dockerfile containing the base image to run java apps
* Makefile and docker-compose to make the pipeline steps much simpler
* Kubernetes deployment file demonstrating how to deploy this app in a simple Kubernetes cluster

## Pipeline Setup

Pipelines exist at Travis.

Some pipelines are configured by **GitHub/Projects**. If you have created a repository in one of these, your project will be **automatically** built if it has a Jenkinsfile/Travis/Gitlab/CircleCI.

Other pipelines are configured manually under folders. You can create a project manually with the following steps:

How to run the app:

```make
make
```

## Jenkins Pipeline (Academic Exercise)

This repository includes a Jenkinsfile aligned to the exercise requirements:

* Checkout
* Build (Maven package)
* Docker Build (image for this app)
* Test (unit tests)
* Static Analysis (SonarQube) + Quality Gate
* Container Security Scan (Trivy)
* Deploy (local Docker run on master)

### Jenkins Agent Prereqs

* Jenkins running with Docker access (for example, mount `/var/run/docker.sock`).
* Docker CLI available inside Jenkins (you can build a custom image from [jenkins/Dockerfile](jenkins/Dockerfile)).
* SonarQube server configured in Jenkins with the name `sonarqube`.
* No extra cleanup plugin required (the pipeline uses `deleteDir()` in the post block).

### Local SonarQube

Start SonarQube locally:

```bash
docker-compose up -d sonarqube
```

Open `http://localhost:9000` (default credentials: `admin` / `admin`).
Create a token and configure it in Jenkins under the SonarQube server settings.

### Quality Gate (Security Hotspot)

To satisfy the gatekeeping requirement, configure a Quality Gate in SonarQube
that fails when Security Hotspots are above 0. The Jenkins pipeline enforces
this gate via `waitForQualityGate`.

### Trivy Scan

The pipeline runs Trivy using the official container image and fails the build
if CRITICAL vulnerabilities are found in the built image.

## Testing

Unit tests and integrations tests are separated using [JUnit Categories][].

[JUnit Categories]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html

### Unit Tests

```java
mvn test -Dgroups=UnitTest
```

Or using Docker:

```bash
make build
```

### Integration Tests

```java
mvn integration-test -Dgroups=IntegrationTests
```

Or using Docker:

```bash
make integrationTest
```

### System Tests

System tests run with Selenium using docker-compose to run a [Selenium standalone container][] with Chrome.

[Selenium standalone container]: https://github.com/SeleniumHQ/docker-selenium

Using Docker:

* If you are running locally, make sure the `$APP_URL` is populated and points to a valid instance of your application. This variable is populated automatically in Jenkins.

```bash
APP_URL=http://dev-cicd-demo-master.anzcd.internal/ make systemTest
```