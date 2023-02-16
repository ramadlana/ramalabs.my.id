---
title: "How to Create Custom Service Systemd on Linux"
date: 2023-02-16T10:18:49+07:00
draft: false
---

## Backgound:

As developers, we often want to create an service system that can run on background and then launch a service every system startup. These sevice can be anything, maybe you have python scripts, bash scripts, or node js script, or maybe like this example we create a custome service for Metabase (https://www.metabase.com/start/).

Metabase can be launched from user terminal by typing `java -jar metabase.jar` but every user logout or close the terminal metabase will stopped immediately.

## Goal:

Our goal is simple we want to create services for Metabase then control it using systemd for example `systemctl start metabase.service`

## Step 1: Goto systemd folder

Find your user defined services. Ubuntu was at `/etc/systemd/system/`

## Step 2:

Create a text file with your favorite text editor name it `whatever_you_want.service`

## Step 3:

Put following **Template** to the file `whatever_you_want.service`

```sh
[Unit]
Description=Metabase Daemon

[Service]
PIDFile=/var/tmp/metabase.pid
WorkingDirectory=/usr/local/bin/metabase/
User=ubuntu
Group=ubuntu

#RUN COMMAND HERE
ExecStart=/usr/bin/java -jar -Xmx512m /usr/local/bin/metabase/metabase.jar

Restart=on-failure
RestartSec=30
PrivateTmp=true

StandardOutput=metabase
StandardError=metabase_err
SyslogIdentifier=metabase

[Install]
WantedBy=multi-user.target
```

### Explanation

1. Unit: Every service is called unit and give description with name for this example is Metabase Daemon
