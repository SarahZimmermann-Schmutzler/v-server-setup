# Configurate a Web Server

1. [The Web Server - Nginx](#the-web-server---nginx)
   * [Install and Activate Nginx](#install-and-activate-nginx)
   * [Configure Nginx](#configure-nginx)
     * [Create a Customized Page and configure it](#create-a-customized-page-and-configure-it)
     * [Set the Alternate Page as Homepage](#set-the-alternate-page-as-homepage)

## The Web Server - Nginx

If you enter your IP-address in the browser's address bar and press enter, nothing happens - the browser sends HTTP(S) requests to the VM, but no program receives and responds to them.

To access the VM via the web browser, a **web server** like `Nginx` is required to process the requests:

* It is fast and lightweight. It requires few resources, has built-in reverse proxy and load balancing, is easy to set up, and is simple to configure. It provides an optimal foundation for modern web projects.

### Install and Activate Nginx

1. Update and upgrade the VM like you should do regularly:

    ```bash
    sudo apt update && sudo apt upgrade
    ```
  
1. **Install `Nginx`**:

    ```bash
    sudo apt install nginx -y
    ```
  
1. Check if the **status** is `active`:

    ```console
    systemctl status nginx
    ```

### Configure Nginx

If you now **browse the IP-address of your VM** you are welcomed by the **Nginx homepage**. It is stored under `/var/www/html/index.nginx-debian.html`.  
  
The following steps shows how to proceed if you want to **be greeted by your own page**.

#### Create a Customized Page and configure it

1. Create a **directory called `alternatives`** under the following path:  

    ```bash
    sudo mkdir /var/www/alternatives
    ```
  
1. Add a customized index page called **alternate-index.html**.  

    ```bash
    sudo nano /var/www/alternatives/alternate-index.html
    ```
  
* Here is an [example](./alternate-index.html) of what it can look like.  

By default, the **default configuration file** is loaded, which contains the standard Nginx homepage: `/etc/nginx/sites-enabled/default`.  
In order for the `alternate-index.html` to be loaded by Nginx, a **special configuration file** is needed.

1. Create a **configuration file named `alternatives`** with the following **content**:

    ```bash
    sudo nano /etc/nginx/sites-enabled/alternatives
    ```
  
    ```nginx
    server {
            listen 8081;
            listen [::]: 8081;

            root /var/www/alternatives;
            index alternate-index.html

            location / {
                     try_files $uri $uri/ =404;
            }
    }
    ```

* <ins>Explaination</ins>:
  * **server block**: It defines a virtual website that listens on specific ports and domains and controls how requests are processed and content is delivered.

    * **listen 8081 and [::]: 8081**: The web server now listens on IP-port 8081 for IPv4 and IPv6. Requests that go to this port end up in this server block.
      * **`alternate-index-html` will be running at `http://ip-adress_vm:8081`**
  
    * **root**: Specifies the root directory for this website. All URLs processed here refer to files in this folder.

    * **index**: When a user simply calls / (all paths, no specific file: http://IP-address:8081/), nginx tries to load alternate-index.html.
  
    * **location block**: Takes care of the paths that come after http://IP-address:8081/. In this case, it checks whether a file ($uri) or directory ($uri/) with exactly this path exists. If it doesn't, a 404 error is displayed. This prevents nginx from forwarding anything that doesn't exist.
      * The url `http://ip-address_vm/abc` will return the error `404 Not Found` because there is no file or dirextory called *abc*.

2. **Restart Nginx**:

    ```bash
    sudo service nginx restart
    ```
  
#### Set the Alternate Page as Homepage

Requests without a port specification land on port 80 (http) by default. This is where the Nginx standard page is currently displayed.  
To display the **alternative page as the homepage**, a redirect can be set up in the server block on port 80, which **automatically forwards requests to port 8081**:

1. **Modify the default configuration file** under `/etc/nginx/sites-enabled/default`:  

    ```nginx
    server {
           listen 80 default_server;
           listen [::]: 80 default_server;

           server_name _;

           #forwarding to port 8081 (alternate-index.html)
           location / {
                    proxy_pass http://localhost:8081;
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto $scheme;
           }
    }
    ```

1. Perform a **syntax and plausibility check**:  

    ```bash
    sudo nginx -t
    ```
  
1. If it was successful, **restart Nginx**:  

    ```bash
    sudo service nginx restart
    ```
  
1. If you now **browse the IP-address of your VM**, the **alternative homepage** should be displayed:

  ![alternate_homepage](./img/alternate.png)