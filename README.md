# HOW TO SET UP A VM  

This guide was created as part of my **DevSecOps training** at the Developer Academy.  

## Table of Contents

1. [What exactly is a VM?](#what-exactly-is-a-vm)
1. [The secure Login](#the-secure-login)
   * [Create a SSH-Key for your local server](#create-a-ssh-key-for-your-local-server)
   * [Store the SSH-Key on your VM](#store-the-ssh-key-on-your-vm)
   * [Deactivate the possibility to login with a password](#deactivate-the-possebility-to-login-with-a-password)
   * [Simplify Connection Setup](#simplify-connection-setup)
     * [Alias the SSH Connection](#alias-the-ssh-connection)
     * [SSH Config for Several Identities](#ssh-config-for-several-identities)
1. [Further Security Precautions](#further-security-precautions)
   * [Change the SSH-Port](#change-ssh-port)
   * [Install Fail2Ban](#install-fail2ban)
1. [The Web Server - Nginx](#the-web-server---nginx)
   * [Install and Activate Nginx](#install-and-activate-nginx)
   * [Configurate Nginx](#configurate-nginx)
     * [Create a Customzied Page and configure it](#create-a-customized-page-and-configure-it)
     * [Set the Alternate Page as Homepage](#set-the-alternate-page-as-homepage)

## What exactly is a VM?

A **VM**, **Virtual Machine** or **Virtual Server** is a software program that runs on a bare-metal server and emulates it.

* It can run an **operating system and applications** as if they were running on real hardware.  
* A software environment called **Hypervisor** creates and manages the VMs.

## The secure Login

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

1. **Copy and transfer** your public SSH-Key on the VM with the program `ssh-copy-id`:

    ```bash
    ssh-copy-id -i ~/.ssh/id_ed25519.pub user_vm@ip-address_vm
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

2. Do not close the session but open a new terminal and **log in** with the usual `ssh user_vm@ip-address_vm` but **without password query**.

> [!CAUTION]
> **Make sure your SSH-Key access is working** before closing the current session or proceeding to the next step!

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

* Open a new terminal and try to **log in with password authetication by explicitly disabling public key authentication**:  

    ```bash
    ssh -o PubkeyAuthentication=no user_vm@ip-address_vm
    ```

* You should get the information:  
`user_vm@ip-address_vm: Permission denied (pubkey)`

### Simplify Connection Setup

The command for *establishing an SSH connection** can be quite cumbersome. This can be **remedied** by using a `shell alias` or registering the host in the `config file`.

#### Alias the SSH Connection

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

#### SSH Config for Several Identities  

On the local server there is a **SSH-Client** or **-Agent** installed that is executed every time a connection to a server is opened.
You can **register hosts (VMs) and identities (SSH-Keys)** in its configuration file.

1. Open the **SSH-Client `config` file** or create one if there is none:

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

    ```bash
    ssh vm1
    ```

## Further Security Precautions

### Change the SSH-Port

The service **running by default on port 22** is the **SSH server (sshd)**. Whenever you specify ssh user@host without a port, port 22 is automatically used.  
  
The problem – everyone knows it. That's why bots, scanners, and attackers around the world systematically try IP addresses on port 22 – and attempt brute-force logins. Even if you have good SSH keys and passwords, you'll often see a ton of connection attempts in the log.  
  
While **changing the SSH port isn't a real security measure**, it does reduce the attack surface for simple bots and scanners. Logs stay quieter, and you're less likely to be attacked. In combination with real measures (SSH keys, firewall, fail-to-ban, etc.), it's a **useful addition**.

1. Activate and **change the port in the `sshd_config` file** on the VM from 22 to e.g. **2222**:

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

1. **Restart** the **SSH service**:

    ```bash
    sudo systemctl restart ssh.service
    ```

1. **Test the connection** via your local server in a new terminal:

* **Usual SSH connection**:

    ```bash
    ssh -p 2222 user_vm@ip-adress_vm
    ```

* Connection with **SSH-Client `config` file** like shown [here](#ssh-config-for-several-identities):

  * Add the **new portnumber to the configuration file**:

    ```bash
    nano ~/.ssh/config
    ```

    ```bash
    Host vm1
         HostName ip-address_vm
         User user_vm
         PreferredAuthentications publickey
         Port 2222
         IdentityFile ~/.ssh/id_ed25519
    ```

  * Connect to the VM with the **defined host profile name**.  

    ```bash
    ssh vm1
    ```

### Install Fail2Ban

`Fail2Ban` is a small security program that monitors your log files (e.g., SSH logins) and blocks IP  addresses temporarily or permanently that do suspicious things (e.g., enter the wrong password five times).

* Automatically blocks brute-force attacks.

* Reduces log clutter because of fewer pointless entries from bots.

1. **Install `Fail2Ban`** on the VM:

    ```bash
    sudo apt install fail2ban
    ```

1. Create a **customized configuration file** so that the **correct SSH port** is used:

* `Fail2Ban` first loads the user-specific `jail.local` file if it exists. If it doesn't exist or values ​​aren't defined here, the default `jail.conf` file is used.

    ```bash
    sudo nano /etc/fail2ban/jail.local 
    ```

    ```bash
    [sshd]
    enabled = true
    port = 2222
    maxretry = 5
    bantime = 10m
    findtime = 10m
    ```

  * maxretry: Number of failed attempts within the findtime before IP is banned (5)
  * findtime: Period within which failed attempts are counted (10 minutes)
  * bantime: How long the IP remains blocked (10 minutes)

1. **Restart `Fail2Ban`**:

    ```bash
    sudo systemctl restart fail2ban
    ```

1. To **check the status** use:

    ```bash
    sudo fail2ban-client status
    # or
    sudo fail2ban-client status sshd
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
