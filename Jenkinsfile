// pipeline {
//        agent {
//         label "docker-agent"
//         }
    
    
//     environment {
//         DOCKER_IMAGE = 'far-devops-assignment'
//         DOCKER_TAG = 'latest'
//     }
    
//     stages {
//         stage('Clean Workspace') {
//             steps {
//                 cleanWs() // Ensures a clean workspace before starting 
//             }
//         }

//         stage('Checkout') {
//             steps {
//                 git branch: 'main',
//                     url: 'https://github.com/farhanhameed010/far-devops-assignment-6.git'
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 script {
//                     def scannerHome = tool 'SonarQube Scanner' // Make sure this matches your Jenkins configuration
//                     withSonarQubeEnv('SonarQube') {
//                         sh """
//                             ${scannerHome}/bin/sonar-scanner \
//                             -Dsonar.projectKey=far-devops-assignment \
//                             -Dsonar.projectName=far-devops-assignment \
//                             -Dsonar.sources=. \
//                             -Dsonar.javascript.node.maxspace=4096
//                         """
//                     }
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     sh """
//                         docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
//                     """
//                 }
//             }
//         }
//     }
    
//     post {
//         always {
//             echo 'Cleaning up workspace...'
//             cleanWs()
//         }
//         success {
//             echo 'Pipeline succeeded!'
//         }
//         failure {
//             echo 'Pipeline failed!'
//             script {
//                 echo 'Check the logs for details.'
//             }
//         }
//     }
// }


pipeline {
    agent {
        label "docker-agent"
    }
    
    environment {
        DOCKER_IMAGE = 'far-devops-assignment-6'
        DOCKER_TAG = 'latest'
        // Add Docker Hub credentials - replace YOUR_DOCKER_HUB_USERNAME with your username
        DOCKER_HUB_CREDS = credentials('docker-hub-credentials')
        DOCKER_HUB_USERNAME = 'farhanhameed010'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
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
                    def scannerHome = tool 'SonarQube Scanner'
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
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Tag the image with Docker Hub username
                    sh """
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                    
                    // Login to Docker Hub
                    sh """
                        echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin
                    """
                    
                    // Push the image
                    sh """
                        docker push ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
            // Logout from Docker Hub
            sh 'docker logout'
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