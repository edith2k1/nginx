# Content

- [Launch WordPress With OpenSSL](#wp-ssl)

- [Configure Cache Static File](#static)

- [Amplify – NGINX Monitoring Made Easy](#amplify)

- [Configure NGINX as a Reverse Proxy](#proxy)
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

[^1]: https://www.techrepublic.com/article/how-to-cache-static-content-on-nginx/
[^2]: https://www.tecmint.com/amplify-nginx-monitoring-tool/
[^3]: https://viblo.asia/p/cau-hinh-reverse-proxy-tren-nginx-Az45bGxqKxY
