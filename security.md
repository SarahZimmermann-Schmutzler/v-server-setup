# Further Security Precautions

1. [Change the SSH-Port](#change-the-ssh-port)
1. [Activate Firewall ufw](#activate-firewall-ufw)
1. [Install Fail2Ban](#install-fail2ban)

## Change the SSH-Port

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

* Connection with **SSH-Client `config` file** like shown [here](login.md#ssh-config-for-several-identities):

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

## Activate Firewall ufw

It is likely that the Ubuntu-VM has the program `ufw` pre-installed.

* `ufw` stands for **Uncomplicated Firewall**. It helps you manage a firewall on a Linux system.

1. At the beginning it is **inactive**. You can test this with:

    ```bash
    ufw status
    ```

1. It is important that the **ports are opened before activation** - all that are needed individually, e.g.:

    ```bash
    sudo ufw allow 2222/tcp # new SSH port
    sudo ufw allow 80/tcp # http
    sudo ufw allow 443/tcp #https
    ```

1. Enable the firewall:

    ```bash
    sudo ufw enable
    ```

1. Check if the activation worked:

    ```bash
    sudo ufw status verbose
    ```

1. Before closing the current terminal, open a new one and check if connection via SSH works.

## Install Fail2Ban

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

  * **maxretry**: Number of failed attempts within the findtime before IP is banned (5)
  * **findtime**: Period within which failed attempts are counted (10 minutes)
  * **bantime**: How long the IP remains blocked (10 minutes)

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
