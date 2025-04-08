# HOW TO SET UP AN UBUNTU-VM  

This guide explains **how to set up and secure an Ubuntu VM including login hardening, firewall activation, and web server installation**.  
  
It was created as part of my **DevSecOps training** at the Developer Academy and expanded in your own practical work.  

## Table of Contents

1. [What exactly is a VM?](#what-exactly-is-a-vm)
1. [The Setup](#the-setup)

## What exactly is a VM?

A **VM**, **Virtual Machine** or **Virtual Server** is a software program that runs on a bare-metal server and emulates it.

* It can run an **operating system and applications** as if they were running on real hardware.  
* A software environment called **Hypervisor** creates and manages the VMs.

The **following procedure** is aimed at the Linux operating system, more precisely the **Ubuntu distribution**.

## The Setup

1. The first part will cover **commissioning and secure login** to the VM:

* [From Root to User](./login.md#from-root-to-user)
  * [Create a User](./login.md#create-a-user)

* [The secure Login](./login.md#the-secure-login)
  * [Create a SSH-Key for your local server](./login.md#create-a-ssh-key-for-your-local-server)
  * [Store the SSH-Key on your VM](./login.md#store-the-ssh-key-on-your-vm)
  * [Disable Password Login](./login.md#disable-password-login)
  * [Simplify Connection Setup](./login.md#simplify-connection-setup)
  * [Alias the SSH Connection](./login.md#alias-the-ssh-connection)
  * [SSH Config for Several Identities](./login.md#ssh-config-for-several-identities)

1. This is followed by **options to secure the VM against attacks**:

* [Further Security Precautions](./security.md)
  * [Change the SSH-Port](./security.md#change-the-ssh-port)
  * [Activate Firewall ufw](./security.md#activate-firewall-ufw)
  * [Install Fail2Ban](./security.md#install-fail2ban)

1. To access the VM via the web browser, a **web server** is required to process the requests. You will learn how to set it up in the third part of the setup:

* [The Web Server - Nginx](./nginx.md#the-web-server---nginx)
  * [Install and Activate Nginx](./nginx.md#install-and-activate-nginx)
  * [Configure Nginx](./nginx.md#configure-nginx)
    * [Create a Customized Page and configure it](./nginx.md#create-a-customized-page-and-configure-it)
    * [Set the Alternate Page as Homepage](./nginx.md#set-the-alternate-page-as-homepage)
