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
                    // Tagging images correctly as per DockerHub username/repository
                    sh '''
                    docker compose -f docker-compose.yaml build
                    docker tag yourservice_server:latest $DOCKERHUB_USER/test-khabib-server:latest
                    docker tag yourservice_client:latest $DOCKERHUB_USER/test-khabib-client:latest
                    '''
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker images to DockerHub...'
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        // Logging into DockerHub before pushing
                        sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker compose -f docker-compose.yaml push
                        '''
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
                                    remoteDirectory: '/root/jenkinstest',
                                    execCommand: '''
                                        set -x # Enable shell command echoing
                                        cd /root/jenkinstest
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
