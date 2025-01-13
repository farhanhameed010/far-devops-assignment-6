pipeline {
       agent {
        label "docker-agent"
        }
    
    
    environment {
        DOCKER_IMAGE = 'far-devops-assignment'
        DOCKER_TAG = 'latest'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs() // Ensures a clean workspace before starting
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/farhanhameed010/far-devops-assignment-6.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube Scanner' // Make sure this matches your Jenkins configuration
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=far-devops-assignment \
                            -Dsonar.projectName=far-devops-assignment \
                            -Dsonar.sources=. \
                            -Dsonar.javascript.node.maxspace=4096
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            script {
                echo 'Check the logs for details.'
            }
        }
    }
}
