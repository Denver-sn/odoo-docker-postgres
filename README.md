

# Guide to Setting Up Odoo with Docker and Nginx Reverse Proxy

## Prerequisites

- Docker and Docker Compose installed
- Basic knowledge of Docker and Nginx
- Domain name configured (e.g., `domain.com`)

## Step 1: Create the Project Directory

Create a project directory to store your Docker Compose file and Nginx configuration:

```sh
mkdir odoo-project
cd odoo-project
```

## Step 2: Create the `.env` File

Create a `.env` file to store environment variables for Odoo and PostgreSQL:

```sh
touch .env
```

Add the following environment variables to the `.env` file:

```env
# Odoo environment variables
HOST=localhost
PORT=8069

# PostgreSQL environment variables
POSTGRES_DB=your_db_name
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password
```

## Step 3: Create the `docker-compose.yml` File

Create a `docker-compose.yml` file with the following content:

```yaml
version: "3"
services:
  odoo:
    image: odoo:15.0
    env_file: .env
    depends_on:
      - postgres
    ports:
      - "127.0.0.1:8069:8069"
    volumes:
      - data:/var/lib/odoo
  postgres:
    image: postgres:13
    env_file: .env
    volumes:
      - db:/var/lib/postgresql/data/pgdata

volumes:
  data:
  db:
```

## Step 4: Install Nginx
```sh
sudo apt update
sudo apt install nginx
sudo ufw allow "Nginx Full"
```

## Step 4.1: Create the Nginx Configuration File

Create an Nginx configuration file to set up the reverse proxy:

```sh
sudo nano /etc/nginx/sites-available/domain_name.com.conf
```

Add the following content to the file:

```nginx
server {
    listen       80;
    listen       [::]:80;
    server_name  my_domain;

    access_log  /var/log/nginx/odoo.access.log;
    error_log   /var/log/nginx/odoo.error.log;

    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Proto https;
      proxy_pass http://localhost:8069;
    }
}
```

## Step 4.2: Linking domain_name.com.conf to /etc/nginx/sites-enabled/
```sh
sudo ln -s /etc/nginx/sites-available/domain_name.com.conf  /etc/nginx/sites-enabled/
```

Test Ngninx Config
```sh
sudo nginx -t
```


## Step 5: Start the Docker Containers

Navigate to your project directory and start the Docker containers using Docker Compose:

```sh
docker-compose up -d
```

## Step 6: Restart Nginx

Restart Nginx to apply the new configuration:

```sh
sudo systemctl restart nginx
```

## Step 7: Verify the Setup

Open your web browser and navigate to your domain (e.g., `http://domain.com`). You should see the Odoo login page.

## SSL (Optional)

```sh
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d domain.com
```


## Conclusion

You have successfully set up Odoo with Docker and configured Nginx as a reverse proxy. You can now use Odoo with your domain name.

