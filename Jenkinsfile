pipeline {
    agent any
    
    tools {
        maven 'maven'
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
        stage('Maven Clean') {
            steps {
                script {
                    sh 'mvn clean'  // This will clean your workspace
                }
            }
        }
        stage('Maven Validate') {
            steps {
                script {
                    sh 'mvn validate'  // This will validate your code
                }
            }
        }
        stage('Maven Compile') {
            steps {
                script {
                    sh 'mvn compile'  // This will compile your code
                }
            }
        }
        stage('Maven Test') {
            steps {
                script {
                    sh 'mvn test'  // This will test your code
                }
            }
        }
        stage('Maven Package') {
            steps {
                script {
                    sh 'mvn package'  // This will make a package
                }
            }
        }
        stage('Maven Verify') {
            steps {
                script {
                    sh 'mvn verify'  // This will verify your code
                }
            }
        }
        stage('Maven Install') {
            steps {
                script {
                    sh 'mvn install'  // This will install and give you a JAR file as output
                }
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=SpringApp \
                    -Dsonar.projectKey=SpringApp \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SONARQUBE_CREDENTIALS'
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt -o table"  // Run Trivy scan on file system
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t ${IMAGE_NAME} -f Dockerfile .  // Build the Docker image
                    '''
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ${IMAGE_NAME} > trivy_image.txt -o html"  // Run Trivy scan on Docker image
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d --name ${MYSQL_CONTAINER} -e MYSQL_ROOT_PASSWORD=1234 -p 3306:3306 mysql:latest'  // Start MySQL container
                    sh 'docker run -d --name springapp -p 8081:8081 ${IMAGE_NAME}'  // Run Spring Boot container
                }
            }
        }
        stage('SonarQube Webhook') {
            steps {
                script {
                    // Send a webhook to Jenkins from SonarQube
                    sh """
                    curl -X POST -d "payload={\"build_status\":\"success\",\"project_name\":\"SpringApp\"}" ${SONAR_WEBHOOK_URL}
                    """
                }
            }
        }
    }
}
