def imageName="192.168.56.44:8082/docker_registry/frontend"
def dockerRegistry="http://192.168.56.44:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent {
  label 'agent'
}

environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
    stage('Get Code') {
            steps {
                // Get some code from a GitHub repository
                checkout scm
            }
        }
    
       stage('install requirements.txt') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }


        stage('Run tests') {
            steps {
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
    
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }   
    
    stage('Build application image') {
            steps {
                script {
                  // Prepare basic image for application
                dockerTag = "RC-${env.BUILD_ID}"
                applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
    
    }
    stage ('Pushing image to Artifactory') {
            steps {
                script {
                docker.withRegistry("$dockerRegistry", "$registryCredentials") {
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