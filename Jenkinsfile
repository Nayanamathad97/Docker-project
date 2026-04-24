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

        stage("Copy Code to Docker EC2") {
            steps {
                sh """
                scp -o StrictHostKeyChecking=no -r . ${DOCKER_EC2}:/home/ubuntu/app
                """
            }
        }

        stage("Build on Docker EC2") {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${DOCKER_EC2} '
                    cd /home/ubuntu/app &&
                    docker build -t ${DockerImageTag} .
                '
                """
            }
        }

        stage("Login & Push on Docker EC2") {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${DOCKER_EC2} '
                    echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin &&
                    docker push ${DockerImageTag}
                '
                """
            }
        }

        stage("Deploy on Docker EC2") {
            steps {
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