pipeline {
    agent any

    environment {
        // Jenkins credentials configuration
        DOCKER_HUB_CREDENTIALS = credentials('danielchen_dockerhub_cre') // Docker Hub credentials ID stored in Jenkins

        // Docker Hub Repository's name
        DOCKER_IMAGE = 'danielchen3/teedy2025_manual' // Your Docker Hub username and repository name

        // Use build number as tag
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/danielchen3/teedy2.git']] // Your GitHub repository
                )
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Building image') {
            steps {
                script {
                    // Assume Dockerfile is at the root
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        stage('Upload image') {
            steps {
                script {
                    // Sign in to Docker Hub and push the image
                    docker.withRegistry('https://registry.hub.docker.com', 'DOCKER_HUB_CREDENTIALS') {
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        // Optional: tag as latest
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Run containers') {
            steps {
                script {
                    // Stop and remove existing container if it exists
                    sh 'docker stop teedy-container-8081 || true'
                    sh 'docker rm teedy-container-8081 || true'

                    // Run the container
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('--name teedy-container-8081 -d -p 8081:8080')

                    // Optional: list all teedy containers
                    sh 'docker ps --filter "name=teedy-container"'
                }
            }
        }
    }
}
