pipeline {
    agent any

    tools {
        maven 'maven'  // Ensure Maven is configured in Jenkins
    }

    environment {
        JAR_NAME = 'spring_app_sak-0.0.1-SNAPSHOT.jar'  // Correcting the JAR file name
        IMAGE_NAME = 'tohidspring'  // Docker image name
        DOCKER_HUB_REPO = 'tohidaws/tohidspring'  // Replace with your Docker Hub repo name
        GIT_CRED = 'git-cred'  // Git credentials ID
        DOCKER_HUB_CRED = 'docker-hub-cred'  // Docker Hub credentials ID in Jenkins
        JENKINS_URL = 'http://43.204.24.237:8080'  // Jenkins IP address
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                // Checkout code from GitHub
                git credentialsId: "${GIT_CRED}", url: 'https://github.com/tohidhanfi20/SpringBoot-APP.git', branch: 'main'
            }
        }

        stage('Maven Build and Test') {
            steps {
                script {
                    // Build and run tests using Maven
                    sh 'mvn clean validate compile test package verify install'
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                // Run Trivy scan on the file system
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the provided Dockerfile
                    sh '''
                    docker build -t ${IMAGE_NAME} -f Dockerfile .
                    '''
                }
            }
        }

        stage('Tag and Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub and push the image
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CRED}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker tag ${IMAGE_NAME} ${DOCKER_HUB_REPO}
                        docker push ${DOCKER_HUB_REPO}
                        '''
                    }
                }
            }
        }

        stage('Remove Local Docker Image') {
            steps {
                script {
                    // Remove the local Docker image
                    sh '''
                    docker rmi -f ${IMAGE_NAME} || true
                    '''
                }
            }
        }

        stage('Pull and Run Docker Image from Docker Hub') {
            steps {
                script {
                    // Pull and run the image from Docker Hub
                    sh '''
                    docker pull ${DOCKER_HUB_REPO}
                    docker run -d --name springapp -p 8081:8081 ${DOCKER_HUB_REPO}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Container is running from the pulled Docker Hub image.'
        }
    }
}
