# Node.js Deployment

> Steps to deploy a Node.js app to DigitalOcean using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## Architecture (what we want)
```
Browser
   ↓
Nginx :80
   ├── /        → React (client/dist)
   └── /api     → Node.js backend (localhost:8080 via PM2)
```

## 1. Create Free AWS Account
Create free AWS Account at https://aws.amazon.com/

## 2. Create and Lauch an EC2 instance and SSH into machine
I would be creating a t2.medium ubuntu machine for this demo.

## 3. Install Node and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
```
git clone https://github.com/piyushgargdev-01/short-url-nodejs
```

## 5. ✅ STEP 1: Build the frontend (client)
```
cd ~/NexAI/client
npm install
npm run build

```

## 5. Install dependencies and test app(server)
```
npm install
sudo npm i pm2 -g
pm2 start server

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```

## 6. Fix frontend API URL (VERY COMMON MISTAKE)
```
VITE_API_URL=/api

```

## 7. Setup Firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```

sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
  server {
    listen 80;
    server_name _;

    # ---------- FRONTEND ----------
    root /home/ubuntu/NexAI/client/dist;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    # ---------- BACKEND ----------
    location /api {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}


```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo nginx -s reload
```

## 9. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```
## 10. when there is more than 1 server and also it will send user request in round robin format
```
http {

    upstream backend_servers {

        server 127.0.0.1:3000;

        server 127.0.0.1:3001;

        server 127.0.0.1:3002;

    }


    server {

        listen 80;

        server_name example.com;


        location / {

            proxy_pass http://backend_servers;

        }

    }

}
```
