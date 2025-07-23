pipeline {

  agent any

  environment {
    dockerimagename = "my-application"
    dockerTag = "${latest1}"
    registryCredential = 'docker-registry-creds'
  }

  stages {

    stage('Checkout Source') {
      steps {
        git branch: 'main', credentialsId: 'Jenkinscreds', url: 'https://github.com/Shri19-web/Kubernetes.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
           dockerImage = docker.build("131924/${dockerimagename}:${dockerTag}")
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

