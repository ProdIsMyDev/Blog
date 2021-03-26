---
layout: post
title: "Setup SSL/HTTPS For Your Dockerized Application with Nginx and Let's Encrypt"
date: 2020-11-30 06:30:00 +0100
categories: Guides
---

> **_Preface:_** I will be using a server running Ubuntu 18.04 with Docker already installed.

### Step 1 - Dockerize Your Application:

The easiest way to Dockerize your Application, is to just put the build into one.\
That can be accomplished by a simple Dockerfile like this:
{% highlight Docker %}
FROM adoptopenjdk/openjdk11
COPY <PATH TO YOUR .jar> /home/app.jar
CMD ["java","-jar","/home/app.jar"]
{% endhighlight %}

### Step 2 - Install And Configure Nginx:

Install Nginx:
{% highlight shell %}
sudo apt install nginx -y
{% endhighlight %}

Create a configuration file:
{% highlight shell %}
sudo vim /etc/nginx/sites-available/<YOUR-APPLICATION-NAME>.conf
{% endhighlight %}
With the following content:
{% highlight shell %}
server {
listen [::]:80;
listen 80;

     server_name <YOUR-DOMAIN>;

     location / {
         proxy_set_header X-Forwarded-Host $host;
         proxy_set_header X-Forwarded-Server $host;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_pass http://localhost:<PORT-OF-YOUR-APPLICATION>;
         client_max_body_size 10M;
     }

}
{% endhighlight %}
We need this configuration, to create the SSL certificate.\
We'll implement the actual certificate and redirect later in this guide.

Next create a symlink:
{% highlight shell %}
sudo ln -s /etc/nginx/sites-available/<YOUR-APPLICATION-NAME>.conf /etc/nginx/sites-enabled/<YOUR-APPLICATION-NAME>.conf
{% endhighlight %}
And restart the nginx service:
{% highlight shell %}
sudo service nginx restart
{% endhighlight %}

### Step 3 - Install The Let's Encrypt Python Bot For NGINX:

Install the certbot and run it:
{% highlight shell %}
sudo add-apt-repository -r ppa:certbot/certbot
sudo apt-get update
sudo apt install python-certbot-nginx

sudo certbot --nginx certonly
{% endhighlight %}
You can also use the `python3-certbot-nginx`. Just change the install command accordingly.\
Since we are writing our own nginx config, we'll just create the certificate.

### Step 4 - Finish The NGINX Configuration:

Now that we have successfully created the certificates, we can finish our configuration file:
{% highlight shell %}
server {
listen [::]:80;
listen 80;

     server_name <YOUR-DOMAIN>;


     return 301 https://<YOUR-DOMAIN>$request_uri;

}

server {
listen [::]:443 ssl;
listen 443 ssl;

     server_name <YOUR-DOMAIN>;

     ssl_certificate /etc/letsencrypt/live/<YOUR-DOMAIN>/fullchain.pem;
     ssl_certificate_key /etc/letsencrypt/live/<YOUR-DOMAIN>/privkey.pem;

     location / {
         proxy_set_header X-Forwarded-Host $host;
         proxy_set_header X-Forwarded-Server $host;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_pass http://localhost:<PORT-OF-YOUR-APPLICATION>;
         client_max_body_size 10M;
     }

}
{% endhighlight %}
And finally restart the nginx service once again:
{% highlight shell %}
sudo service nginx restart
{% endhighlight %}