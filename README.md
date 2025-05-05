# 🚀 LEMP Wordpress & SSL with Docker-Compose
by Luca Facchini ([lucafacchini0](https://github.com/lucafacchini0))

A complete, production-ready WordPress environment with SSL support, running in Docker containers. This setup includes Nginx as a web server with SSL termination, WordPress with PHP-FPM, MySQL for database storage, and phpMyAdmin for database management.

## 🔐 Features

- **Secure by Default**: SSL/TLS encryption with Let's Encrypt certificates
- **Performance Optimized**: Nginx with HTTP/2 support and optimized configuration
- **Containerized**: Complete isolation between components
- **Easy Management**: phpMyAdmin included for database administration
- **Security Headers**: Comprehensive security headers configuration
- **Scalable**: Easy to scale with Docker Compose

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- A domain name pointed to your server's IP address
- Basic understanding of Docker and WordPress

## 🛠️ Setup Instructions

### Step 1: Clone the Repository

```bash
git clone https://github.com/lucafacchini0/setup-wordpress-dockercompose.git
cd setup-wordpress-dockercompose
```

### Step 2: Generate SSL Certificates

⚠️ **IMPORTANT**: Before running the containers, you need to generate SSL certificates using Let's Encrypt and create a Diffie-Hellman parameter file.

```bash
# Install certbot (if not already installed)
sudo apt-get update
sudo apt-get install certbot

# Generate certificates for your domain
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com

# Generate Diffie-Hellman parameters (this may take a while)
sudo mkdir -p /etc/nginx/ssl
sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

### Step 3: Configure Environment Variables

```bash
# Edit the .env file with your settings
nano .env
```

Update the following variables in the `.env` file:

```
DOMAIN=yourdomain.com
SQL_ROOT_PASSWORD=your_secure_root_password
SQL_DATABASE_NAME=wordpress
SQL_DATABASE_USER=wordpress_user
SQL_DATABASE_PASSWORD=your_secure_password
```

### Step 4: Update Nginx Configuration

Edit the Nginx configuration file to use your domain name:

```bash
nano nginx/conf.d/YOUR_DOMAIN.com.conf
```

and replace YOUR_DOMAIN with the domain you generated SSL Certificates for.

### Step 5: Start the Containers

```bash
docker-compose up -d
```

### Step 6: Complete WordPress Installation

Visit `https://yourdomain.com` in your browser and follow the WordPress installation wizard.

## 🔧 Accessing Services

- **WordPress**: https://yourdomain.com
- **phpMyAdmin**: http://localhost:8080 (only accessible from the local server)

## 🔄 Maintenance

### Updating WordPress

WordPress updates can be managed through the WordPress admin dashboard or by updating the Docker image version in the `docker-compose.yaml` file.

### Renewing SSL Certificates

Let's Encrypt certificates expire after 90 days. To renew them:

```bash
sudo certbot renew
```

Consider setting up a cron job to automate this process.

## 📚 Documentation

For more detailed information about each component and configuration, please check the `/docs` directory.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⚠️ Security Note

This setup includes security best practices, but you should always keep your systems updated and regularly check for security advisories related to WordPress, Nginx, MySQL, and phpMyAdmin.
