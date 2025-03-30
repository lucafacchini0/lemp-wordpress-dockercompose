# Docker Compose Configuration Documentation

This document provides a detailed explanation of the `docker-compose.yaml` file used in this WordPress setup with Nginx, MySQL, and phpMyAdmin.

## Overview

The `docker-compose.yaml` file defines the services, networks, and volumes that make up our WordPress application stack. It orchestrates how these components interact with each other.

## File Structure

The file consists of three main sections:
1. **Services**: Defines the containers that will be created
2. **Networks**: Defines the networks that connect the containers
3. **Volumes**: Defines persistent storage for data

## Services

### Database Service

```yaml
database:
  image: mysql:8.0
  container_name: database
  networks:
    - wp-db-network
  volumes:
    - mysql_data:/var/lib/mysql
  env_file: .env
  environment:
    - MYSQL_ROOT_PASSWORD=${SQL_ROOT_PASSWORD}
    - MYSQL_DATABASE=${SQL_DATABASE_NAME}
    - MYSQL_USER=${SQL_DATABASE_USER}
    - MYSQL_PASSWORD=${SQL_DATABASE_PASSWORD}
```

- **image: mysql:8.0**: Uses the official MySQL 8.0 image from Docker Hub
- **container_name: database**: Names the container "database" for easy reference
- **networks: - wp-db-network**: Connects this container to the wp-db-network (used for database connections)
- **volumes: - mysql_data:/var/lib/mysql**: Mounts the mysql_data volume to /var/lib/mysql inside the container, ensuring data persistence
- **env_file: .env**: Loads environment variables from the .env file
- **environment**: Sets specific environment variables for MySQL:
  - **MYSQL_ROOT_PASSWORD**: Sets the root password from the .env file
  - **MYSQL_DATABASE**: Creates a database with the name from the .env file
  - **MYSQL_USER**: Creates a user with the name from the .env file
  - **MYSQL_PASSWORD**: Sets the password for the user from the .env file

### WordPress Service

```yaml
wordpress:
  depends_on:
    - database
  image: wordpress:6.7.2-php8.4-fpm
  container_name: wordpress
  networks:
    - wp-network
    - wp-db-network
  volumes:
    - ./wordpress:/var/www/html
  env_file: .env
  environment:
    - WORDPRESS_DB_HOST=database
    - WORDPRESS_DB_USER=${SQL_DATABASE_USER}
    - WORDPRESS_DB_PASSWORD=${SQL_DATABASE_PASSWORD}
    - WORDPRESS_DB_NAME=${SQL_DATABASE_NAME}
```

- **depends_on: - database**: Ensures the database container starts before the WordPress container
- **image: wordpress:6.7.2-php8.4-fpm**: Uses the official WordPress image with PHP-FPM 8.4 (version 6.7.2)
- **container_name: wordpress**: Names the container "wordpress" for easy reference
- **networks**: Connects to two networks:
  - **wp-network**: For communication with Nginx
  - **wp-db-network**: For communication with the database
- **volumes: - ./wordpress:/var/www/html**: Mounts the local ./wordpress directory to /var/www/html in the container, storing WordPress files locally
- **env_file: .env**: Loads environment variables from the .env file
- **environment**: Sets specific environment variables for WordPress:
  - **WORDPRESS_DB_HOST**: Points to the database container
  - **WORDPRESS_DB_USER**: Sets the database user from the .env file
  - **WORDPRESS_DB_PASSWORD**: Sets the database password from the .env file
  - **WORDPRESS_DB_NAME**: Sets the database name from the .env file

### Nginx Service

```yaml
nginx:
  image: nginx:1.15.12-alpine
  ports:
    - "80:80"
    - "443:443"
  container_name: nginx
  networks:
    - wp-network
  volumes:
    - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    - ./nginx/conf.d:/etc/nginx/conf.d
    - ./wordpress:/var/www/html
    - /etc/letsencrypt:/etc/letsencrypt
    - /etc/nginx/ssl:/etc/nginx/ssl #dhparam - optional
```

- **image: nginx:1.15.12-alpine**: Uses the official Nginx image with Alpine Linux (lightweight)
- **ports**: Maps container ports to host ports:
  - **80:80**: HTTP port
  - **443:443**: HTTPS port
- **container_name: nginx**: Names the container "nginx" for easy reference
- **networks: - wp-network**: Connects to the wp-network for communication with WordPress
- **volumes**: Mounts several directories:
  - **./nginx/nginx.conf:/etc/nginx/nginx.conf**: Main Nginx configuration file
  - **./nginx/conf.d:/etc/nginx/conf.d**: Directory for site-specific configurations
  - **./wordpress:/var/www/html**: WordPress files (same as in WordPress container)
  - **/etc/letsencrypt:/etc/letsencrypt**: Let's Encrypt SSL certificates from the host
  - **/etc/nginx/ssl:/etc/nginx/ssl**: SSL-related files, including the Diffie-Hellman parameters

### phpMyAdmin Service

```yaml
phpmyadmin:
  container_name: phpmyadmin
  image: phpmyadmin/phpmyadmin
  env_file: .env
  environment:
    - PMA_HOST=database
    - PMA_PORT=3306
    - MYSQL_ROOT_PASSWORD=${SQL_ROOT_PASSWORD}
  ports:
    - "127.0.0.1:8080:80" # only accessible from localhost
  networks:
    - wp-db-network
```

- **container_name: phpmyadmin**: Names the container "phpmyadmin" for easy reference
- **image: phpmyadmin/phpmyadmin**: Uses the official phpMyAdmin image
- **env_file: .env**: Loads environment variables from the .env file
- **environment**: Sets specific environment variables for phpMyAdmin:
  - **PMA_HOST**: Points to the database container
  - **PMA_PORT**: Specifies the MySQL port (3306 is default)
  - **MYSQL_ROOT_PASSWORD**: Sets the root password from the .env file
- **ports: - "127.0.0.1:8080:80"**: Maps container port 80 to host port 8080, but only on localhost (127.0.0.1), making it inaccessible from outside the server for security
- **networks: - wp-db-network**: Connects to the wp-db-network for database access

## Networks

```yaml
networks:
  wp-network:
    driver: bridge
  wp-db-network:
    driver: bridge
```

- **wp-network**: Network for communication between Nginx and WordPress
  - **driver: bridge**: Uses the standard bridge network driver
- **wp-db-network**: Network for database-related communications
  - **driver: bridge**: Uses the standard bridge network driver

Using separate networks enhances security by isolating different types of traffic. The database is not directly accessible from the web server, only through the WordPress container.

## Volumes

```yaml
volumes:
  mysql_data:
```

- **mysql_data**: A named volume for storing MySQL data
  - This ensures database data persists even if the container is removed or recreated
  - Docker manages this volume automatically

## Security Considerations

1. **Database Isolation**: The database is only accessible through the wp-db-network
2. **phpMyAdmin Restriction**: phpMyAdmin is only accessible from localhost (127.0.0.1)
3. **Environment Variables**: Sensitive data is stored in the .env file, not hardcoded
4. **SSL Support**: The setup includes proper SSL configuration for secure HTTPS connections

## Performance Considerations

1. **PHP-FPM**: Using WordPress with PHP-FPM provides better performance than using Apache with mod_php
2. **Alpine Linux**: The Nginx image uses Alpine Linux, which is lightweight and has a smaller attack surface
3. **Separate Services**: Each component runs in its own container, allowing for better resource allocation and scaling