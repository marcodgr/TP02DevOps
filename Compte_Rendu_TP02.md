
# TP 2
## Setup Github Actions

### Build and test your application
on build l'application du tp 1 avec 
`mvn clean verify`


on push ensuite tout le projet sur git hub

on créer un fichier mail.yml dans le dossier .github/workflows

le voici : 

```
name: CI devops 2022 CPE
on:
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3
      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'zulu' # See 'Supported distributions' for available options
          java-version: '11'
      #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify --file backend-API/simple-api/

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: backend-API/simple-api
# Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:simple-api
      - name: Build image and push database
        uses: actions/checkout@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: database
      - name: Build image and push httpd
        uses: actions/checkout@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: http

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: backend-API/simple-api
# Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/simple-api:1
# build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}


      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: database
# Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/database:1
# build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}




      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: http
# Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/http:1
# build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}
```

## Quality gate
### Register to SonarCloud
on creer un compte, on ajoute repository github sur sonar cloud

ajout de lignes dans le yml pour passer par sonar cloud :
```
name: CI devops 2022 CPE
on:
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3
      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'zulu' # See 'Supported distributions' for available options
          java-version: '11'
      #finally build your app with the latest command
      - name: Build and test with Maven

        run: 
# mvn clean verify --file backend-API/simple-api/
          mvn -B verify sonar:sonar -Dsonar.projectKey=marcodgr_TP02DevOps -Dsonar.organization=marcodgr -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{secrets.SONAR_TOKEN }} --file backend-API/simple-api/pom.xml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}


      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: backend-API/simple-api
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:simple-api


      - name: Build image and push database
        uses: actions/checkout@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: database


      - name: Build image and push httpd
        uses: actions/checkout@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: http

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: backend-API/simple-api

          tags: ${{secrets.DOCKERHUB_USERNAME}}/simple-api:1
          push: ${{ github.ref == 'refs/heads/main' }}


      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: database
# Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/database:1
# build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
         # relative path to the place where source code with Dockerfile is located
          context: http
          tags: ${{secrets.DOCKERHUB_USERNAME}}/http:1
          push: ${{ github.ref == 'refs/heads/main' }}
```


