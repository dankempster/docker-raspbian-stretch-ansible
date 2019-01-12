#!/usr/bin/env groovy

def IMAGE_NAME = "dankempster/raspbian-stretch-ansible"
def IMAGE_TAG = "build"

pipeline {

  agent {
    label 'raspberrypi'
  }

  stages {

    stage('Build') {
      steps {
        
        // Ensure we have the latest base docker image
        sh "docker pull \$(head -n 1 Dockerfile | cut -d \" \" -f 2)"

        // Work out the correct tag to use
        script { 
          if (env.BRANCH_NAME == 'develop') {
            IMAGE_TAG = 'develop'
          }
          else if (env.BRANCH_NAME == 'master') {
            IMAGE_TAG = 'latest'
          }
          else {
            IMAGE_TAG = 'build'
          }
        }
        
        // Build the image
        sh "docker build -f Dockerfile -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('Ansible Test') {
      steps {
        script {
          CONTAINER_ID = sh(
            script: "docker run --detach --privileged --volume=/sys/fs/cgroup:/sys/fs/cgroup:ro ${IMAGE_NAME}:${IMAGE_TAG}",
            returnStdout: true
          ).trim()
        }
        
        sh "docker exec --tty ${CONTAINER_ID} env TERM=xterm ansible --version"

        sh """
          docker stop ${CONTAINER_ID}
          docker rm ${CONTAINER_ID}
        """
      }
    }
    
    stage('Publish') {
      when {
        branch 'master'
      }
    
      steps {
        withDockerRegistry([credentialsId: "com.docker.hub.dankempster", url: ""]) {
          sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
        }
      }
    }
  }
}
