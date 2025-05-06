pipeline {
    agent any

    environment {
        // Docker Hub repository
        DOCKER_IMAGE = 'danielchen3/teedy2025_manual'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/danielchen3/teedy2.git']]
                )
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Upload Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'danielchen_dockerhub_cre', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'danielchen_dockerhub_cre') {
                            docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                            docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                        }
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh 'docker stop teedy-container-8081 || true'
                    sh 'docker rm teedy-container-8081 || true'
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").run('--name teedy-container-8081 -d -p 8081:8080')
                    sh 'docker ps --filter "name=teedy-container"'
                }
            }
        }
    }
}
