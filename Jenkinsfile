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
        SONARQUBE_SERVER = 'sonar-server'  // SonarQube server environment
        SONARQUBE_TOKEN = credentials('sonar-token')  // SonarQube token from Jenkins credentials
        JENKINS_URL = 'http://43.204.24.237:8080'  // Jenkins IP address
        SONAR_WEBHOOK_URL = 'http://43.204.24.237:8080/sonarqube-webhook'  // Jenkins webhook URL
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
        stage("SonarQube Analysis with Docker") {
            steps {
                script {
                    // Use Docker container to run SonarScanner
                    sh '''
                    docker run --rm -e SONARQUBE_HOST_URL=${SONARQUBE_SERVER} -e SONARQUBE_TOKEN=${SONARQUBE_TOKEN} \
                    -v $(pwd):/usr/src \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectName=SpringApp -Dsonar.projectKey=SpringApp -Dsonar.login=${SONARQUBE_TOKEN}
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    // Wait for quality gate result
                    waitForQualityGate abortPipeline: false, credentialsId: 'SONARQUBE_CREDENTIALS'
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                // Run Trivy scan on file system
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
        stage('SonarQube Webhook') {
            steps {
                script {
                    // Send a webhook to Jenkins from SonarQube after analysis
                    sh """
                    curl -X POST -d "payload={\"build_status\":\"success\",\"project_name\":\"SpringApp\"}" ${SONAR_WEBHOOK_URL}
                    """
                }
            }
        }
    }
}
