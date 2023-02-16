---
title: "How to Create Custom Service Systemd on Linux"
date: 2023-02-16T10:18:49+07:00
draft: false
categories:
  - linux
tags:
  - systemd
  - linux
---

## Summary:

### Background

As developers, we often want to create an service system that can run on background and then launch a service every system startup. These sevice can be anything, maybe you have python scripts, bash scripts, or node js script, or maybe like this example we create a custome service for Metabase (https://www.metabase.com/start/).

Metabase can be launched from user terminal by typing `java -jar metabase.jar` but every user logout or close the terminal metabase will stopped immediately.

### Goal:

Our goal is simple we want to create services for Metabase then control it using systemd for example `systemctl start metabase.service`

## Step 1: Goto systemd folder

Find your user defined services. Ubuntu was at `/etc/systemd/system/`

## Step 2: Create .service files

Create a text file with your favorite text editor name it `whatever_you_want.service`

## Step 3: .service example

Put following **Template** to the file `whatever_you_want.service`

```sh
[Unit]
Description=Metabase Daemon

[Service]
PIDFile=/run/metabase.pid
WorkingDirectory=/usr/local/bin/metabase/
User=ubuntu
Group=ubuntu

#RUN COMMAND HERE
ExecStart=/usr/bin/java -jar -Xmx512m /usr/local/bin/metabase/metabase.jar

Restart=on-failure
RestartSec=30
PrivateTmp=true

StandardOutput=file:/var/log/sf-app.log
StandardError=file:/var/log/sf-app.err.log

[Install]
WantedBy=multi-user.target
```

## Explanation

1. Unit: Every service is called unit and give description with name for this example is Metabase Daemon
2. PID: process identifier, and location is on /run folder.
3. Working directory: Where your application folder is located
4. User & Group: user and group your operating system
5. ExecStart: Command to execute. You must specify full path to the executable. For example if our application is running using `java my_app.jar` you must specify `/usr/bin/java`. if you not sure where the executable is located you can run `which java` on terminal
6. Standart Output and Standart Error: For logging purposes every your application returned value to user and every your application facing errors
7. Multi-user.target: Normally defines a system state where all network services are started up and the system will accept logins, but a local GUI is not started.so for simplicity here the rule of thumb. Use multi-user.target if your application is not GUI and use graphical.target if your application is GUI
