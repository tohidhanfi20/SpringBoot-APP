pipeline {
    agent any

    tools {
        maven 'maven'  // Ensure Maven is configured in Jenkins
    }

    environment {
        JAR_NAME = 'spring_app_sak-0.0.1-SNAPSHOT.jar'  // Correcting the JAR file name
        IMAGE_NAME = 'sakgroup'  // Docker image name
        MYSQL_CONTAINER = 'mysql-container'  // Name of the MySQL container
        SPRING_APP_CONTAINER = 'springapp'  // Name of the Spring Boot container
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
                    // Ensure MySQL container is running (no need to stop or remove it)
                    sh """
                    docker ps -a -q -f name=${MYSQL_CONTAINER} | xargs -r docker start
                    """

                    // Run Spring Boot container (if not already running)
                    sh '''
                    docker ps -a -q -f name=${SPRING_APP_CONTAINER} | xargs -r docker start
                    docker run -d --name ${SPRING_APP_CONTAINER} -p 8081:8080 ${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('Post-Run Cleanup') {
            steps {
                script {
                    // Optionally, remove any unused Docker images to free up space
                    sh 'docker system prune -f'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up resources after the pipeline execution...'
            // No MySQL container stop here, it will keep running.
            // Spring Boot container is not touched after it starts running.
        }
    }
}
