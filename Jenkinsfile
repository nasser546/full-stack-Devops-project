pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "mohamednasser22/nasser:1.0"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']], 
                    extensions: [], 
                    userRemoteConfigs: [[url: 'https://github.com/nasser546/full-stack-Devops-project.git']]
                )
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                trivy fs \
                  --scanners vuln \
                  --severity CRITICAL,HIGH \
                  --exit-code 0 .
                '''
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonnar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=twitter-app \
                          -Dsonar.sources=src \
                          -Dsonar.java.binaries=target \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image \
                  --timeout 10m \
                  --scanners vuln \
                  --severity CRITICAL,HIGH \
                  --exit-code 0 \
                  ${IMAGE_NAME}
                """
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Quick Test)') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-jenkins', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl apply -f deployment-service.yml
                        kubectl rollout status deployment/bloggingapp-deployment
                    '''
                }
            }
        }
    }
}

