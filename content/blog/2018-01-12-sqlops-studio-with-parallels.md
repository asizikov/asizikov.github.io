---
date: "2018-01-12T00:00:00Z"
image: /images/2018/01-sqlops/connected.png
title: Connecting SQL Operations Studio to SQL Express server in Parallels VM
tags: 
 - tools
slug: sqlops-studio-with-parallels
---

In this post, I'm going to show how to marry SQL Operations Studio running on macOS with MS SQL Express running on Windows VM in Parallels.

Even though SQL Ops Studio is not a fully mature project it's already sufficient enough to perform simple and quick actions. 

### Motivation

Since I'm running most of the apps on a host macOS and I'm trying to keep my windows VM as lean and possible, I think SQL Ops Studio is a good choice for most of my SQL related tasks. I still have to keep MS SQL Management Studio installed because I'm using RedGate tools such as SQL Source Control, but that's a different story.

### My system overview

First of all, let's check what I've got installed here: 

* MacOS host system. Running macOS High Sierra 10.13.2
* Parallels Desktop 13 Pro
* MS Windows 10 Pro Virtual Machine
* SQL Operations Studio 0.24.1
* SQL Express Server 2014


Windows VM is [configured](http://kb.parallels.com/en/112093) to have a static IP address. 

### Set up SQL Express to accept incoming connections

To set up SQL Express we need SQL Server Configuration Manager.

I'm running SQL Express 2014 so need to type `SQLServerManager12.msc`.

![Starting SQL Server Network Configuration Manager](/images/2018/01-sqlops/sql-manager.png)

Go to SQL Server Network Configuration -> Protocols for SQLEXPRESS -> TCP/IP

![SQL Server Network Configuration screenshot](/images/2018/01-sqlops/sql-manager-protocols.png)

Right click and select *Enable*: 

![Enable TCP/IP protocol](/images/2018/01-sqlops/sql-manager-tcp-enable.png)

Set up port number as on the picture (you're free to use any value, I'm using 5171 here)

![Set up tcp/ip port](/images/2018/01-sqlops/tcp-ip-port.png)

### Configure Windows Firewall

We must allow `sqlserver.exe` to accept inbound connections: 

![Windows Firewall, create new rule](/images/2018/01-sqlops/windows-firewall-rule.png)

### Kerberos

By this time you should be ready to connect to the SQL Server.

Connection string should look like this one: 

`tcp:<your-vm-ip>,5171, database-name`

In case your SQL EXPRESS instance is set up to use Windows Authentication most probably you would see an error like that: 

![Kerberos Error message](/images/2018/01-sqlops/kerberos-error.png)

There is a good article with step-by-step instructions. [Please refer to it](https://docs.microsoft.com/en-us/sql/sql-operations-studio/enable-kerberos).

You can always [switch to Mixed Authentication Mode](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/change-server-authentication-mode#SSMSProcedure) :) 


That's all folks!

![Connected](/images/2018/01-sqlops/connected.png)