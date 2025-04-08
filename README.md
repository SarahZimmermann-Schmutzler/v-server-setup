# HOW TO SET UP AN UBUNTU-VM  

This guide explains **how to set up and secure a VM including login hardening, firewall activation, and web server installation**.  
  
It was created as part of my **DevSecOps training** at the Developer Akademie and expanded in my own practical work.  

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

* [The Login Process](./login.md)

2. This is followed by **options to secure the VM against attacks**:

* [Further Security Precautions](./security.md)

3. To access the VM via the web browser, a **web server** is required to process the requests. You will learn how to set it up in the third part of the setup:

* [Configurate a Web Server](./nginx.md)
