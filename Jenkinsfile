// pipeline {
//     agent {
//         label "docker-agent"
//     }
    
//     environment {
//         DOCKER_IMAGE = 'far-devops-assignment-6'
//         DOCKER_TAG = 'latest'
//         // Add Docker Hub credentials - replace YOUR_DOCKER_HUB_USERNAME with your username
//         DOCKER_HUB_CREDS = credentials('docker-hub-credentials')
//         DOCKER_HUB_USERNAME = 'farhanhameed010'
//     }
    
//     stages {
//         stage('Clean Workspace') {
//             steps {
//                 cleanWs()
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
//                     def scannerHome = tool 'SonarQube Scanner'
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
        
//         stage('Push to Docker Hub') {
//             steps {
//                 script {
//                     // Tag the image with Docker Hub username
//                     sh """
//                         docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
//                     """
                    
//                     // Login to Docker Hub
//                     sh """
//                         echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin
//                     """
                    
//                     // Push the image
//                     sh """
//                         docker push ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
//                     """
//                 }
//             }
//         }
//     }
    
//     post {
//         always {
//             echo 'Cleaning up workspace...'
//             cleanWs()
//             // Logout from Docker Hub
//             sh 'docker logout'
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
        DOCKER_HUB_CREDS = credentials('docker-hub-credentials')
        DOCKER_HUB_USERNAME = 'farhanhameed010'
        // EC2 connection details
        EC2_HOST = '54.173.51.159'
        EC2_USER = 'ubuntu'
        EC2_KEY = credentials('ec2-ssh-key')
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
                            ${scannerHome}/bin/sonar-scanner \\
                            -Dsonar.projectKey=far-devops-assignment \\
                            -Dsonar.projectName=far-devops-assignment \\
                            -Dsonar.sources=. \\
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
                    sh """
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                    
                    sh """
                        echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin
                    """
                    
                    sh """
                        docker push ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    // Copy docker-compose file to EC2
                    sh """
                        scp -o StrictHostKeyChecking=no -i ${EC2_KEY} docker-compose-app.yml ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/
                    """
                    
                    // Execute deployment commands on EC2
                    sh """
                        ssh -o StrictHostKeyChecking=no -i ${EC2_KEY} ${EC2_USER}@${EC2_HOST} '
                            echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin
                            docker pull ${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker-compose -f docker-compose-app.yml down
                            docker-compose -f docker-compose-app.yml up -d
                        '
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
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