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
        // JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
        MYSQL_ROOT_PASSWORD = 'leehoang'  // Mật khẩu của root
        MYSQL_USER = 'leehoang'                // Tên người dùng
        MYSQL_PASSWORD = 'leehoang'    // Mật khẩu của người dùng
        MYSQL_DATABASE = 'User'              // Cơ sở dữ liệu
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
                    // Xây dựng Docker image cho ứng dụng PHP từ thư mục chứa Dockerfile
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image_php = docker.build("${IMAGE_NAME_PHP}", '.')
                    }
                }
            }
        }
        stage('Build MySQL Docker Image') {
            steps {
                script {
                    // Sử dụng image chính thức MySQL và cấu hình các biến môi trường
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image_mysql = docker.build("${IMAGE_NAME_MYSQL}", {
                            dockerfile: './mysql/Dockerfile',  // Chỉ định đường dẫn nếu bạn có Dockerfile riêng
                            args: [
                                "MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}",
                                "MYSQL_USER=${MYSQL_USER}",
                                "MYSQL_PASSWORD=${MYSQL_PASSWORD}",
                                "MYSQL_DATABASE=${MYSQL_DATABASE}"
                            ]
                        })
                    }
                }
            }
        }
        stage('Push PHP Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image_php.push("${IMAGE_TAG}")
                        docker_image_php.push('latest')
                    }
                }
            }
        }
        stage('Push MySQL Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image_mysql.push("${IMAGE_TAG}")
                        docker_image_mysql.push('latest')
                    }
                }
            }
        }
        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME_PHP}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_PHP}:latest"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:latest"
                }
            }
        }
    }
}
