# Content

- [Launch WordPress With OpenSSL](#wp-ssl)

- [Configure Cache Static File](#static)

- [Amplify – NGINX Monitoring Made Easy](#amplify)

- [Configure NGINX as a Reverse Proxy](#proxy)

- [Block IP](#ip)

- [Install ApacheBench](#ab)
***

## Launch WordPress With OpenSSL <a id="wp-ssl"></a>

Click [here](https://github.com/edith2k1/wordpress-ssl) to find the solution.

***

## Configure Cache Static File <a id="static"></a> [^1]

Run this command

    sudo vi /etc/nginx/sites-available/default

 Within that file, scroll down to the bottom of the server block and add the following

    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
          expires 7d;
    }

Reload NGINX with the command 

    sudo systemctl reload nginx 

### Testing

1. Create an index.html file

       sudo vi /var/www/wordpress/index.html

      > Copy this to the file

       <!DOCTYPE html>
       <html lang="en">

       <head>
         <meta charset="UTF-8">
         <meta http-equiv="X-UA-Compatible" content="IE=edge">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>Document</title>
         <link rel="stylesheet" href="css.css">
       </head>

       <body>
         <h1>Hello World</h1>
         <img src="linux-bsd.gif" alt="Linux">
         <p>
           Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat.
           Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat.
         </p>
       </body>

       </html>

2. Create a css.css file

       sudo vi /var/www/wordpress/css.css

      > Copy this to the file

       body {
          background-color: powderblue;
       }
       h1 {
          color: blue;
       }
       p {
          color: red;
       }

3. Download an image

       sudo wget https://www.electrictoolbox.com/images/icons/linux-bsd.gif

4. Results

    ![image](https://user-images.githubusercontent.com/100410064/167249968-0b7d3839-5ae6-435b-8f90-bdda8c498e12.png)
    
    ![image](https://user-images.githubusercontent.com/100410064/167250036-585c3fd9-4047-430e-a6f6-d8d58bb93957.png)

***

## Amplify – NGINX Monitoring Made Easy <a id="amplify"></a>[^2]

**Step 1: Install Amplify Agent on Linux System**

1. Open your web browser, type the address below and create an account. A link will be sent to your email, use it to verify the email address and login to your new account.

       https://amplify.nginx.com

2. Use SSH to connect and log in to the system that you want to monitor and download the nginx amplify agent auto-install script. 

       curl -L -O https://github.com/nginxinc/nginx-amplify-agent/raw/master/packages/install.sh

3. Now run the command below with superuser privileges using the `sudo command`, to install the amplify agent package (the API_KEY will probably be different, unique for every system that you add).

       sudo API_KEY='bd26251338a5f2c86e820d2038c16956' sh ./install.sh

4. After a successful installation, the new system appears in the list on the left in about 1 min or so.

**Step 2. Configure NGINX to visualize essential metrics**

1. Create a new file with the stub_status configuration.

       sudo vi /etc/nginx/conf.d/stub_status.conf

    Then copy/paste the following lines to your terminal window.

       server {
          listen 127.0.0.1:80;
          server_name 127.0.0.1;
          location /nginx_status {
            stub_status on;
            allow 127.0.0.1;
            deny all;
          }
        }

2. Reload NGINX so that the stub_status module configuration becomes active.

       sudo systemctl restart nginx

**Step 3. Set up additional NGINX metrics to learn more about the application performance**

1. Run this command

       sudo vi /etc/nginx/nginx.conf

2. Delete all and paste this to the file

       user www-data;
       worker_processes auto;
       pid /run/nginx.pid;
       include /etc/nginx/modules-enabled/*.conf;

       events {
         worker_connections 768;
       }

       http {

         sendfile on;
         tcp_nopush on;
         tcp_nodelay on;
         keepalive_timeout 65;
         types_hash_max_size 2048;

         include /etc/nginx/mime.types;
 	     default_type application/octet-stream;

         ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
         ssl_prefer_server_ciphers on;

	     log_format  main_ext  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" '
                      '"$host" sn="$server_name" '
                      'rt=$request_time '
                      'ua="$upstream_addr" us="$upstream_status" '
                      'ut="$upstream_response_time" ul="$upstream_response_length" '
                      'cs=$upstream_cache_status' ;	

         access_log  /var/log/nginx/access.log  main_ext;
         error_log  /var/log/nginx/error.log warn;

         gzip on;

         include /etc/nginx/conf.d/*.conf;
         include /etc/nginx/sites-enabled/*;
       }
       
3. Now restart Nginx services once more, to effect the latest changes.

       sudo systemctl restart nginx
       
**Step 4. Monitor Nginx Web Server Via Amplify Agent**
       
Finally, you can begin monitoring your Nginx web server from the Amplify Web UI.

![image](https://user-images.githubusercontent.com/100410064/167259469-e8a2174a-85b7-4b45-b0ab-0012f6f7f41a.png)

![image](https://user-images.githubusercontent.com/100410064/167259520-5d794c0b-0f7b-4adf-b042-84fb3c74023a.png)

***

## Configure NGINX as a Reverse Proxy <a id="proxy"></a> [^3]

Step 1. Create a new config file

    sudo vi /etc/nginx/conf.d/domain.conf
    
> Copy this to the file

    server {
        listen 81;
        server_name 0.0.0.0; #change to your domain name

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_pass http://192.168.217.140:8080;  #change to your internal server IP
                proxy_redirect off;
        }
    }
    
> `http://192.168.217.139:81` ===> `http://192.168.217.140:8080`
    
Step 2. Restart Nginx Service

    sudo service nginx restart
    
### Testing

![image](https://user-images.githubusercontent.com/100410064/167261556-35792c61-744d-4da8-b745-4c812a91654a.png)

***

## Block IP <a id="ip"></a> [^4][^5]

Before Block

![image](https://user-images.githubusercontent.com/100410064/167282816-4fef60a4-6859-4bfb-aee2-a001b7bd5b37.png)

Run this command

    sudo vi /etc/nginx/sites-available/default
    
Edit like this to block IP 192.168.217.140

    server {

        ...

        error_page   403  /403.html;
        location /403.html {
            root      /var/www/html;
            allow all;
        }
    
        ...
    
        location / {
            ...
            deny    192.168.217.140;
            ...
        }
    }
    
Custom 403 error page

    sudo vi /var/www/html/403.html;
    
Paste this to the file

    Your IP is blocked

Restart Nginx Service

    sudo service nginx restart
    
Testing

![image](https://user-images.githubusercontent.com/100410064/167283202-86717f8d-d003-493d-b11e-6fb5489390ae.png)

***

## Install ApacheBench <a id="ab"></a> [^6][^7]

Run this command

    sudo apt-get install apache2-utils -y
    
Testing

    ab -n 100000 -c 1000 https://192.168.217.139/
    
   > -n requests     Number of requests to perform
   > 
   > -c concurrency  Number of multiple requests to make at a time

Result

    This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking 192.168.217.139 (be patient)
    Completed 10000 requests
    Completed 20000 requests
    Completed 30000 requests
    Completed 40000 requests
    Completed 50000 requests
    Completed 60000 requests
    Completed 70000 requests
    Completed 80000 requests
    Completed 90000 requests
    Completed 100000 requests
    Finished 100000 requests


    Server Software:        nginx/1.18.0
    Server Hostname:        192.168.217.139
    Server Port:            443
    SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256
    Server Temp Key:        X25519 253 bits

    Document Path:          /
    Document Length:        19 bytes

    Concurrency Level:      1000
    Time taken for tests:   104.282 seconds
    Complete requests:      100000
    Failed requests:        0
    Non-2xx responses:      100000
    Total transferred:      19800000 bytes
    HTML transferred:       1900000 bytes
    Requests per second:    958.94 [#/sec] (mean)
    Time per request:       1042.819 [ms] (mean)
    Time per request:       1.043 [ms] (mean, across all concurrent requests)
    Transfer rate:          185.42 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:       12  935 2370.2    506   77772
    Processing:     0   28  81.0      2    2546
    Waiting:        0   19  61.3      2    2546
    Total:        105  962 2376.1    512   77803

    Percentage of the requests served within a certain time (ms)
      50%    512
      66%    583
      75%    697
      80%    782
      90%   1537
      95%   3300
      98%   4393
      99%   7748
     100%  77803 (longest request)

***

[^1]: https://www.techrepublic.com/article/how-to-cache-static-content-on-nginx/
[^2]: https://www.tecmint.com/amplify-nginx-monitoring-tool/
[^3]: https://viblo.asia/p/cau-hinh-reverse-proxy-tren-nginx-Az45bGxqKxY
[^4]: https://vinasupport.com/chan-dia-chi-ip-address-tren-nginx-web-server/
[^5]: https://stackoverflow.com/questions/3119108/return-custom-403-error-page-with-nginx
[^6]: https://viblo.asia/p/benchmark-peformance-on-web-server-season-1-924lJp3zKPM
[^7]: https://www.digitalocean.com/community/tutorials/how-to-use-apachebench-to-do-load-testing-on-an-ubuntu-13-10-vps
