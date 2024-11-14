pipeline {
    agent any
    environment {
        APP_NAME = "demo-php"
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
                        // Tạo container PHP từ image chính thức
                        def container_php = docker.image("php:7.4-apache")
                        
                        // Pull image PHP chính thức trước khi tạo container
                        container_php.pull()

                        // Chạy container PHP với quyền root để sao chép các file cần thiết
                        container_php.inside('--user root') {
                            sh "cp -r ./public /var/www/html"
                            sh "cp -r ./database /var/www/database"
                        }

                        // Sử dụng docker build để tạo image PHP mới từ container
                        sh """
                            docker build -t ${IMAGE_NAME_PHP}:${IMAGE_TAG} -f- . <<EOF
                            FROM ${container_php.id}
                            COPY ./public /var/www/html
                            COPY ./database /var/www/database
                            EOF
                        """
                        sh "docker tag ${IMAGE_NAME_PHP}:${IMAGE_TAG} ${IMAGE_NAME_PHP}:latest"
                    }
                }
            }
        }
        stage('Build and Tag MySQL Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Tạo container MySQL từ image chính thức và cấu hình các biến môi trường
                        def container_mysql = docker.image('mysql:latest')
                        
                        // Pull image MySQL chính thức trước khi tạo container
                        container_mysql.pull()

                        // Tạo container MySQL với các biến môi trường
                        container_mysql.inside("-e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} -e MYSQL_USER=${MYSQL_USER} -e MYSQL_PASSWORD=${MYSQL_PASSWORD} -e MYSQL_DATABASE=${MYSQL_DATABASE}") {
                            // Không cần làm gì thêm trong container MySQL
                        }

                        // Sử dụng docker build để tạo image MySQL mới từ container
                        sh """
                            docker build -t ${IMAGE_NAME_MYSQL}:${IMAGE_TAG} -f- . <<EOF
                            FROM ${container_mysql.id}
                            EOF
                        """
                        sh "docker tag ${IMAGE_NAME_MYSQL}:${IMAGE_TAG} ${IMAGE_NAME_MYSQL}:latest"
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
