# Azuremigration-
To configure NGINX on Azure and meet portal requirements, here’s what you typically need in terms of source code and configuration:

1. Source Code for NGINX Configuration
Example: nginx.conf
This is the primary configuration file for NGINX. Customize it based on your application's requirements.

nginx
Copy code
# /etc/nginx/nginx.conf

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    keepalive_timeout  65;

    # Reverse proxy configuration for backend application
    server {
        listen 80;
        server_name your-domain.com;

        location / {
            proxy_pass http://backend-service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        # Static files (if applicable)
        location /static/ {
            alias /path/to/static/files/;
        }
    }
}
2. Application Deployment Source Code
Place the web application in a directory accessible by NGINX or configure NGINX as a reverse proxy for your application hosted on a different server.

3. Azure Portal Requirements
Deploying NGINX on Azure:
Create a VM:

Use an Ubuntu-based VM from Azure Marketplace.
Configure networking for public/private IP and security groups.
Install NGINX: Connect to the VM via SSH and run:

bash
Copy code
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
Custom NGINX Configuration: Upload or modify the /etc/nginx/nginx.conf or specific site configurations in /etc/nginx/sites-available/.

Expose Ports: Ensure port 80 (and 443 if using SSL) is open in the Azure Network Security Group (NSG) attached to your VM.

DNS Configuration: Point your domain name to the public IP of the Azure VM hosting NGINX.

4. Optional Features
SSL Configuration:
Install SSL certificates using Let's Encrypt:

bash
Copy code
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
Load Balancing:
Modify the NGINX configuration to load-balance between multiple backend servers:

nginx
Copy code
http {
    upstream backend {
        server backend1.com;
        server backend2.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
