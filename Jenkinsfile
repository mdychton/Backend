def imageName="mddocker1990/backend"
def dockerRegistry=""
def registryCredentials="mddocker1990"
def dockerTag=""


pipeline {
    agent {
        label 'agent'
    }
    
    environment {
  scannerHome = tool 'SonarQube'
    }

    stages {
        stage ('Get CodeBackend' ) {
            steps {
            //git branch: 'main', url: 'https://github.com/mdychton/Backend.git'
             checkout scm
            }
        }
        
        stage ('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
         }
         stage ('Sonarqube analysis') {
            steps {
                withSonarQubeEnv ('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(1) {
               waitForQualityGate abortPipeline: true
            }
        }
        }
        stage ('Build application image'){
            steps {
                script {
                    dockerTag = "${env.BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        stage ('Pushing image to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry","$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}