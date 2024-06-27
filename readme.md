## Multi-Project PHP Development Environment
This container setup is designed to support multiple PHP projects within a single Nginx and PHP-FPM environment. Each project resides in its own directory within the /application directory.

### Prerequisites
Docker
Docker Compose
### Directory Structure
The directory structure should be organized as follows:
```text
/projetos/
    project1/
        public/
            index.php
    project2/
        public/
            index.php
/phpdocker/
    nginx/
        nginx.conf
    php-fpm/
        php-ini-overrides.ini

```
- /projetos/: This directory contains all your projects. Each project should have its own directory.
-  /phpdocker/: This directory contains the Docker configuration files.
-  nginx.conf: The Nginx configuration file.
-  php-ini-overrides.ini: PHP configuration overrides.

### Nginx Configuration
The Nginx configuration is set up to support multiple projects by using a single server block and rewriting URLs to the appropriate project directory.

nginx.conf:
```text
server {
    listen 80;

    client_max_body_size 108M;

    access_log /var/log/nginx/application.access.log;

    # Serve arquivos estáticos diretamente e fallback para index.php se não encontrar o arquivo
    location / {
        try_files $uri $uri/ @php;
    }

    location @php {
        rewrite ^/([^/]+)/?(.*)$ /$1/public/index.php?$2 last;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "error_log=/var/log/nginx/application_php_errors.log";
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        include fastcgi_params;
    }

    # Define o root para servir os arquivos estáticos e scripts PHP
    root /application;
    index index.php;
}
```

### Docker Compose Configuration
The Docker Compose configuration sets up the Nginx and PHP-FPM services, mounting the projects and configuration files into the containers.

docker-compose.yml:

```yaml
version: '3.1'

services:
  webserver:
    image: 'nginx:alpine'
    working_dir: /application
    volumes:
      - './projetos/:/application'
      - './phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf'
    ports:
      - '8005:80'

  php-fpm:
    build: phpdocker/php-fpm
    working_dir: /application
    volumes:
      - './projetos/:/application'
      - './phpdocker/php-fpm/php-ini-overrides.ini:/etc/php/8.1/fpm/conf.d/99-overrides.ini'
```

### How to Use
1. Clone the repository to your local machine.
```bash
git clone https://github.com/andersonaraf/multi-project-php.git
cd multi-project-php
```
2. Add your projects to the /projetos/ directory:

 Each project should have its own directory inside /projetos/ and should include a public directory containing the index.php file.
3. Build and run the Docker containers:
```bash
docker-compose up -d
```
4. Access your projects:
Open your browser and navigate to:
- http://localhost:8005/project1
- http://localhost:8005/project2
Replace project1 and project2 with the names of your actual project directories.

### Notes
Ensure that the project directories and files have the correct permissions to be read by the web server.
You can customize the PHP and Nginx configurations further by editing the files in the phpdocker directory.
### Troubleshooting
File Not Found: Ensure the directory structure and paths are correct.
Permission Issues: Ensure the correct file permissions for the www-data user inside the container.
### Conclusion
This setup provides a convenient way to manage and run multiple PHP projects in a single Docker environment, simplifying the development process.
