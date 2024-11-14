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
        PMA_PORT = "82"                   // Thay đổi cổng phpMyAdmin thành 82
        PMA_IMAGE = "phpmyadmin/phpmyadmin" // Image phpMyAdmin chính thức
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Stop and Remove Old Containers') {
            steps {
                script {
                    // Dừng tất cả các container đang chạy và xóa chúng
                    sh 'docker ps -q | xargs -r docker stop'   // Dừng tất cả các container đang chạy
                    sh 'docker ps -aq | xargs -r docker rm'     // Xóa tất cả các container
                }
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

                        // Tạo Dockerfile tạm thời trong thư mục hiện tại
                        writeFile(file: 'Dockerfile.temp', text: """
                            FROM ${container_php.id}
                            COPY ./public /var/www/html
                            COPY ./database /var/www/database
                        """)

                        // Sử dụng docker build với Dockerfile tạm thời
                        sh """
                            docker build -t ${IMAGE_NAME_PHP}:${IMAGE_TAG} -f Dockerfile.temp . 
                        """
                        
                        // Xóa Dockerfile tạm thời sau khi build xong
                        sh "rm Dockerfile.temp"

                        // Đánh tag image PHP
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

                        // Tạo Dockerfile tạm thời cho MySQL
                        writeFile(file: 'Dockerfile.temp', text: """
                            FROM ${container_mysql.id}
                        """)

                        // Sử dụng docker build với Dockerfile tạm thời cho MySQL
                        sh """
                            docker build -t ${IMAGE_NAME_MYSQL}:${IMAGE_TAG} -f Dockerfile.temp . 
                        """
                        
                        // Xóa Dockerfile tạm thời sau khi build xong
                        sh "rm Dockerfile.temp"

                        // Đánh tag image MySQL
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
        stage('Deploy Containers') {
            steps {
                script {
                    // Tạo Docker network cho các container giao tiếp với nhau
                    sh 'docker network create --driver bridge my-network || true'

                    // Pull và chạy MySQL container
                    sh """
                        docker pull ${IMAGE_NAME_MYSQL}:latest
                        docker run -d \
                            --name mysql-container \
                            --network my-network \
                            -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                            -e MYSQL_USER=${MYSQL_USER} \
                            -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
                            -e MYSQL_DATABASE=${MYSQL_DATABASE} \
                            -p 3306:3306 \
                            ${IMAGE_NAME_MYSQL}:latest
                    """

                    // Chờ 30 giây để MySQL container khởi động hoàn toàn
                    sh 'sleep 30'

                    // Pull và chạy PHP container
                    sh """
                        docker pull ${IMAGE_NAME_PHP}:latest
                        docker run -d \
                            --name php-container \
                            --network my-network \
                            -p 9001:80 \
                            ${IMAGE_NAME_PHP}:latest
                    """
                    
                    // Pull và chạy phpMyAdmin container trên cổng 82
                    sh """
                        docker pull ${PMA_IMAGE}
                        docker run -d \
                            --name phpmyadmin-container \
                            --network my-network \
                            -e PMA_HOST=mysql-container \
                            -e PMA_PORT=3306 \
                            -p ${PMA_PORT}:80 \
                            ${PMA_IMAGE}
                    """

                    // Sao chép file Person.sql từ thư mục project vào trong container MySQL
                    sh """
                        docker cp ./database/Person.sql mysql-container:/tmp/Person.sql
                    """
                    
                    // Import dữ liệu từ file Person.sql vào MySQL
                    sh """
                        docker exec -i mysql-container mysql -u ${MYSQL_USER} -p${MYSQL_PASSWORD} ${MYSQL_DATABASE} < /tmp/Person.sql
                    """
                }
            }
        }
    }
}
