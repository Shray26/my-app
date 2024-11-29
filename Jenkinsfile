pipeline {
    agent any

    tools {
        dockerTool 'Docker'
    }

    environment {
        DOCKER_IMAGE = 'shray26/simple_app'
        GIT_CREDENTIALS_ID = 'github-https-credentials'
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-token'
        SERVER_SSH_CREDENTIALS = 'server-ssh-credentials'
        SERVER_IP = '52.66.130.176'
    }

    stages {
        stage('Source Code Checkout') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Shray26/my-app.git',
                        credentialsId: env.GIT_CREDENTIALS_ID
                    ]]
                ]
            }
        }

        stage('Build') {
            steps {
                sh 'echo "No build needed for HTML/CSS."'
            }
        }

        stage('Push Artifacts to Git') {
            steps {
                script {
                    sh '''
                        git add .
                        git commit -m "Automated commit from Jenkins pipeline"
                        git push origin main
                    '''
                }
            }
        }

        stage('Docker Build Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image to Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CREDENTIALS_ID,
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent([env.SERVER_SSH_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.SERVER_IP} '
                        docker pull ${env.DOCKER_IMAGE} &&
                        docker stop getting-started-container || true &&
                        docker rm getting-started-container || true &&
                        docker run -d --name getting-started-container -p 8080:80 ${env.DOCKER_IMAGE}'
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution complete.'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
