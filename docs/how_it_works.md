# How It Works: WordPress with Docker, Nginx, MySQL, and phpMyAdmin

This document explains how all components of the SecurePress system work together to deliver a secure, efficient WordPress website.

## System Architecture

The system consists of four main components, each running in its own Docker container:

1. **Nginx**: Web server that handles HTTP/HTTPS requests and serves static content
2. **WordPress (PHP-FPM)**: Application server that processes PHP code
3. **MySQL**: Database server that stores WordPress data
4. **phpMyAdmin**: Web-based database management tool

These components are connected through Docker networks and communicate with each other to serve the WordPress website.

## Component Interaction Flow

### 1. Client Request Flow

When a visitor accesses your WordPress site, here's what happens:

```
Client → HTTPS Request → Nginx → PHP-FPM (WordPress) → MySQL → Response back to Client
```

Let's break down each step:

#### Step 1: Client Makes a Request

A visitor types your domain (e.g., `https://example.com`) in their browser or clicks a link to your site.

#### Step 2: DNS Resolution

The domain name is resolved to your server's IP address through DNS.

#### Step 3: SSL Termination at Nginx

- The request reaches your server on port 443 (HTTPS)
- Nginx handles the SSL/TLS termination using Let's Encrypt certificates
- The encrypted connection is established between the client and Nginx

#### Step 4: Nginx Request Processing

- Nginx checks the request against its configuration rules
- For static files (JS, CSS, images), Nginx serves them directly without involving WordPress
- For PHP files or dynamic content, Nginx passes the request to the WordPress container

#### Step 5: WordPress (PHP-FPM) Processing

- The WordPress container receives the request via FastCGI protocol
- PHP-FPM processes the PHP code in WordPress
- WordPress may need to retrieve or store data in the database

#### Step 6: Database Interaction

- WordPress connects to the MySQL database container
- It retrieves or stores data as needed
- The database returns the requested data to WordPress

#### Step 7: Response Generation

- WordPress generates HTML content based on the request and database data
- The generated content is passed back to Nginx

#### Step 8: Response Delivery

- Nginx adds security headers to the response
- The response is sent back to the client over the secure HTTPS connection
- The client's browser renders the page

### 2. Network Isolation

The system uses two separate Docker networks for security:

```
wp-network: Nginx ↔ WordPress
wp-db-network: WordPress ↔ MySQL ↔ phpMyAdmin
```

This network separation ensures that:

- The database is not directly accessible from the web server
- External requests can only reach Nginx, not the internal services
- phpMyAdmin is only accessible through the database network

### 3. Data Persistence

The system ensures data persistence through Docker volumes and bind mounts:

#### Docker Named Volume

```
mysql_data: Stores MySQL database files
```

This volume persists even if the containers are removed or recreated, ensuring your database data is safe.

#### Bind Mounts

```
./wordpress:/var/www/html: WordPress files
./nginx/nginx.conf:/etc/nginx/nginx.conf: Nginx main configuration
./nginx/conf.d:/etc/nginx/conf.d: Nginx site configurations
/etc/letsencrypt:/etc/letsencrypt: SSL certificates
/etc/nginx/ssl:/etc/nginx/ssl: SSL-related files (dhparam)
```

These bind mounts map directories from the host to the containers, allowing for easy configuration and persistence.

## Security Features

### 1. SSL/TLS Encryption

All traffic is encrypted using Let's Encrypt certificates, with:

- Modern TLS protocols (TLSv1.2 and TLSv1.3)
- Strong cipher suites
- Perfect Forward Secrecy via ECDHE and DHE
- HSTS (HTTP Strict Transport Security)

### 2. Network Isolation

The two-network architecture prevents direct access to the database from the internet.

### 3. Security Headers

Nginx adds numerous security headers to responses, protecting against common web vulnerabilities:

- XSS (Cross-Site Scripting) protection
- Clickjacking protection
- MIME type sniffing protection
- Content Security Policy
- Cross-Origin Resource Sharing restrictions

### 4. Rate Limiting

Sensitive endpoints like wp-login.php and xmlrpc.php are protected by rate limiting to prevent brute force attacks.

### 5. IP Address Blocking

Direct access to the server's IP address (bypassing the domain name) is blocked with a 444 response code.

### 6. phpMyAdmin Restriction

phpMyAdmin is only accessible from localhost (127.0.0.1), preventing remote access.

## Performance Optimizations

### 1. PHP-FPM

Using PHP-FPM instead of mod_php provides better performance by:

- Separating PHP processing from the web server
- Better memory management
- Process pooling for handling multiple requests

### 2. Nginx Optimizations

- Static file caching with long expiry times
- Gzip compression for text-based content
- HTTP/2 support for faster loading
- Buffer size optimizations
- Connection keepalive

### 3. Container Isolation

Running each component in its own container allows for:

- Independent scaling of components
- Better resource allocation
- Easier updates and maintenance

## Maintenance Procedures

### 1. Updating Components

To update any component, you can change the image version in the docker-compose.yaml file and restart the containers:

```bash
docker-compose down
docker-compose up -d
```

### 2. Backing Up Data

#### Database Backup

```bash
docker exec database mysqldump -u root -p[ROOT_PASSWORD] [DATABASE_NAME] > backup.sql
```

#### WordPress Files Backup

The WordPress files are stored in the ./wordpress directory, which can be backed up using standard file backup methods.

### 3. SSL Certificate Renewal

Let's Encrypt certificates expire after 90 days. To renew them:

```bash
sudo certbot renew
```

Setting up a cron job for automatic renewal is recommended:

```bash
0 3 * * * /usr/bin/certbot renew --quiet
```

## Troubleshooting

### 1. Checking Container Status

```bash
docker-compose ps
```

### 2. Viewing Container Logs

```bash
docker-compose logs [service_name]
```

### 3. Accessing Container Shell

```bash
docker exec -it [container_name] /bin/bash
# or for Alpine-based containers
docker exec -it [container_name] /bin/sh
```

### 4. Common Issues

- **Database Connection Issues**: Check the .env file for correct database credentials
- **SSL Certificate Problems**: Verify the certificate paths in the Nginx configuration
- **Permission Issues**: Check file permissions in mounted volumes
- **Network Issues**: Ensure the Docker networks are properly configured