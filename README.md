# docker-php
Spin up an entire php infrastructure using docker.
The infrastructure this docker-compose file creates is:

<pre><b>NGINX ( Webserver / Load Balancer ) --> 2 PHP/Apache Servers ---> MYSQL Database.</b></pre><br>

1st Nginx Folder
Contains : Dockerfile, nginx.conf and script.sh

Dockerfile for Nginx:
We are using a centos image for this. And then installing the nginx image.
Script file depends on personal choice, you can create one or you can opt out of it.

nginx.conf file for Nginx:
You can make use of default file for NGINX or you can copy the one i created.
It doesn't have the load balancer configurations yet, but once the NGINX is up you can add those information depending on the IP's of your webserver.
Location for the file is : <b> /etc/nginx/ </b></br>
Syntax : Add the following 
<pre><b>
http {
  upstream allbackend {
    server <webserver-IP>
    server <webserver-IP>
  }
  
  server{
    listen 8080;
    root /usr/share/nginx/html;
   }
   
   server{
     listen 8081;
     location / {
       proxy_pass http://allbackend;
     }
    }
}
</b></pre>
I'm using the default document root for the NGINX to serve pages. But when launching you can attach it to any other local storage you want. Would explain it when i talk about
the docker compose file.

Now lets talk about the php-apache folder. It has 2 files Dockerfile and script.sh

Here too we are using a centos image, and installing couple of softwares we need for this to work.
Now some of us might face an issue for the PHP not working.
I was getting an Service unavailable error. So i was able to find a work around for it. 
So we are creating the <b>/var/run/php-fpm</b>, most of the time this folder i.e <b>php-fpm</b> is not created and that causes the issue.
Now the script has httpd and php-fpm to start the respective services. Recommended to use the script but again personal preference.
I know we should not have 2 services in the same container, but i didn't find any significant usecase to have them in seperate containers. Again personal preference.

Now the most important part of the project the <b>docker-compose file</b>.

Remember to use <b>build instead of image</b> attribute, if you don't create an image before using the docker-compose file.
Replace the name of the image with the image name you created or else use the build attribue.</br>

First service is the <b>NGINX</b> server.
Here we are exposing 2 ports. The default files are being exposed from 8080 and the PHP is served through 8081 as you can already see in the above nginx.conf file.
Next i'm attaching a local storage to the default NGINX document root.
But if you are doing this remember to check the <b>SELINUX</b> context. As it might cause problems and you would get forbidden error.
You can change the SELINUX context of the local folder by using the following syntax,
<pre><b>      
      ls -dZ /your-folder/  <--- to check the selinux labels.
      semanage fcontext -a -t httpd_sys_content_t '/folder(/.*)?'
      restorecon -RFvv /folder/ <--- to relabel it with the new type we mentioned above.
</b></pre>
But if you don't want to go through all this hassle, you can just simply remove the volumes line from the webserver.

After checking, i found that <b>mysql:5.7</b> is compatible with the PHP version we are using. 
We are just using the image from docker hub. You can simply pull it from docker hub.
We are defining some environmental variables, you can change them as per your needs.

Last we create the php/apache servers, 
We have to link php and mysql and so we are using the <b>depends_on</b> attribute, if you have created your custom network you dont need that attribute. 
I'm using the default network, so i need that attribute as i don't want to connect them using IP addresses.
Also i'm using a local storage to connect with the container storage. Again personal preference, i have done it so that there can be same data in my multiple php containers.
We don't have to expose the ports in this server, the pages are being served from the NGINX servers.

<b>Notes</b> : I had an issue as the containers would stop after running the script when i used docker-compose.</br>
<b>Solution</b>: i added the <b>tty: true</b> to each of the services and now they run completely fine. There are many workarounds for this issue.
I went with the one i described.

Now we want to spin up multiple php/apache servers, so that can be achieved as following,
<pre>
<b>docker-compose up --scale <container-name>=value
where value is the number of containers you want.
and container-name = dbos, myserver etc
</b></pre>

Some useful commands,
1) to check the IP,</br>
<pre><b>docker inspect container-name</b></pre>
2) to reload the NGINX server without stopping them,</br>
<pre><b>docker exec -it <container-name> nginx -s reload</b></pre>

Thanks! Do let me know what changes i can do to make it better!
