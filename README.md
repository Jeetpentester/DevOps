# DevSecOps Trainee GitHub Build Lab

A hands-on repository for learning how developers build and package an application and how DevOps/DevSecOps teams integrate security checks into CI/CD.

## Learning objectives

By completing this lab, the trainee will learn to:

1. Clone a repository and understand its structure.
2. Identify the application language and dependency files.
3. Build a Java/Maven application.
4. Run unit tests.
5. Package the application as a JAR.
6. Build and run a Docker image.
7. Push source code to GitHub.
8. Understand GitHub Actions CI/CD.
9. Identify where SCA, SAST, Secret Scanning, IaC and Container scanning fit.
10. Understand the relationship between source code, build artifact, container image and deployment manifests.

## Repository security indicators

| File / Folder | What it indicates | Security control |
|---|---|---|
| `pom.xml` | Maven dependencies | SCA |
| `src/main/java` | Java source | SAST |
| `Dockerfile` | Container build | Container/Docker scan |
| `terraform/*.tf` | Infrastructure as Code | IaC scan |
| `k8s/*.yaml` | Kubernetes deployment | Kubernetes/IaC scan |
| `.github/workflows/*.yml` | CI/CD workflow | Pipeline security / security gates |

## 1. Developer build flow

```text
Developer
   |
   v
Git clone
   |
   v
Modify source code
   |
   v
Unit Test
   |
   v
Maven Build
   |
   v
JAR artifact
   |
   v
Docker Build
   |
   v
Container Image
   |
   v
Registry
   |
   v
Kubernetes / Runtime
```

## 2. DevSecOps security flow

```text
                    Pull Request / Push
                           |
                           v
                 +---------------------+
                 |   GitHub Actions    |
                 +---------------------+
                    |    |    |    |
                    |    |    |    +--> Secret Scan
                    |    |    +-------> IaC Scan
                    |    +------------> SCA
                    +-----------------> SAST
                           |
                           v
                       Build/Test
                           |
                           v
                      Docker Build
                           |
                           v
                    Container Scan
                           |
                           v
                    Quality Gate
                           |
                 +---------+---------+
                 |                   |
              PASS                  FAIL
                 |                   |
                 v                   v
             Package              Stop build
             / Deploy             Fix findings
```

## 3. Prerequisites

Install:

- Git
- JDK 17+
- Maven 3.8+
- Docker Desktop / Docker Engine
- A GitHub account

Verify:

```bash
git --version
java -version
mvn -version
docker --version
```

## 4. Run the application locally

From the repository root:

```bash
mvn clean test
mvn clean package
```

The JAR should be created under:

```text
target/
```

Run:

```bash
java -jar target/devsecops-demo-1.0.0.jar
```

The application exposes a simple health endpoint:

```text
http://localhost:8080/health
```

## 5. Build the Docker image

```bash
docker build -t devsecops-trainee-app:1.0 .
```

Check the image:

```bash
docker images
```

Run it:

```bash
docker run --rm -p 8080:8080 devsecops-trainee-app:1.0
```

Test:

```bash
curl http://localhost:8080/health
```

## 6. Understand the Docker build

The Dockerfile copies the Maven-built JAR into a runtime image.

The important distinction is:

- Maven build creates the application artifact.
- Docker build packages the artifact into a container image.
- Container scanning evaluates the image and its installed packages/configuration.
- SCA evaluates application dependencies.
- SAST evaluates source code.

## 7. Push the project to GitHub

Create an empty GitHub repository, for example:

```text
devsecops-trainee-build-lab
```

Then from the project directory:

```bash
git init
git branch -M main
git add .
git commit -m "Initial DevSecOps training lab"
git remote add origin https://github.com/<YOUR-USERNAME>/devsecops-trainee-build-lab.git
git push -u origin main
```

Replace `<YOUR-USERNAME>` with your GitHub username.

## 8. Branch and Pull Request exercise

Create a feature branch:

```bash
git checkout -b feature/update-health-message
```

Modify the application.

Then:

```bash
git add .
git commit -m "Update health message"
git push -u origin feature/update-health-message
```

Create a Pull Request from the feature branch to `main`.

Observe the GitHub Actions workflow.

## 9. Build pipeline

The workflow in:

```text
.github/workflows/ci.yml
```

performs:

1. Checkout
2. Set up Java
3. Maven dependency resolution
4. Unit tests
5. Maven package
6. Docker build

This is the basic CI build pipeline.

## 10. Where DevSecOps scans fit

A typical sequence is:

```text
Checkout
   |
   +--> Secret Scanning
   |
   +--> SAST
   |
   +--> SCA
   |
   +--> IaC Scan
   |
   +--> Unit Tests
   |
   +--> Build
   |
   +--> Docker Build
   |
   +--> Container Scan
   |
   +--> Quality Gate
   |
   +--> Publish / Deploy
```

The exact order can vary according to the organization's architecture and security policy.

## 11. Trainee exercises

### Assignment 1 — Repository identification

Identify every artifact in this repository and create a table:

| Artifact | Found? | Why it exists | Security scan |
|---|---|---|---|
| pom.xml | | | |
| Java source | | | |
| Dockerfile | | | |
| Terraform | | | |
| Kubernetes YAML | | | |
| GitHub Actions | | | |

### Assignment 2 — Local build

Run:

```bash
mvn clean test
mvn clean package
```

Capture the successful build output.

### Assignment 3 — Docker

Build and run:

```bash
docker build -t devsecops-trainee-app:1.0 .
docker run --rm -p 8080:8080 devsecops-trainee-app:1.0
```

### Assignment 4 — GitHub

Push the project to your own GitHub repository and create a feature branch.

### Assignment 5 — CI/CD

Modify the GitHub Actions workflow so that the pipeline:

- Builds the Java application.
- Runs tests.
- Builds the Docker image.
- Fails when a command fails.

### Assignment 6 — DevSecOps

Integrate security checks appropriate to the repository:

- SCA for `pom.xml`
- SAST for Java
- Secret scanning
- IaC scanning for `terraform/`
- Kubernetes configuration scanning for `k8s/`
- Container image scanning after Docker build

Document which tool you selected and why.

### Assignment 7 — Quality gate

Define a training quality gate, for example:

- Fail for High/Critical vulnerabilities.
- Define a separate threshold for Medium findings.
- Document exceptions instead of silently ignoring findings.

The exact thresholds should be aligned with the organization's security policy.

## 12. Questions for trainee review

1. Why does `pom.xml` indicate an SCA requirement?
2. Why does `src/main/java` indicate a SAST requirement?
3. Is every YAML file IaC? Why or why not?
4. Why should the Docker image be scanned after it is built?
5. What is the difference between SCA and SAST?
6. What is the difference between a Dockerfile scan and an image vulnerability scan?
7. Where should security checks run in a pull-request workflow?
8. What should happen when a quality gate fails?
9. Why should secrets never be committed to Git?
10. Which artifacts are deployed to the runtime environment?

## 13. Trainer outcome

At the end of this lab, the trainee should be able to look at an unfamiliar repository and answer:

> "What is this repository building, what dependencies does it use, what infrastructure does it define, what gets deployed, and which DevSecOps security controls should be applied?"

