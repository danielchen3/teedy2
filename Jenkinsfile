pipeline {
    agent any

    environment {
        // Docker Hub 镜像名（用户名/仓库）
        DOCKER_IMAGE = 'danielchen3/teedy2025_manual'

        // 使用 Jenkins 构建号作为 tag
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/danielchen3/teedy2.git']]
                )
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
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

        stage('Run 3 Containers') {
            steps {
                script {
                    def ports = [8082, 8083, 8084]
                    for (int i = 0; i < ports.size(); i++) {
                        def containerName = "teedy-container-${ports[i]}"
                        sh "docker stop ${containerName} || true"
                        sh "docker rm ${containerName} || true"
                        sh "docker run --name ${containerName} -d -p ${ports[i]}:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }

                    // Optional: show running containers
                    sh 'docker ps --filter "name=teedy-container-"'
                }
            }
        }
    }
}
