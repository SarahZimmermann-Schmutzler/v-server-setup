# The Login Process

1. [From Root to User](#from-root-to-user)
   * [Create a User](#create-a-user)
1. [The secure Login](#the-secure-login)
   * [Create a SSH-Key for your local server](#create-a-ssh-key-for-your-local-server)
   * [Store the SSH-Key on your VM](#store-the-ssh-key-on-your-vm)
   * [Disable Password Login](#disable-password-login)
   * [Simplify Connection Setup](#simplify-connection-setup)
     * [Alias the SSH Connection](#alias-the-ssh-connection)
     * [SSH Config for Several Identities](#ssh-config-for-several-identities)

## From Root to User

Almost every new VM works in such a way that the host (cloud provider) gives the user **root access** to the VM directly.  
You must **log in as root with the default password** because there are no other users initially. The password is usually replaced with your own password the first time you log in.  
  
According to the saying `Don’t be root unless you really, really have to`, it is safer to create a user even for personal use.

* Root logins are the first target of SSH brute-force attacks. In an emergency, attackers can immediately take over your system. Instead, disable SSH login for root unless it is otherwise circumvented.

* One wrong command as root can destroy your entire system. Better work with the normal user + sudo.

* You can't properly monitor who did what if multiple people use the same root account.

### Create a User

1. **Log in** in to the VM with the **root user account** via SSH:

    ```bash
    ssh root@ip-address_vm
    ```

1. **Create a user** with a name, password and, if necessary, additional information:

    ```bash
    adduser user_vm
    ```

1. Add the user to the **sudo group**:

    ```bash
    usermod -aG sudo user_vm
    ```

1. Test if it worked by opening a **new terminal** and establishing an **SSH connection** to the VM with your **new user**. The **password** you entered when creating the user will be used as the password.

    ```bash
    ssh user_vm@ip-address_vm
    ```

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

### Disable Password Login

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

> [!Note]
> Password login is now disabled for all users, including root. Root can only log in with an SSH-Key if one has been stored for them. If this is the case, **PermitRootLogin** should also be disabled in the `sshd_config` file because **logging in with root**, as [already written above](#from-root-to-user), represents a **security risk**.

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
