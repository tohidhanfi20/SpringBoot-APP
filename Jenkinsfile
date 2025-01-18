pipeline {
    agent any

    tools {
        maven 'maven'  // Ensure Maven is configured in Jenkins
    }

    environment {
        JAR_NAME = 'spring_app_sak-0.0.1-SNAPSHOT.jar'  // Correcting the JAR file name
        IMAGE_NAME = 'sakgroup'  // Docker image name
        MYSQL_CONTAINER = 'mysql-container'  // Name of the MySQL container
        GIT_CRED = 'git-cred'  // Git credentials ID
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
                sh "trivy fs . > trivyfs.txt -o table"
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

        stage('Trivy Image Scan') {
            steps {
                // Run Trivy scan on Docker image
                sh "trivy image ${IMAGE_NAME} > trivy_image.txt -o html"
            }
        }

        stage('Stop and Remove Existing Containers') {
            steps {
                script {
                    // Stop and remove existing MySQL and Spring app containers if they exist
                    sh 'docker ps -aq --filter name=${MYSQL_CONTAINER} | xargs -I {} docker stop {} || true'
                    sh 'docker ps -aq --filter name=${MYSQL_CONTAINER} | xargs -I {} docker rm {} || true'
                    sh 'docker ps -aq --filter name=springapp | xargs -I {} docker stop {} || true'
                    sh 'docker ps -aq --filter name=springapp | xargs -I {} docker rm {} || true'
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    // Run MySQL container
                    sh 'docker run -d --name ${MYSQL_CONTAINER} -e MYSQL_ROOT_PASSWORD=1234 -p 3306:3306 mysql:latest'

                    // Run Spring Boot container
                    sh 'docker run -d --name springapp -p 8081:8081 ${IMAGE_NAME}'
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up containers after pipeline execution
                sh 'docker ps -aq --filter name=${MYSQL_CONTAINER} | xargs -I {} docker stop {} || true'
                sh 'docker ps -aq --filter name=${MYSQL_CONTAINER} | xargs -I {} docker rm {} || true'
                sh 'docker ps -aq --filter name=springapp | xargs -I {} docker stop {} || true'
                sh 'docker ps -aq --filter name=springapp | xargs -I {} docker rm {} || true'
            }
        }
    }
}
