---
title: "My First Post"
date: 2023-02-16T05:42:43+07:00
draft: true
---

# Python Cheatsheet

### Init

1. `sudo apt install python3-pip`
2. `python3 -m venv env`
3. `source bin/activate`

#### requirements.tx

```python
# create
pip3 freeze --local > requirements.txt

# load
pip3 install -r requirements.txt
```

---

## Snippets:

#### SSH - Paramiko

```python
import re
import paramiko
import time

# \ndisplay ip interface brief | no-more
iplist = ['172.16.0.21', '172.16.0.25', '172.16.0.22']

for host in iplist:
    cmd = 'display int desc | no-more\ndisplay ip int br | no-more'
    promt = 'N'
    ssh_client = paramiko.SSHClient()
    ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh_client.connect(hostname=host, username='username_here', password='password_here', port='2135',look_for_keys=False)
    stdin, stdout, stderr = ssh_client.exec_command(cmd)
    stdin.write(promt + '\n')
    out_byte = stdout.read()
    out_str = out_byte.decode()

```

#### SSH - Netmiko - send_command (Read)

more example https://github.com/ktbyers/netmiko/blob/develop/EXAMPLES.md

```python
from netmiko import ConnectHandler
from getpass import getpass
from pprint import pprint

# device type list: https://github.com/ktbyers/netmiko/blob/develop/PLATFORMS.md
cisco1 = {
    "device_type": "cisco_ios",
    "host": "cisco1.lasthop.io",
    "username": "pyclass",
    "password": getpass(),
}

command = "show ip int brief"
with ConnectHandler(**cisco1) as net_connect:
    # Use TextFSM to retrieve structured data
    output = net_connect.send_command(command, use_textfsm=True)

print()
pprint(output)
print()
```

#### SSH - Netmiko - send_config_set (Write)

more example https://github.com/ktbyers/netmiko/blob/develop/EXAMPLES.md

```python
config_commands = [ 'logging buffered 20000',
                    'logging buffered 20010',
                    'no logging console' ]
output = net_connect.send_config_set(config_commands)
print(output)
```

#### SSH - Netmiko expect_string

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
    "device_type": "cisco_ios",
    "host": "cisco1.lasthop.io",
    "username": "pyclass",
    "password": getpass(),
}

command = "del flash:/test4.txt"
net_connect = ConnectHandler(**cisco1)

# CLI Interaction is as follows:
# cisco1#delete flash:/testb.txt
# Delete filename [testb.txt]?
# Delete flash:/testb.txt? [confirm]y

# Use 'send_command' and the 'expect_string' argument (note, expect_string uses
# RegEx patterns). Netmiko will move-on to the next command when the
# 'expect_string' is detected.

# strip_prompt=False and strip_command=False make the output
# easier to read in this context.
output = net_connect.send_command(
    command_string=command,
    expect_string=r"Delete filename",
    strip_prompt=False,
    strip_command=False
)
output += net_connect.send_command(
    command_string="\n",
    expect_string=r"confirm",
    strip_prompt=False,
    strip_command=False
)
output += net_connect.send_command(
    command_string="y",
    expect_string=r"#",
    strip_prompt=False,
    strip_command=False
)
net_connect.disconnect()

print()
print(output)
print()
```

#### SSH - Netmiko send_command_timing (Handle Prompt)

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
    "device_type": "cisco_ios",
    "host": "cisco1.lasthop.io",
    "username": "pyclass",
    "password": getpass(),
}

command = "del flash:/test3.txt"
net_connect = ConnectHandler(**cisco1)

# CLI Interaction is as follows:
# cisco1#delete flash:/testb.txt
# Delete filename [testb.txt]?
# Delete flash:/testb.txt? [confirm]y

# Use 'send_command_timing' which is entirely delay based.
# strip_prompt=False and strip_command=False make the output
# easier to read in this context.
output = net_connect.send_command_timing(
    command_string=command,
    strip_prompt=False,
    strip_command=False
)
if "Delete filename" in output:
    output += net_connect.send_command_timing(
        command_string="\n",
        strip_prompt=False,
        strip_command=False
    )
if "confirm" in output:
    output += net_connect.send_command_timing(
        command_string="y",
        strip_prompt=False,
        strip_command=False
    )
net_connect.disconnect()

print()
print(output)
print()
```

#### Regex

```python
import re
string_example = "this is string"
patern = (
		r'(.*\d{1,100}#.*)'
	)
xx = re.findall(patern, string_example)

```

#### NTC / TextFSM

[[ntc_template textfsm]]
https://github.com/ramadlana/ntc-templates

```bash
ntc-templates
Unparsed data:
Interface                      Status         Protocol Description
Vl1                            admin down     down
Vl99                           up             up       10.20.99.0_Switch_mgmt_VLAN
Gi0/1                          down           down     D3 USER
Gi0/2                          down           down     D3 USER
Gi0/3                          down           down     D3 USER
Gi0/4                          down           down     D3 USER
Gi0/5                          down           down     D3 USER
Gi0/6                          down           down     D3 USER
Gi0/7                          down           down     D3 USER
Gi0/8                          up             up       MERAKI TEST AP
Gi0/9                          admin down     down
Gi0/10                         up             up       UPLINK TO TULCCD3S01P
Gi0/10.10                      deleted        down
Gi0/10.20                      up             up       Carrier VLAN

TextFsm:
Value INTERFACE (\S+)
Value STATUS (up|down|admin down|deleted)
Value PROTOCOL (\S+)
Value DESCRIPTION (\S.*?)

Begin
  ^Interface\.*$$ -> Start

Start
  ^Interface.*$$
  ^${INTERFACE}\s+${STATUS}\s+${PROTOCOL}\s+$$ -> Record
  ^${INTERFACE}\s+${STATUS}\s+${PROTOCOL}$$ -> Record
  ^${INTERFACE}\s+${STATUS}\s+${PROTOCOL}\s+${DESCRIPTION}$$ -> Record
  ^. -> Error



https://pyneng.readthedocs.io/en/latest/book/21_textfsm/textfsm_examples.html

```
