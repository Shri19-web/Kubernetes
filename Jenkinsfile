pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-application'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_REGISTRY = '131924/kubernetes.com'
        K8S_NAMESPACE = 'my-application-ns'
        KUBECONFIG = credentials('kubeconfig') // Must be stored in Jenkins Credentials
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/Shri19-web/Kubernetes.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                echo "Repo cloned successfully!"
            }
        }
        
stage('Install Node.js') {
  steps {
    sh '''
      curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
      apt-get install -y nodejs
    '''
  }
}

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'npm ci'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh 'npm test'
                }
            }
            post {
                always {
                    junit testResults: 'test-results.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-creds') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    sh """
                        sed -i 's|IMAGE_TAG|${DOCKER_TAG}|g' k8s-manifests/deployment.yaml
                        kubectl apply -f k8s-manifests/ -n staging
                        kubectl rollout status deployment/${DOCKER_IMAGE} -n staging
                    """
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    input message: 'Deploy to production?', ok: 'Deploy'
                    sh """
                        sed -i 's|IMAGE_TAG|${DOCKER_TAG}|g' k8s-manifests/deployment.yaml
                        kubectl apply -f k8s-manifests/ -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment/${DOCKER_IMAGE} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
      }        
 }
