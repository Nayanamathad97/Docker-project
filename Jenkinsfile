pipeline {
    agent any

    environment {
        ImageRegistry = 'nayanapreetham'
        DOCKER_EC2 = 'ubuntu@43.205.95.246'
        DockerImageTag = "${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('nayanapreetham')
    }

    stages {

        stage("Checkout") {
            steps {
                git 'https://github.com/Nayanamathad97/Docker-project.git'
            }
        }

        stage("Build on Docker EC2") {
            steps {
                script {
                    sshagent(['ec2']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_EC2} '
                            cd /home/ubuntu &&
                            docker build -t ${DockerImageTag} .
                        '
                        """
                    }
                }
            }
        }

        stage("Login & Push on Docker EC2") {
            steps {
                script {
                    sshagent(['ec2']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_EC2} '
                            echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin &&
                            docker push ${DockerImageTag}
                        '
                        """
                    }
                }
            }
        }

        stage("Deploy on Docker EC2") {
            steps {
                script {
                    sshagent(['ec2']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_EC2} '
                            docker stop app || true &&
                            docker rm app || true &&
                            docker run -d --name app -p 80:80 ${DockerImageTag}
                        '
                        """
                    }
                }
            }
        }
    }
}