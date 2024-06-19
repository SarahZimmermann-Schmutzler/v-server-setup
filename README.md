# HOW TO SET UP A VM  
sources: Developer Akademie (DevSecOps Masterclass); ChatGPT for debugging and deeper definitions; google translate for translation

## What exactly is a VM?  
Long story short - A VM or Virtual Machine is a software program that runs on a 
physical server and emulates it. A VM can run an operating system and applications as if they were running on real hardware.
A software environment called Hypervisor creates and manages the VMs.

## The Login

### Create a SSH-Key for your local Server  
SSH (Secure Shell) is a protocol that allows secure access to a remote computer over an insecure network.
An SSH-Key (Secure Shell Key) is a cryptographic key pair used to authenticate SSH connections.SSH-Keys are more secure than password-based authentication because they use strong encryption. 
Once set up, you no longer have to enter a password to connect to the server. 
They are also well suited for automated processes and scripts that need to access remote servers in a secure manner. 
The SSH-Key pair consists of a private key and a public key.  
The **private key** should be kept secure and secret. It is saved on the client (your computer) and is used to authenticate to a server.  
The **public key** can be passed on without hesitation or stored on the server you want to log in to. 
It is used to encrypt messages that can only be decrypted with the corresponding private key.

#### Procedure  
1. Open the program *Git Bash* as admin  
  
2. Create a ED25519 SSH-Key pair (more secure than RSA SSH-Key pair)  
>`ssh-keygen -t ed25519`  
choose to use a password or not

### Store the SSH-Key on your VM  
You can login to your vm without crating a SSH-Key. But then you always need the password and that is laborious and insecure (by the way).
Why? Basically every password can be bruteforced.  
    i: A brute force attack is a method in which an attacker systematically tries all possible combinations of passwords to find the right combination and gain unauthorized access to a system or account.  
That's why we want to store the SSH-Key on the vm so that we can login with it instead of a password.

#### Procedure  
copy your public SSH-Key  
    `ssh-copy-id -i path/to/your/id_ed25519.pub user_vm@ip-address_vm`  
at best the public SSH-Key is now stored in `~/.ssh/authorized_keys` - or not...  
Why? Honestly I don't know.  
    i: Note from the future: Maybe the problem were the wrong question marks. I prewrote the command in MSWord and ChatGPT told me, that they are graphic question marks then. So don't do that.  
copy the public SSH-Key after displey it with:  
    `cat ~/.ssh/id_ed25519.pub`  
login to your vm  
    `ssh user_vm@ip-address_vm`  
enter your password  
at best the vm environment has opened  
add the SSH-Key to the file authorized_keys and save it  
    `sudo nano ~/.ssh/authorized_keys`  
after you logged out from your VM with `logout` or `exit`, you can now login with `ssh user_vm@ip-address_vm` and without password

### Deactivate the possebility to login with a password  
Now we want that NOBODY could login to our VM with a username password combination.
We have to deactivate the PasswordAuthentication option in the sshd config file.

#### Procedure  
open the config file with an editor  
    `sudo nano /etc/ssh/sshd_config`  
activate the option `PasswordAuthentication` and change from `yes` to `no`  
restart the ssh service  
    `sudo systemctl restart ssh.service`  
how to proof that it worked? we know that the login without password is activated and the SSH-Key connection is working. But we don't know if it is impossible to login with one. We can test ist with:  
    `ssh -o PubkeyAuthentication=no user_vm@ip-address_vm`  
if you get the information `user_vm@ip-address_vm: Permission denied (pubkey)` then it worked out  
    i: The command explicitly disables public key authentication and attempts to use password authentication instead.
       However, if the server is configured to only accept public keys, authentication will fail.

## The Webserver - Nginx 
Yesssss, I have a VM! Let's type the IP-address in the address bar of the browser and have a look what happens. Nothing?  
The accessibility of the VM from the Internet depends heavily on the network configuration, particularly on how ports and services are configured.
There are some reasons why you can't open the VM in the webbrowser. However we need a webserver to do so.


### Intall and activate Nginx

#### Procedure  
update the vm (you should do it regularly)  
    `sudo apt update`  
install nginx  
    `sudo apt install nginx -y`  
    i: Why -y? Y stands for yes. The answer of the question the command line would ask you instead in the next step: Are you sure you want to install Nginx?  
check the status of Nginx  
    `systemctl status nginx`  
uhhh, it's already active!

### Configurate Nginx  
If we now browse our IP-address we are welcomed from the Nginx homepage. But we want our own homepage, don't we?

#### Procedure  
the default homepage is stored under `/var/www/html/index.nginx-debian.html`  
to get our own homepage we create the folder *alternatives* under the path  
   `sudo mkdir /var/www/alternatives`  
and gives it an alternate index.html  
    `sudo mkdir /var/www/alternatives/alternate-index-html`  
you can write:  
    `<!DOCTYPE html>  
     <html>  
     <head>  
      <meta charset="utf-8">  
      <title>Hello, Nginx!</title>  
     </head>  
     <body>  
      <h1>Hello, I am an alternative Homepage for Nginx</h1>  
      <p>Noice to see ya :)</p>  
     </body>  
     </html>`  
we have a html now and we want Nginx to load it. create a Nginx config file  
    `sudo nano /etc/nginx/sites-enabled/alternatives`  
    i: by default there is only the default config file that loads the current homepage `/etc/nginx/sites-enabled/default`  
create a server and a location block  
    `server {  
            listen 8081;  
            listen [::]: 8081;  
            root /var/www/alternatives;  
            index alternate-index.html;  
            location / {  
                       try_files $uri $uri/ =404;  
            }  
     }`  
wait what?  
    i: server block: d  
    i: listen 8081: s  
    i: root and index: ss  
    i: location block: s  
restart Nginx to update the news  
    `sudo service nginx restart`  
our homepage is running on `http://ip-adress_vm`  
the url `http://ip-address_vm/abc` returns Error 404  
everything as is should be
