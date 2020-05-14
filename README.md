# Container Pipeline
This document explains how to use and control the example pipeline. Full description of the pipeline is in 
[var/exampleContainerPipeline.groovy](../vars/exampleContainerPipeline.groovy)

## Instantiating the Pipeline
In your application repo add a Jenkinsfile with the following content:
```groovy
@Library('sample-pipeline@ci-pipeline-refactor') _
properties([pipelineTriggers([githubPush()])])
  exampleContainerPipeline build_config: "build-info.yaml",
  env: env,
  cd_repo: "git@github.com:ops-guru/jenkins-mock-cd.git",
  cd_branch: "dev",
  cd_git_user: ["name": "Jenkins Worker", "email": "vlahovic@google.com"],
  cd_git_credentials: "github-ssh-marko7460",
  cd_jenkins_job: "mock-cd-pipeline"
```

The following input parameters are available:
* *build_config* (required)
   * Yaml file describing how the pipeline is controlled. See section on build-info.yaml
* *env* (required)
   * Pass current environment to the pipeline
* *cd_repo* (required)
   * repo of the SaasOPS pipeline
* *cd_git_credentials* (required)
   * Jenkins credentials id to use for cd_repo cloning. You should store an ssh private key in the jenkins credentials.
* *cd_branch* (required)
   * Which branch of *cd_repo* should we use
* *cd_git_user* (required)
   * Which git user should be used to clone `cd_repo`.
* *cd_jenkins_job* (required)
   * Name of the continous deployment job.
* *cd_jenkins_url* (optional)
   * If the CD job is on another jenkins master set this variable to be the URL of the CD jenkins. 
   Example: https://jenkins.titantesting.dev/jenkins/
* *cd_jenkins_credentials* (optional)
   * If the CD job is on another jenkins master set this variable to be used for authentication. This variable refers to
   a jenkins credentials id. In these credentials store API key/token from CD jenkins master.
   
## build-info.yaml
This yaml file defines building, testing, and publishing steps. There are three major sections:
1. **build_steps**
   1. List of build steps that will be executed inside specified containers
1. **test_steps**
   1. List of test steps that will be executed inside specified containers
1. **docker_build**
   1. List of docker images to build. These are product team's container images.
1. **helm_build**
   1. List of helm charts to build. These are product team's charts that deploy the microservice(s).
   
### build_steps
For examples see **build_steps** in sample [build-info.yaml](../resources/build-info.yaml)
```yaml
- name: <Name of the build stage. Can be any string> (required)
  pre_steps: <List of shell commands to execute before starting docker image.name> (optional)
    - "echo something" # <Example pre_step command>
  image: <Docker image definition> (required)
    name: <name of the docker image> (required if image.Dockerfile is not defined)
    docker_run_params: <Docker run parameters> (optional)
    Dockerfile: <location of the dockerfile with respect to "source"> (required if image.name is not defined)
  source: <location from within your repository that will be mounted to the image> (optional. Default: "." (Mount the whole repo))
  commands: <List of build commands that will be executed within the image.name conatiner> (required)
    - "echo mvn build"
  archive: <List of files that you want to reuse in other build steps. These files will be uploaded to GCS> (optional)
    - <archive_name: archive_files> # archive_name: name of the archive, archive_files: File(s) to be uploaded to archive
    - testfile1: output/testfile1.txt # output/testfile1.txt will be uploaded to gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/testfile1
  unarchive: <List of archives to download and extract. This is done before pre_steps and image> (optional)
    - testfile2 # This will download all the files from gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/testfile2/* to newly created directory testfile2 in your workspace
  artifacts: <List of files that will be uploaded to gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/artifacts/ > (optional)
    - targets/*.jar # Files will be placed in gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/artifacts/targets/*.jar
```

### test_steps
For examples see **test_steps** in sample [build-info.yaml](../resources/build-info.yaml)
```yaml
- name: <Name of the test stage. Can be any string> (required)
  pre_steps: <List of shell commands to execute before starting docker image.name> (optional)
    - "echo something" # <Example pre_step command>
  image: <Docker image definition> (optional)
    name: <name of the docker image> (required if image.Dockerfile is not defined)
    docker_run_params: <Docker run parameters> (optional)
    Dockerfile: <location of the dockerfile with respect to "source"> (required if image.name is not defined)
  source: <location from within your repository that will be mounted to the image> (optional. Default: "." (Mount the whole repo))
  commands: <List of build commands that will be executed within the image.name conatiner> (required)
    - "pytest ."
  archive: <List of files that you want to reuse in other build steps. These files will be uploaded to GCS> (optional)
    - <archive_name: archive_files> # archive_name: name of the archive, archive_files: File(s) to be uploaded to archive
    - tests1: output/tests.junit # output/testfile1.txt will be uploaded to gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/test1
  unarchive: <List of archives to download and extract. This is done before pre_steps and image> (optional)
    - tests2 # This will download all the files from gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/test2/* to newly created directory tests2 in your workspace
  artifacts: <List of files that will be uploaded to gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/artifacts/ > (optional)
    - junits/junit.xml # Files will be placed in gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/artifacts/targets/*.jar
  junit: resources/junit.xml <Location of the junit files. This is how tests are reported> (required)
```
sonar 
```yaml
sonar_conf:
  - sonar.projectKey: project key value
    sonar.projectName: name of the project
    sonar.projectVersion: version of the project 
    sonar.sources: path of the source code folder
    sonar.language: programming language of the project
    sonar.java.binaries: path to the binaries (for java)
    sonar.sourceEncoding : UTF-8
    sonar.web.host: URL to the sonarqube
    sonar.web.port: specify port
    sonar.login: token to login into sonarqube
    ```




### docker_build
For examples see **docker_build** in sample [build-info.yaml](../resources/build-info.yaml)
```yaml
- name: <Name of the docker image to build. Can be any string> (required)
  repo: <Repo to which the docker image will be uplaoded during publish stages> (required)
  pre_steps: <List of shell commands to execute before building the container> (optional)
    - "echo something" # <Example pre_step command>
  Dockerfile_dir: <Directory where dockerfile is located. The docker build will be executed from this directory> (required)
  Dockerfile_name: <Name of the dockerfile> (required)
  unarchive: <List of archives to download and extract. This is done before pre_steps and docker build> (optional)
    - jars # This will download all the files from gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/jars/* to newly created directory jars in your workspace
  version: "latest" <Version of the docker container> (Optional. Defualt value is git commit sha)
```

When this step is executed the container with the name `$repo}/${name}/${BRANCH_NAME}:${version}` will be created. The
container WILL NOT BE PUBLISHED with this step

### helm_build
For examples see **helm_build** in sample [build-info.yaml](../resources/build-info.yaml)
```yaml
- name: <Name of this helm stage. Can be any string> (required)
  pre_steps: <List of shell commands to execute before building the container> (optional)
    - "echo something" # <Example pre_step command>
  helm_chart: <Location of your helm chart files. Helm commands will be executed within this directory> (required)
  image: <Helm docker image definition. We execute helm commands within this docker images> (required)
    name: <name of the helm docker image> (required)
    docker_run_params: <docker run parameters> (optional)
  unarchive: <List of archives to download and extract. This is done before pre_steps and docker build> (optional)
    - jars # This will download all the files from gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/jars/* to newly created directory jars in your workspace
  commands: <List of helm commands executed within image.name>
      - helm init --client-only
      - helm lint
      - helm package .
```

When this step is executed the helm chart will be uploaded to `gs://$ARTIFACT_BUCKET/$JOB_NAME/$BUILD_NUMBER/archive/helm/`


deploy_config
```yaml
deploy_config:
  deploy: true
  cd_repo_dev: "git@github.gwd.broadcom.net:dockcpdev/gtso-helm-helloworld.git"
  cd_repo_verify: "git@github.gwd.broadcom.net:dockcp/gtso-helm-helloworld.git"
  cd_git_credentials: "sg037737_GitOps_SSH"
  cd_git_user:
    name: "Jenkins Worker"
    email: "chintan.vyas@broadcom.com"
  environments:
    dev:
      gke: us-west1-dev
    #feature-2:
    #  gke: us-east1-f2
    feature-cv-15:
        gke: gdu1
 ```       

image_scan
```yaml
  image_scan:
    skip_image_scan: "yes"
    severity_threshold: "warn"
    ignore_failures: "yes"
 ```   


## Flow
The following image describes the flow of stages in the pipeline.
![build-info-flow](images/build-info-flow.png)
As we can see from the above image, **build_steps** and **unit_tests** are running in parallel after which the 
**docker_build** and **helm_build** will execute in parallel.


   
   
```yaml
publish_config:
  docker:
    {dev}
  helm:
    {dev}
 ```
