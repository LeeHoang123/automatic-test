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
        // stage('Build and Run PHP Docker Container') {
        //     steps {
        //         script {
        //             // Tạo container PHP từ image chính thức và sao chép các file vào
        //             docker.withRegistry('', DOCKER_PASS) {
        //                 // Tạo container PHP từ image chính thức PHP
        //                 def container_php = docker.image("php:7.4-apache").run('-d')

        //                 // Sao chép các file từ repo vào container
        //                 sh "docker cp ./public ${container_php.id}:/var/www/html"
        //                 sh "docker cp ./database ${container_php.id}:/var/www/database"

        //                 // Commit lại container thành image PHP của bạn
        //                 docker.image(container_php.id).commit("${IMAGE_NAME_PHP}:${IMAGE_TAG}")
        //             }
        //         }
        //     }
        // }
        stage('Build and Run PHP Docker Container') {
            steps {
                script {
                    // Đăng nhập vào Docker Registry nếu cần
                    docker.withRegistry('', DOCKER_PASS) {
                        // Tạo container PHP từ image chính thức PHP
                        def container_php = docker.image("php:7.4-apache").run('-d')
        
                        // Sao chép các file từ repo vào container
                        sh "docker cp ./public ${container_php.id}:/var/www/html"
                        sh "docker cp ./database ${container_php.id}:/var/www/database"
        
                        // Commit lại container thành image PHP của bạn
                        def php_image = docker.image("${IMAGE_NAME_PHP}:${IMAGE_TAG}")
                        php_image.build("--no-cache") // Xây dựng lại image từ container đã chỉnh sửa
                        php_image.push() // Đẩy image lên Docker registry nếu cần
                    }
                }
            }
}

        stage('Build and Run MySQL Docker Container') {
            steps {
                script {
                    // Tạo container MySQL từ image chính thức và cấu hình
                    def container_mysql = docker.image('mysql:latest').run("-d -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} -e MYSQL_USER=${MYSQL_USER} -e MYSQL_PASSWORD=${MYSQL_PASSWORD} -e MYSQL_DATABASE=${MYSQL_DATABASE}")
                    
                    // Commit lại container MySQL thành image của bạn
                    docker.image(container_mysql.id).commit("${IMAGE_NAME_MYSQL}:${IMAGE_TAG}")
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
                    sh "docker rmi ${IMAGE_NAME_PHP}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_PHP}:latest"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME_MYSQL}:latest"
                }
            }
        }
    }
}
