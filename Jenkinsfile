pipeline {
    agent any
    
    environment {
        // PHP Application
        APP_DIR = '/var/www/html/automatic-test'
        APACHE_CONFIG = '/etc/apache2/sites-available/automatic-test.conf'
        
        // MySQL
        DB_NAME = 'leehoang'
        DB_USER = 'leehoang'
        DB_PASS = credentials('leehoang')
        DB_ROOT_PASS = credentials('leehoang')
        
        // Backup
        BACKUP_DIR = '/var/backups/automatic-test'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/LeeHoang123/automatic-test.git'
            }
        }
        
        stage('Install System Dependencies') {
            steps {
                sh '''
                    # Update package list
                    sudo apt-get update
                    
                    # Install PHP and required extensions
                    sudo apt-get install -y php8.2 \
                        php8.2-cli \
                        php8.2-common \
                        php8.2-mysql \
                        php8.2-zip \
                        php8.2-gd \
                        php8.2-mbstring \
                        php8.2-curl \
                        php8.2-xml \
                        php8.2-bcmath
                    
                    # Install Apache
                    sudo apt-get install -y apache2
                    
                    # Install MySQL
                    sudo apt-get install -y mysql-server
                    
                    # Install Composer
                    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
                    sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                    php -r "unlink('composer-setup.php');"
                '''
            }
        }
        
        stage('Configure MySQL') {
            steps {
                sh '''
                    # Backup existing database if exists
                    if mysql -u root -e "USE ${DB_NAME}"; then
                        mkdir -p ${BACKUP_DIR}
                        mysqldump -u root ${DB_NAME} > ${BACKUP_DIR}/backup-${BUILD_NUMBER}.sql
                    fi
                    
                    # Create/Reset database and user
                    mysql -u root <<EOF
                    DROP DATABASE IF EXISTS ${DB_NAME};
                    CREATE DATABASE ${DB_NAME};
                    DROP USER IF EXISTS '${DB_USER}'@'localhost';
                    CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';
                    GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';
                    FLUSH PRIVILEGES;
EOF
                    
                    # Import initial database schema
                    mysql -u ${DB_USER} -p${DB_PASS} ${DB_NAME} < database/schema.sql
                '''
            }
        }
        
        stage('Configure Apache') {
            steps {
                sh '''
                    # Create Apache virtual host configuration
                    sudo tee ${APACHE_CONFIG} <<EOF
<VirtualHost *:80>
    ServerName automatic-test.local
    DocumentRoot ${APP_DIR}/public
    
    <Directory ${APP_DIR}/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog \${APACHE_LOG_DIR}/automatic-test-error.log
    CustomLog \${APACHE_LOG_DIR}/automatic-test-access.log combined
</VirtualHost>
EOF
                    
                    # Enable site and required modules
                    sudo a2ensite automatic-test
                    sudo a2enmod rewrite
                    
                    # Create application directory
                    sudo mkdir -p ${APP_DIR}
                    sudo chown -R www-data:www-data ${APP_DIR}
                '''
            }
        }
        
        stage('Deploy Application') {
            steps {
                sh '''
                    # Copy application files
                    sudo cp -r . ${APP_DIR}
                    
                    # Install PHP dependencies
                    cd ${APP_DIR}
                    composer install --no-dev --optimize-autoloader
                    
                    # Set proper permissions
                    sudo chown -R www-data:www-data ${APP_DIR}
                    sudo chmod -R 755 ${APP_DIR}
                    
                    # Create .env file
                    sudo tee ${APP_DIR}/.env <<EOF
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=${DB_NAME}
DB_USERNAME=${DB_USER}
DB_PASSWORD=${DB_PASS}
EOF
                '''
            }
        }
        
        stage('Run Database Migrations') {
            steps {
                dir(APP_DIR) {
                    sh '''
                        # If using Laravel
                        php artisan migrate --force
                        
                        # If using custom migrations
                        # for sql_file in database/migrations/*.sql; do
                        #     mysql -u ${DB_USER} -p${DB_PASS} ${DB_NAME} < $sql_file
                        # done
                    '''
                }
            }
        }
        
        stage('Cache Configuration') {
            steps {
                dir(APP_DIR) {
                    sh '''
                        # If using Laravel
                        php artisan config:cache
                        php artisan route:cache
                        php artisan view:cache
                    '''
                }
            }
        }
        
        stage('Restart Services') {
            steps {
                sh '''
                    # Restart Apache
                    sudo systemctl restart apache2
                    
                    # Restart MySQL
                    sudo systemctl restart mysql
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                sh '''
                    # Check Apache status
                    systemctl is-active apache2
                    
                    # Check MySQL status
                    systemctl is-active mysql
                    
                    # Check PHP version and modules
                    php -v
                    php -m
                    
                    # Test database connection
                    php ${APP_DIR}/artisan db:monitor
                '''
            }
        }
        
    }
  }
}
