pipeline {

  agent any

  environment {
    dockerimagename = "my-application"
    dockerTag = "${BUILD_NUMBER}"
    registryCredential = 'docker-registry-creds'
  }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/Shri19-web/Kubernetes.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${dockerimagename}:${dockerTag}")
        }
      }
    }

    stage('Push Image to DockerHub') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
            dockerImage.push("${dockerTag}")
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(
            configs: 'deploymentservice.yaml',
            kubeconfigId: 'kubeconfig'
          )
        }
      }
    }

  }
}

