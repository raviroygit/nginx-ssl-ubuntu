Nginx Installation & HTTPS (SSL) Setup with Certbot In AWS EC2
#
aws
#
nginx
#
certbot
#
ssl
Prerequisites
Before we dive into setting up Nginx and SSL, let's start by installing the necessary tools:

Install Certbot and update your package list:
```
sudo apt-get update -y
sudo snap install --classic certbot
```
Install Nginx:
```
 sudo apt install nginx -y
```
With these prerequisites in place, you're ready to secure your EC2 instance with SSL.

Step 1: Obtaining an SSL Certificate with Certbot
Our first task is to obtain an SSL certificate for your domain using Certbot. This certificate will enable HTTPS for your web server.

Stop Nginx temporarily to free up port 80:
 ```
  sudo systemctl stop nginx
```
Run Certbot to obtain the SSL certificate for your domain (replace api.yourdomain.in with your actual domain):
  ```
 sudo certbot certonly --standalone -d api.yourdomain.in
```
Certbot will guide you through the certificate issuance process, prompting you to agree to the terms of service and select the appropriate domain. Once complete, Certbot will store the certificates in /etc/letsencrypt/live/yourdomain.in/.

Step 2: Configuring Nginx for SSL
Now that we have our SSL certificate, it's time to configure Nginx to use it for secure connections.

Create a configuration file for your domain (replace api.yourdomain.in with your actual domain):
 ```
 sudo vim /etc/nginx/sites-available/api.yourdomain.in
```
Add the following configuration to the file, adjusting the server name and SSL certificate paths accordingly:
```
server {
       listen 80;
       server_name api.yourdomain.in;
       return 301 https://$server_name$request_uri;
   }

   server {
       listen 443 ssl;
       server_name api.yourdomain.in;

       ssl_certificate /etc/letsencrypt/live/api.yourdomain.in/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.in/privkey.pem;

       location / {
           proxy_pass http://localhost:8080; # Adjust the port if needed
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
```
Create a symbolic link to enable the site configuration:
 ```
  sudo ln -s /etc/nginx/sites-available/api.yourdomain.in /etc/nginx/sites-enabled/
```
Restart Nginx to apply the changes:
 ```
 sudo systemctl restart nginx
```
Conclusion
Congratulations! You've successfully set up Nginx with SSL on your AWS EC2 instance. Your web server is now configured to provide secure connections using HTTPS. Your users can enjoy a safer browsing experience, and you can rest assured that their data is protected during transit.

Remember that SSL certificates typically expire after a few months, so it's essential to set up automated certificate renewal to keep your website secure.

```
sudo crontab -e
```
This cron job will check for expiring certificates daily and renew them if necessary.
```
0 0 * * * certbot renew
```
Thank you for following this guide, and we hope your web server now runs securely and efficiently. If you encounter any issues or have questions, feel free to leave a comment below.
