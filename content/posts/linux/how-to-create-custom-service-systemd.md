---
title: "How to Create Custom Service Systemd on Linux"
date: 2023-02-16T10:18:49+07:00
draft: false
---

## Step 1:

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
