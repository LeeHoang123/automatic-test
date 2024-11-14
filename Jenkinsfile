pipeline {
    agent any
    environment {
        APP_NAME = "Demo-PHP"
        RELEASE = "1.0.0"
        DOCKER_USER = "hoangb2013534"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME_PHP = "${DOCKER_USER}/${APP_NAME}-php"
        IMAGE_NAME_MYSQL = "${DOCKER_USER}/${APP_NAME}-mysql"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        MYSQL_ROOT_PASSWORD = 'leehoang'  // Mật khẩu của root
        MYSQL_USER = 'leehoang'           // Tên người dùng
        MYSQL_PASSWORD = 'leehoang'       // Mật khẩu của người dùng
        MYSQL_DATABASE = 'User'           // Cơ sở dữ liệu
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/LeeHoang123/automatic-test.git'
            }
        }
        stage('Build PHP Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Build image PHP từ Dockerfile và gắn tag
                        sh """
                            docker build -t ${IMAGE_NAME_PHP}:${IMAGE_TAG} -f Dockerfile-php .
                            docker tag ${IMAGE_NAME_PHP}:${IMAGE_TAG} ${IMAGE_NAME_PHP}:latest
                        """
                    }
                }
            }
        }
        stage('Build MySQL Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Build image MySQL với các biến môi trường
                        sh """
                            docker build -t ${IMAGE_NAME_MYSQL}:${IMAGE_TAG} --build-arg MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} --build-arg MYSQL_USER=${MYSQL_USER} --build-arg MYSQL_PASSWORD=${MYSQL_PASSWORD} --build-arg MYSQL_DATABASE=${MYSQL_DATABASE} -f Dockerfile-mysql .
                            docker tag ${IMAGE_NAME_MYSQL}:${IMAGE_TAG} ${IMAGE_NAME_MYSQL}:latest
                        """
                    }
                }
            }
        }
        stage('Push PHP Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Push image PHP lên Docker Hub
                        docker.image("${IMAGE_NAME_PHP}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME_PHP}:latest").push()
                    }
                }
            }
        }
        stage('Push MySQL Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Push image MySQL lên Docker Hub
                        docker.image("${IMAGE_NAME_MYSQL}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME_MYSQL}:latest").push()
                    }
                }
            }
        }
        stage('Cleanup Artifacts') {
            steps {
                script {
                    // Xóa images đã tạo sau khi push lên Docker Hub
                    sh "docker rmi ${IMAGE_NAME_PHP}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_PHP}:latest"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:latest"
                }
            }
        }
    }
}
