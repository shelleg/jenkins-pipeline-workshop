# jenkins-pipeline-workshop - get ready for the workshop

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
1. git (cli installed on laptop)
1. A GitHub account
1. Docker (cli installed on laptop and run as daemon or connected to remote Docker server)
1. docker-compose (cli installed on laptop)
1. A Docker hub account (https://hub.docker.com/)
1. A local clone of this repository (for the template files in it), which will be your workspace for this workshop.
1. A [forked](https://help.github.com/articles/fork-a-repo/) clone of the https://github.com/shelleg/jenkins-pipeline-workshop-library repository.
1. A [forked](https://help.github.com/articles/fork-a-repo/) clone of the https://github.com/shelleg/docker-gitbook repository.

## Setup steps

#### 1. Copy the `config.yml.template` to `config.yml` inside your local repository workspace
``` cp config.yml.template config.yml ```

#### 2. prepare a git account with a private ssh-key and add it to the credentials section in `config.yml`
```
gitsshkey:
  type: sshkey
  description: gitsshkey
  username: < github username >
  passphrase: < your ssh key passphrase if no empty >
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
          script {
              docker.withRegistry("https://registry.hub.docker.com", "dockerHub"){
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
export HOST_IP="$(ifconfig | grep 'inet ' | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1 | sed -e 's/addr://')"
```

#### 6. Copy the `docker-compose.yml.template` to `docker-compose.yml`
``` cp docker-compose.yml.template docker-compose.yml ```

#### 7. Start Jenkins

At this stage run:

```
docker-compose up -d
```

Please note: if it's the first time running my bloody jenkins it will take time to download the image, + perform a reload to Jenkins.

#### 8. Browse to Jenkins and validate that build passed OK

Browse to http://localhost:8080 and login as 'admin' user (password is in the config.yml file).
After the server is loaded OK (after restart once for plugins update), a first build should start running.

Once done you should have something like the following:
![](https://www.tikalk.com/media/gittbook-docker__Jenkins_.png)

### Once you're done with last step, you're ready for the workshop.

The workshop tasks are described in the [next page](workshop.md).


