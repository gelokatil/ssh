# sensor.ssh
Generic SSH based sensor and switch for Home Assistant

- (Only sensor) Tested and verified using Home Assistant 2021.7.4


Sensor:
Login to a remote server via ssh, execute a command and retrieve the result as sensor value

Switch:
Login to a remote server via ssh, execute specific remote commands to perform switch on, off, and status commands


To get started download
```
/custom_components/ssh/manifest.json
/custom_components/ssh/sensor.py
/custom_components/ssh/switch.py
```
into
```
<config directory>/custom_components/ssh/
```

**Example configuration.yaml:**

```yawl
sensor:
  - platform: ssh
    scan_interval: 3600
    host: 192.168.100.100
    port: 2222
    name: 'My Sensor Name'
    username: !secret device-username
    password: !secret device-password
    key: !secret REALLY-LONG-SSH-HASH
    command: "sensors | grep 'Package id 0:' | cut -c17-20"
    value_template: >-
        {%- set line = value.split("\r\n") -%}
        {{ line[1] }}
    unit_of_measurement: "ºC"


switch:
  - platform: ssh
    scan_interval: 3600
    host: 192.168.100.100
    port: 2222
    name: 'My Switch Name'
    username: !secret device-username
    password: !secret device-password
    key: !secret REALLY-LONG-SSH-HASH
    command_on: '/usr/local/sbin/remote-command'
    command_off: 'echo off'
    command_status: 'echo off'

NOTE: Above Switch example will always indicate off, since status command's return value will equal 'off'

```

NOTE: To obtain the ssh key, you can run the following command in a linux shell and use the rsa hash in the key parameter.  

```yawl
ssh-keyscan 192.168.1.10

192.168.1.10 ssh-rsa AAAAB3ZzaC1yc2EAAAADAQABAAABAQCvBLTLoH3eBXxgLj59XbMtJnOckGKBKgPy2+cVMIroDvX7uH3QUyL84fD43/7OmeNPzlwDfK83rqpLBTpuOiZaa6d5u4oFd0gDegLkOz3G7mg0g/+JLckqvHI8u2Xs3P1DZzUpx0y0cP7cjYeVxjLFDLKPU4HrXuRLBNqwtOFqVoEX8GuJPEXXFtJUkgEpllHC1ZJ+RwYRjj5KIHKondzT7pCDgMafCTPBlLZUHfV+HQenYiQusBlOHTMvydV4FOGVTPzGvrQS8OuO1mcT6lafsssMWOS5HbHqmEGqhkSNxpyf4t24k6IAP2xcOfsllPQ66J0a4pMdcB4DjC3ll28f
```


### Configuration Variables

**name**

  (string)(Optional) Friendly name of the sensor

**host**

  (string)(Required) The hostname or IP address of the remote server

**username**

  (string)(Required) A user on the remote server
  
**password**

(string)(Required) The password for the account

**port**

  (integer)(Optional) The port to ssh to
  Default value: 22

**key**

  (string)(Required) SSH remote host key to match.
  
**command**

  (string)(Required - Sensor Only) The command to execute on the remote server

**command_on**

  (string)(Required - Switch Only) The command to execute on the remote server when the switch is activated
  
**command_off**

  (string)(Required - Switch Only) The command to execute on the remote server when the switch is deactivated

**command_status**

  (string)(Required - Switch Only) The command to execute on the remote server to determine switch on/off status
  
### Considerations

The sensor utilizes a paramiko client, a python implementation of the ssh v2 library. Documentation can found at http://docs.paramiko.org/en/stable/api/client.html

A sensor value can only by a maximum of 256 characters in length, therefore make use of the value_template to parse data larger than this.

Credits to @ejonesnospam and @jchasey