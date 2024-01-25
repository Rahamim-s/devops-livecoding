# devops-livecoding



# 1) Answer to Question 2-1: What are testcontainers?

Testcontainers are Java libraries that allow you to run Docker containers during your tests. In this project, they're used to spin up Docker containers, like the PostgreSQL container, to test the application. These Docker containers are launched during test execution to ensure that integration tests work correctly with a database.


# 2) Document your Github Actions configurations.
Documentation of :
# CI devops 2023 Workflow

This GitHub Actions workflow is designed for continuous integration (CI) for the devops 2023 project.

## Trigger Conditions
The workflow is triggered on:
- Push events to the 'main' and 'develop' branches.
- Pull requests targeting 'main' and 'develop' branches.

## Job - test-backend
This job runs on the latest Ubuntu environment and consists of the following steps:

1. Checkout code using actions/checkout@v2.5.0.
2. Set up JDK 17 using actions/setup-java@v3.
3. Build and test the Java project with Maven.

Ensure the working directory is set to ./simple-api.

```yaml
name: CI devops 2023

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
      - develop

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build and test with Maven
        working-directory: ./simple-api
        run: mvn clean install --batch-mode --show-version --errors
```

# 3) Document your Github Actions configurations.

# Java CI with Maven and Docker Build Workflow

This GitHub Actions workflow handles Java project building, testing, SonarCloud analysis, and Docker image building.

## Trigger Conditions
The workflow is triggered on:
- Push events to the 'main' and 'test' branches.
- Pull requests targeting 'main' and 'test' branches.

## Jobs
### Job - test-backend
This job runs on the latest Ubuntu environment and consists of the following steps:

1. Checkout code using actions/checkout@v3.
2. Set up JDK 20 using actions/setup-java@v3.
3. Build with Maven and perform SonarCloud analysis.

### Job - build-and-push-docker-image
This job runs on Ubuntu 22.04 and is conditional based on specific push events:

1. Checkout code using actions/checkout@v2.5.0.
2. Log in to DockerHub.
3. Build and push Docker images for 'simple-api,' 'database,' and 'http-server.'

Ensure DockerHub credentials are stored securely in GitHub secrets.

```yaml
name: Java CI with Maven and Docker Build

on:
  push:
    branches: [ "main", "test" ]  # Trigger on push to main and test (develop in your terms)
  pull_request:
    branches: [ "main", "test" ]  

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 20
        uses: actions/setup-java@v3
        with:
          java-version: '20'
          distribution: 'temurin'
          cache: maven
      - name: Build with Maven and SonarCloud analysis
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=Rahamim-s_devops-livecoding -Dsonar.organization=sonarara -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./simple-api/pom.xml

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2.5.0
      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push simple-api Docker image
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          file: ./simple-api/Dockerfile
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/simple-api:latest
          push: true
      - name: Build and push database Docker image
        uses: docker/build-push-action@v3
        with:
          context: ./database
          file: ./database/Dockerfile
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/database:latest
          push: true
      - name: Build and push http-server Docker image
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          file: ./http-server/Dockerfile
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/http-server:latest
          push: true
```
