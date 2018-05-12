# jenkins-pipeline-workshop

## purpose
Getting up and running with a simple setup of Jenkins which will enable us to develop pipelines and pipeline libraries.
Workshop participants should come with this setup up and running on their laptops, so we can focus on `pipeline` it's different features / options.


## Method
Instead of explaining all the things related to a Jenkins setup (plugins mainly), we will be using a paradigm `Jenkins` supports which is basically using a bunch of groovy scripts to setup our environment.
My bloody Jenkins us written in
In order to be able to learn the pipeline we assume you will have a:
1. Jenkins instance,
2. A git repository with a Dockerfile
3. An ECR / DockerHub account so we can push the image there once were done ...

## My Bloody Jenkins - and opinionated (exactly what we need ...) setup
This example will:
1. setup a Jenkins instance
2. Configure it via a config.yaml file
3. Setup Docker as a `build cloud provider` to Jenkins
3. Instantiate a seed job

## Pre-requirements
1. git
1. A git account
1. Docker
1. docker-compose

## Setup steps

#### 1. Copy the `config.yml.template` to `config.yml`
``` cp config.yml.template config.yml ```

#### 2. prepare a git account with a private ssh-key and add it to the credentials section in `config.yml`
```
gitsshkey:
  type: sshkey
  description: gitsshkey
  username: shelleg
  passphrase: my-ssh-key-passphrase
  privatekey: |
    -----BEGIN RSA PRIVATE KEY-----
    your ssh should be here, with all lines idented to the left as this line is ...
```
#### 3. prepare a docker hub account credentials and add it to the credentials section in `config.yml`
```
dockerHub:
  type: userpass
  username: <dockerhub username>
  password: <dockerhub password>
```

#### 4. Choose a repository with a Dockerfile + Jenkinsfile you wish to build and push to dockerHub
**Please note that for the workshop, the config.yml template file already includes the correct repository settings.**

Please make sure the DockerHub org you are pushing to is defined in your Jenkinsfile (line # 14 -> `PROJECT = 'yourorg/yourimagename'`), you can use the following one as a reference:

```
pipeline
{
    options {
      buildDiscarder(logRotator(numToKeepStr: '5'))
    }
      agent {
      node {
        label 'generic'
        }
    }
    environment
    {
        VERSION   = 'latest'
        PROJECT   = 'shelleg/gitbook'
        IMAGE     = "${PROJECT}/${VERSION}:latest"
    }
    stages
    {
        stage('Build preparations')
        {
            steps
            {
                script
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                    IMAGE_LATEST = "$PROJECT:latest"
                }
            }
        }
    stage('Docker build') {
      steps {
          script {
                  // Build the docker image using a Dockerfile
                  docker.build("$IMAGE")
                  // override the latest tag
                  docker.build("$IMAGE_LATEST")
      		        }
	          }
    }
    stage('Publish') {
      steps {
        withDockerRegistry([ credentialsId: "dockerHub", url: "" ]){
          script {
                	// version eq latest git tag
                	docker.image("$IMAGE").push()
                	// override the latest tag
                	docker.image("$IMAGE_LATEST").push()
        	}
        }
      }
    }
  }
}
```

#### 5. set environment variable named `HOST_IP`
```
export HOST_IP="$(ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1)"
```

#### 6. Copy the `docker-compose.yml.template` to `docker-compose.yml`
``` cp docker-compose.yml.template docker-compose.yml ```

#### 7. Start Jenkins

At this stage run:

```
docker-compose up -d
```

Please note: if it's the first time running my bloody jenkins it will take time to download the image, + perform a reload to Jenkins.

Once done you should have something like the following:
![](https://www.tikalk.com/media/gittbook-docker__Jenkins_.png)
