pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        JAR_NAME = 'spring_app_sak-0.0.1-SNAPSHOT.jar'  // Correct JAR file name
        IMAGE_NAME = 'sakgroup'  // Docker image name
        MYSQL_CONTAINER = 'mysql-container'  // Name of the MySQL container
        GIT_CRED = 'git-cred'  // Git credentials ID
        SONARQUBE_SERVER = 'sonar-server'  // SonarQube server environment
        SONARQUBE_TOKEN = credentials('sonar-token')  // SonarQube token from Jenkins credentials
        SONARQUBE_HOST_URL = 'http://43.204.24.237:9000'  // SonarQube server URL (adjust with your IP or domain)
        JENKINS_URL = 'http://43.204.24.237:8080'  // Jenkins URL
        SONAR_WEBHOOK_URL = 'http://43.204.24.237:8080/sonarqube-webhook'  // Jenkins webhook URL for SonarQube
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
                    sh 'mvn clean'  // Clean the workspace
                }
            }
        }

        stage('Maven Validate') {
            steps {
                script {
                    sh 'mvn validate'  // Validate the code
                }
            }
        }

        stage('Maven Compile') {
            steps {
                script {
                    sh 'mvn compile'  // Compile the code
                }
            }
        }

        stage('Maven Test') {
            steps {
                script {
                    sh 'mvn test'  // Run tests
                }
            }
        }

        stage('Maven Package') {
            steps {
                script {
                    sh 'mvn package'  // Create the package
                }
            }
        }

        stage('Maven Verify') {
            steps {
                script {
                    sh 'mvn verify'  // Verify the code
                }
            }
        }

        stage('Maven Install') {
            steps {
                script {
                    sh 'mvn install'  // Install dependencies and generate the JAR file
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh ''' 
                    docker run --rm -e SONARQUBE_HOST_URL=${SONARQUBE_HOST_URL} -e SONARQUBE_TOKEN=${SONARQUBE_TOKEN} \
                    -v /var/lib/jenkins/workspace/SpringBoot-APP:/usr/src sonarsource/sonar-scanner-cli \
                    -Dsonar.projectName=SpringApp -Dsonar.projectKey=SpringApp -Dsonar.login=${SONARQUBE_TOKEN}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SONARQUBE_CREDENTIALS'  // Wait for SonarQube quality gate
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt -o table"  // Run Trivy scan on the filesystem
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
                sh "trivy image ${IMAGE_NAME} > trivy_image.txt -o html"  // Run Trivy scan on the Docker image
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    sh 'docker run -d --name ${MYSQL_CONTAINER} -e MYSQL_ROOT_PASSWORD=1234 -p 3306:3306 mysql:latest'  // Start MySQL container
                    sh 'docker run -d --name springapp -p 8081:8081 ${IMAGE_NAME}'  // Start Spring Boot container
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
