# Content

- [Launch WordPress With OpenSSL](#wp-ssl)

- [Configure Cache Static File](#static)

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

[^1]: https://www.techrepublic.com/article/how-to-cache-static-content-on-nginx/
