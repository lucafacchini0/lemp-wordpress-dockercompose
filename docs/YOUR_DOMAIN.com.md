

This document provides a comprehensive explanation of the Nginx server configuration for YOUR_DOMAIN, designed for complete beginners.

## What is this file?

This file (`YOUR_DOMAIN.com.conf`) is a configuration file for the Nginx web server. It tells Nginx how to handle web traffic for your specific domain. Think of it as a set of instructions that determine how visitors interact with your website.

## Server Blocks Explained

Nginx uses "server blocks" (similar to Apache's virtual hosts) to handle different domains or scenarios. Each `server { ... }` section in the file is a separate server block with its own settings.

### 1. IP Address Blocking (HTTP)

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name YOUR_IP_ADDRESS;
    access_log off;
    return 444;
}
```

#### Detailed explanation:

- `listen 80;` - Tells Nginx to listen for regular HTTP connections on port 80 (the standard HTTP port)
- `listen [::]:80;` - Same as above but for IPv6 connections (the [::] represents IPv6)
- `server_name YOUR_IP_ADDRESS;` - This block applies when someone tries to access your site using the IP address directly instead of the domain name
- `access_log off;` - Disables logging for these requests to save disk space (since these are likely bots or attackers)
- `return 444;` - Returns a special Nginx-only code (444) that immediately closes the connection without sending any response. This is more secure than sending a 403 (Forbidden) because it gives attackers no information at all.

**Why this matters**: This block prevents people from accessing your site via the IP address, which is a security best practice. It forces visitors to use your domain name.

### 2. IP Address Blocking (HTTPS)

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name YOUR_IP_ADDRESS;

    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    access_log off;
    return 444;
}
```

#### Detailed explanation:

- `listen 443 ssl;` - Listen for HTTPS connections on port 443 (the standard HTTPS port)
- `listen [::]:443 ssl;` - Same as above but for IPv6 connections
- `server_name YOUR_IP_ADDRESS;` - This block applies when someone tries to access your site using the IP address with HTTPS
- `ssl_certificate` and `ssl_certificate_key` - Paths to the SSL certificate files. For IP-based access, we use a self-signed certificate (not trusted by browsers, but that's fine since we're blocking access anyway)
- `ssl_protocols TLSv1.2 TLSv1.3;` - Only allow modern, secure TLS protocols (versions 1.2 and 1.3)
- `access_log off;` - Disables logging for these requests
- `return 444;` - Immediately closes the connection without response

**Why this matters**: This complements the first block by also blocking HTTPS access via IP address. Together, they ensure visitors can only access your site through the domain name.

### 3. HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    server_name www.YOUR_DOMAIN YOUR_DOMAIN;
    return 301 https://YOUR_DOMAIN$request_uri;
}
```

#### Detailed explanation:

- `listen 80;` - Listen for HTTP connections on port 80
- `server_name www.YOUR_DOMAIN YOUR_DOMAIN;` - This block applies when someone visits either "www.yourdomain.com" or "yourdomain.com" using HTTP
- `return 301 https://YOUR_DOMAIN$request_uri;` - Sends a 301 (permanent redirect) to the HTTPS version of your site
    - `301` - A permanent redirect that tells browsers and search engines this is a permanent change
    - `https://YOUR_DOMAIN` - The destination (your domain with HTTPS)
    - `$request_uri` - A variable that contains the original request path and query string, ensuring the visitor lands on the same page they requested

**Why this matters**: This block ensures all traffic uses HTTPS, which is essential for security and SEO. Google prefers HTTPS sites, and browsers show warnings for non-HTTPS sites.

### 4. WWW to Non-WWW Redirect (HTTPS)

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name www.YOUR_DOMAIN;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 4h;

    return 301 https://YOUR_DOMAIN$request_uri;
}
```

#### Detailed explanation:

- `listen 443 ssl http2;` - Listen for HTTPS connections on port 443 with HTTP/2 protocol support
- `listen [::]:443 ssl http2;` - Same as above but for IPv6
- `server_name www.YOUR_DOMAIN;` - This block applies when someone visits "www.yourdomain.com" using HTTPS
- SSL configuration:
    - `ssl_certificate` and `ssl_certificate_key` - Paths to your Let's Encrypt SSL certificate files
    - `ssl_protocols TLSv1.2 TLSv1.3;` - Only allow modern, secure TLS protocols
    - `ssl_ciphers HIGH:!aNULL:!MD5;` - Only use strong encryption methods, exclude anonymous and MD5 ciphers
    - `ssl_prefer_server_ciphers on;` - Prefer the server's cipher order over the client's
    - `ssl_session_cache shared:SSL:10m;` - Cache SSL sessions for 10 megabytes of memory to improve performance
    - `ssl_session_timeout 4h;` - Keep SSL sessions in cache for 4 hours
- `return 301 https://YOUR_DOMAIN$request_uri;` - Permanently redirect to the non-www version of your site

**Why this matters**: This standardizes your site on a single domain (non-www), which is important for SEO (prevents duplicate content) and provides a consistent user experience.

### 5. Main Website Configuration

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name YOUR_DOMAIN;

    root /var/www/html;
    index index.php;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ecdh_curve secp384r1;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";

    ssl_session_cache shared:le_nginx_SSL:10m;
    ssl_session_timeout 1440m;
    ssl_session_tickets off;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Header security
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options DENY always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer" always;
    add_header Content-Security-Policy "script-src 'self'; object-src 'self'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header Cross-Origin-Embedder-Policy require-corp always;
    add_header Cross-Origin-Opener-Policy same-origin always;
    add_header Cross-Origin-Resource-Policy same-origin always;
    add_header Expect-CT "max-age=86400, enforce" always;
    add_header Permissions-Policy "fullscreen=(), geolocation=(), microphone=(), camera=()" always;

    # CORS
    add_header Access-Control-Allow-Origin "https://YOUR_DOMAIN" always;
    add_header Access-Control-Allow-Methods "GET, POST, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
    add_header Timing-Allow-Origin "none" always;

    # Log files
    access_log /var/log/nginx/YOUR_DOMAIN.access.log;
    error_log /var/log/nginx/YOUR_DOMAIN.error.log;

    # Location blocks
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass wordpress:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    location ~ ^/(wp-login\.php|xmlrpc\.php) {
        limit_req zone=wordpress burst=5 nodelay;

        # Existing fastcgi configuration for PHP
        try_files $uri =404;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
}
```

#### Detailed explanation of each section:

**Basic Server Settings**
- `listen 443 ssl http2;` and `listen [::]:443 ssl http2;` - Listen for HTTPS connections with HTTP/2 support (HTTP/2 is faster than HTTP/1.1)
- `server_name YOUR_DOMAIN;` - This is the main server block for your domain
- `root /var/www/html;` - The directory where your website files are stored
- `index index.php;` - The default file to serve when a directory is requested (looks for index.php first)

**SSL Configuration (Very Important for Security)**
- `ssl_certificate` and `ssl_certificate_key` - Paths to your Let's Encrypt SSL certificate files
- `ssl_protocols TLSv1.2 TLSv1.3;` - Only allow modern, secure TLS protocols (versions 1.2 and 1.3)
- `ssl_ecdh_curve secp384r1;` - Specifies the elliptic curve for ECDHE ciphers (more secure than the default)
- `ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:...";` - A list of allowed encryption ciphers, ordered from most to least preferred
    - These are all modern, secure ciphers that provide forward secrecy
    - The long string includes multiple cipher options for compatibility with different clients
- `ssl_session_cache shared:le_nginx_SSL:10m;` - Allocates 10MB of shared memory for SSL session caching to improve performance

**Security Headers**
- `X-Content-Type-Options nosniff` - Prevents browsers from MIME-sniffing a response away from declared content type
- `X-Frame-Options DENY` - Prevents the page from being displayed in a frame, protecting against clickjacking
- `X-XSS-Protection "1; mode=block"` - Enables XSS protection in browsers
- `Referrer-Policy "no-referrer"` - Controls the referrer information sent with requests
- `Content-Security-Policy "script-src 'self'; object-src 'self'"` - Restricts where resources can be loaded from
- `Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"` - Enforces HTTPS and prevents SSL stripping attacks

**Rate Limiting**
- `limit_req zone=wordpress burst=5 nodelay;` - Limits requests to WordPress login and XML-RPC endpoints
    - `zone=wordpress` - References the rate limit zone defined in the nginx.conf
    - `burst=5` - Allows bursts of up to 5 requests
    - `nodelay` - Applies the rate limit immediately without delay

**Location Blocks**
- `/` - Main location block that tries to serve static files first, then falls back to index.php
- `\.php$` - Handles PHP files by passing them to the WordPress FastCGI process
- `\.(js|css|png|jpg|jpeg|gif|ico|svg)$` - Serves static files with maximum cache expiration
- `^/(wp-login\.php|xmlrpc\.php)` - Special handling for WordPress login and XML-RPC endpoints with rate limiting
- `/favicon.ico` - Special handling for favicon requests
- `/robots.txt` - Special handling for robots.txt file