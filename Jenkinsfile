pipeline {
    agent any
    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/muhammadhur2/docker-react-node-auth.git'
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker compose -f docker-compose.yml build'
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh 'docker compose -f docker-compose.yaml push'
                    }
                }
            }
        }
        stage('Deploy using Docker Compose') {
            steps {
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: "khabib",
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'docker-compose.yml',
                                    removePrefix: '',
                                    execCommand: '''
                                        cd /remote/deployment/directory
                                        docker compose down
                                        docker compose up -d
                                    '''
                                )
                            ]
                        )
                    ])
                }
            }
        }
        stage('Purge Docker Images') {
            steps {
                script {
                    def imageNames = sh(script: 'docker images -q', returnStdout: true).trim().split('\n')
                    for (image in imageNames) {
                        sh "docker rmi -f ${image}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Deployment succeeded.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
