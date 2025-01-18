pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    environment {
        WAR_NAME = 'web-app.war'
        IMAGE_NAME = 'webapp' 
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                // Checkout code from GitHub
                git credentialsId: 'git-cred', url: 'https://github.com/tohidhanfi20/Portfolio-Webapp.git', branch: 'main'
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
                    sh 'mvn install'  // This will install and give you tar file as a output
                }
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=SpringApp \
                    -Dsonar.projectKey=SpringApp '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: SONARQUBE_CREDENTIALS
                }
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt -o table"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t ${IMAGE_NAME} -f Dockerfile .
                    '''
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image > trivy.txt -o html" 
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d --name container '
                }
            }
        }
    }
}
