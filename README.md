# HOW TO SET UP A VM  

This guide was created as part of my **DevSecOps training** at the Developer Academy.  

## Table of Contents

1. [What exactly is a VM?](#what-exactly-is-a-vm)
1. [The Login](#the-login)
   * [Create a SSH-Key for your local server](#create-a-ssh-key-for-your-local-server)
   * [Store the SSH-Key on your VM](#store-the-ssh-key-on-your-vm)
   * [Deactivate the possibility to login with a password](#deactivate-the-possebility-to-login-with-a-password)
   * [Simplify Connection Setup](#simplify-connection-setup)
     * [Alias the SSH connection](#alias-the-ssh-connection)
     * [SSH config for several identities](#ssh-config-for-several-identities)
1. [The Web Server - Nginx](#the-web-server---nginx)
   * [Install and Activate Nginx](#install-and-activate-nginx)
   * [Configurate Nginx](#configurate-nginx)

## What exactly is a VM?

A **VM**, **Virtual Machine** or **Virtual Server** is a software program that runs on a bare-metal server and emulates it.

* It can run an **operating system and applications** as if they were running on real hardware.  
* A software environment called **Hypervisor** creates and manages the VMs.

## The Login

### Create a SSH-Key for your local server

**SSH (Secure Shell)** is a protocol that allows secure access to a remote computer over an insecure network.

* An **SSH-Key (Secure Shell Key)** is a cryptographic key pair used to authenticate SSH connections SSH-Keys are more secure than password-based authentication because they use strong encryption.
  * Once set up, you no longer have to enter a password to connect to the server. 
  * They are also well suited for automated processes and scripts that need to access remote servers in a secure manner.

* It consists of a **private key and a public key**.  
  * The **private key** should be kept secure and secret. It is saved on the client (your computer) and is used to authenticate to a server.  
  * The **public key** can be passed on without hesitation or stored on the server you want to log in to. It is used to encrypt messages that can only be decrypted with the corresponding private key.

1. Open the program **Git Bash** as admin on your local platform, e.g. Windows.  
  
1. Create a **ED25519 SSH-Key pair** (more secure than RSA SSH-Key pair):

    ```bash
    ssh-keygen -t ed25519  
    # choose to use a password or not
    ```

> [!NOTE]
> It's possible to use the same key for different VMs. If the key is compromised, all connected VMs are affected. It's also possible to generate a separate key pair for each VM, which can then be managed via `~/.ssh/config` for convenience.

### Store the SSH-Key on your VM

**Logging into a VM** doesn't require a connection via an SSH-Key. You can connect **using a username and password combination**. This approach presents a **security vulnerability**, as any password can potentially be brute-forced.
  
Therefore, it is recommended to **store the SSH-Key on the VM** so that you can login with it instead of a password.

1. **Copy and transfer** your public SSH-Key on the VM:

    ```bash
    ssh-copy-id -i path/to/your/id_ed25519.pub user_vm@ip-address_vm
    ```
  
* The public SSH-Key is **now stored** in `~/.ssh/authorized_keys`

#### **If this fails**

* The SSH-key can also be copied and added manually to the VM:
  * **Copy** SSH-Key:

    ```bash
    cat ~/.ssh/id_ed25519.pub
    ```

  * **Log in** to the VM **with username and password** combination:

    ```bash
    ssh user_vm@ip-address_vm
    ```

  * On a freshly installed VM, there is usually no `authorized_keys` file, and often there is no `.ssh/` directory in the home folder - **create** them:

    ```bash
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    nano ~/.ssh/authorized_keys
    ```

  * **Paste the copied public key** into a new line:

    ```bash
    chmod 600 ~/.ssh/authorized_keys
     ```

2. After you logged out from your VM with `logout` or `exit`, you can now **log in** with the usual `ssh user_vm@ip-address_vm` but **without password query**.

> [!CAUTION]
> **Make sure your SSH-Key access is working** before proceeding to the next step!

### Deactivate the possebility to login with a password

Even if you now log in using an SSH-key, the **password login is still active** – and this is a potential **security risk**:

* If no SSH-Key is found when attempting an SSH connection, you will automatically be asked for the password. An attacker can therefore log in using the username-password combination after guessing it!

To prevent that you have to **deactivate the `PasswordAuthentication` option** in the VM's `sshd_config`.

1. **Open the `sshd_config` file** with an editor:

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```
  
1. **Activate** the option **`PasswordAuthentication`** (remove `#`) and change it from `yes` to **`no`**.  
  
1. **Restart the SSH-Service**.  

    ```bash
    sudo systemctl restart ssh.service
    ```

1. **Check** if the password authentication was **successfully issued**:

* **Log out of the VM**:

    ```bash
    logout
    ```

* Try to **log in with password authetication by explicitly disabling public key authentication**:  

    ```bash
    ssh -o PubkeyAuthentication=no user_vm@ip-address_vm
    ```

* You should get the information:  
`user_vm@ip-address_vm: Permission denied (pubkey)`

### Simplify Connection Setup

The command for *establishing an SSH connection** can be quite cumbersome. This can be **remedied** by using a `shell alias` or registering the host in the `config file`.

#### Alias the SSH connection

A **shell alias** is an abbreviation or alternative name for a longer command or sequence of commands in the shell. They help commonly used commands run more efficiently and quickly by associating them with shorter or easier-to-remember names.  
  
By default, aliases are only available for the duration of the current shell session. To make them **persistent**, you need to define them in one of your **shell configuration files** on your local server:

1. **Open or create the `bash_profile` script** that runs whenever you start a new shell session:  

    ```bash
    nano ~/.bash_profile
    ```

1. **Add the following if it's not already listed** and save the file:  

    ```bash
    if [ -f ~/.bashrc ]; then
            . ~/.bashrc
    fi
    ```

* **Reload** the file:  

    ```bash
    source ~/.bash_profile
    ```
  
1. **Open or create the `bashrc`** - the config file for the bash which is, thanks to the bash_profile script, executed every time a terminal is opened:

    ```bash
    nano ~/.bashrc
    ```

1. Add your alias to the document and save it, e.g. vm_connection:

    ```bash
    alias vm_connection="ssh user_vm@ip-address_vm"
    ```
  
1. **Reload** the file:  

    ```bash
    source ~/.bashrc
    ```

1. After ending the current bash with `logout` or `exit`, **use the alias to log in to the VM**:

    ```bash
    vm_connection
    ```

#### SSH config for several identities  

On the local server there is a **SSH-Client** or **-Agent** installed that is executed every time a connection to a server is opened.
You can **register hosts (VMs) or identities** in its configuration file - also with different SSH-Keys.

1. Open the **SSH-Client `config` file** or create one if there is none

    ```bash
    nano ~/.ssh/config
    ```
  
1. Add the **following information**:  

    ```bash
    Host vm1
         HostName ip-address_vm
         User user_vm
         PreferredAuthentications publickey
         IdentityFile ~/.ssh/id_ed25519
    ```
  
1. Connect to the VM just with the **defined host profile name**.  

    ```console
    ssh vm1
    ```

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

### Configurate Nginx

If you now browse the IP-address you are welcomed by the Nginx homepage.  
  
Here is the procedure if you want to be **greeted by your own homepage**:

1. The default homepage is stored under `/var/www/html/index.nginx-debian.html`.  
To get our own homepage we create the folder *alternatives* under the path:  
```console
sudo mkdir /var/www/alternatives
```
  
2. And add an alternate index.html.  
```console
sudo mkdir /var/www/alternatives/alternate-index.html
```
  
Maybe it looks like this:  
```html
<!doctype html>
<html>
  <head>
	<meta charset="utf-8">
	<title>Hello, Nginx!</title>
  </head>

  <body>
	<h1>Hello, I am an alternative Homepage for Nginx</h1>
	<p>Noice to see ya :)</p>
	<p>You can meet me at IP-address: 116.203.104.65</p>
  </body>
</html>
```
3. Now we want the alternate index to be loaded by Nginx. Create a Nginx config file for this purpose.  
```console
sudo nano /etc/nginx/sites-enabled/alternatives
```
> i: By default there is only the default config file that loads the current homepage: /etc/nginx/sites-enabled/default  
  
4. Create a server- and a location-block    
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
Wait what?  
> i: server block: Regulates that incoming HTTP-Requests to our IP-address are handled by the Nginx web server.  
  
> i: listen 8081: The web server now runs on IP-port 8081. The default port is 80 as you can see in the default config file.  
  
> i: root and index: Where is the root or where should Nginx look for the home page? The starting point for the web server should be the index alternate-index.html.  
  
> i: location block: Takes care of the paths that come after http://IP-address/.  
  
5. Restart Nginx to update the news.  
```console
sudo service nginx restart
```
  
Is the alternate homepage running on `http://ip-adress_vm`?  
Does the url: `http://ip-address_vm/abc` returns `404 Not Found`?  
Then everything is as it should be. Congrats!


Maybe you think `Cool, but I don't want to type every time :8081 as ending... Can I set the alternative index.html as an entry point? For example by forwarding the requests to the default port 80 to 8081?`  
Excellent question! Yes, you can!

#### Procedure  
1. Modify the default config file.  
```nginx
server {
        listen 80 default_server;
        listen [::]: 80 default_server;

        server_name _;

	#forwarding to port 8081 (alternatives)
        location / {
                proxy_pass http://localhoast:8081;
		proxy_set_header Host $host;
        	proxy_set_header X-Real-IP $remote_addr;
        	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        	proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```
2. Test if it worked out.  
```console
sudo nginx -t
```
  
3. If the Test was successful, restart Nginx.  
```console
sudo service nginx restart
```
  
4. Open the IP-address of the VM in the browser. The alternative homepage should open. If yes, well done! If not, ask ChatGPT for help, bye!  
<a href="http://116.203.104.65/" target="_blank">Alternate Nginx Homepage</a>
