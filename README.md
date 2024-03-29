# Jenkins Shared Library

This repository contains a shared library for Jenkins, allowing for easier and more consistent pipeline scripting across projects. Below are the steps for setting up and using this shared library.

## Creating the Shared Library

- **Create Repository:** Create a new repository on GitLab to host the shared library.
- **Write the Groovy Code:** Implement the necessary Groovy scripts for the shared library. The scripts are organized as follows:
  - `vars` folder: Contains Groovy scripts representing reusable functions.
    - `buildJar.groovy`: Script for building the application using Maven.
    - `buildImage.groovy`: Script for building and pushing Docker images.
    - `dockerLogin.groovy`: Script for logging in to Docker registry.
    - `dockerPush.groovy`: Script for pushing Docker images to a registry.
  - `src` folder: Contains Groovy classes for organizing logic.
    - `com/example/Docker.groovy`: Class defining Docker-related actions.
  - `resources` folder: Placeholder for any additional resources needed.
- **Create Repository on GitLab:** Create a new repository on GitLab and push the shared library code to it.

<img src="https://i.imgur.com/cQDQSAs.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*****

<img src="https://i.imgur.com/4paGfkP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


****

## Make the Shared Library Available

- **Globally:** Configure Jenkins to make the shared library globally available.
- **Project scoped:** Reference the library directly from the Jenkinsfile.

<img src="https://i.imgur.com/maHObAN.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

****

## Using the Shared Library in Jenkinsfile

The Jenkinsfile in your project can now reference the shared library to simplify pipeline scripting. Here's an example Jenkinsfile:

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    buildImage 'aminaaahmed323/demo-app:jma-3.0'
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }
    }
}

```

<img src="https://i.imgur.com/oIqrurd.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

## Using Parameters in Shared Library

You can also pass parameters to shared library functions. For example, in `buildImage.groovy`, we can accept an image name as a parameter

```groovy
def call(String imageName) {
    echo "building the docker image..."
    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        sh "docker build -t $imageName ."
        sh "echo $PASS | docker login -u $USER --password-stdin"
        sh "docker push $imageName"
    }
}
```

<img src="https://i.imgur.com/Zy45NCm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

****

## Extracting Logic into Groovy Classes

You can organize your logic into Groovy classes for better maintainability. For example, `Docker.groovy`:

```groovy

package com.example

class Docker implements Serializable {

    def script
    
    Docker(script) {
        this.script = script
    }

    def buildDockerImage(String imageName){
        script.echo "building the docker image..."
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            script.sh "docker build -t $imageName ."
            script.sh "echo '${script.PASS}' | docker login -u '${script.USER}' --password-stdin"
            script.sh "docker push $imageName"
        }
    }

    def dockerLogin() {
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            script.sh "echo '${script.PASS}' | docker login -u '${script.USER}' --password-stdin"
        }
    }
    
    def dockerPush(String imageName) {
        script.sh "docker push $imageName"
    }
}

```
****

## Splitting "buildDockerImage" into Separate Steps

You can split complex tasks into separate steps within the shared library. For example, in `Docker.groovy`:

```groovy

def buildDockerImage(String imageName){
    script.echo "building the docker image..."
    script.sh "docker build -t $imageName ."
}

def dockerLogin() {
    script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        script.sh "echo '${script.PASS}' | docker login -u '${script.USER}' --password-stdin"
    }
}

def dockerPush(String imageName) {
    script.sh "docker push $imageName"
}

```
****

<img src="https://i.imgur.com/03S4WEQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

****
<img src="https://i.imgur.com/jY81mZG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
****

## Project Scoped Shared Library

For project scoped shared libraries, remove the global library configuration and reference the library directly from the `Jenkinsfile`:


<img src="https://i.imgur.com/hlkYLI6.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/7EwHMcH.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> 


