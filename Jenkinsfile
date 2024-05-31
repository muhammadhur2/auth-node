pipeline {
    agent any
    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Checking out source code from SCM...'
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/muhammadhur2/docker-react-node-auth.git'
            }
        }
        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                script {
                    sh 'docker compose -f docker-compose.yaml build'
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker images to DockerHub...'
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                        sh 'docker push muhammadhur/test-khabib-server:latest'
                        sh 'docker push muhammadhur/test-khabib-client:latest'
                    }
                }
            }
        }
        stage('Deploy using Docker Compose') {
            steps {
                echo 'Deploying using Docker Compose...'
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: "khabib",
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'docker-compose.yaml',
                                    removePrefix: '',
                                    execCommand: '''
                                        set -x # Enable shell command echoing
                                        cd /root/jenkinstest
                                        docker compose pull
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
                echo 'Purging Docker images...'
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
