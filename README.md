
followed instructions in [fracpete/rpi-remote-access](https://github.com/fracpete/rpi-remote-access) repository

## Server
AWS EC2 instance with Amazon Linux used as server

**steps to configure EC2 instance**
- configure security group with the following rules
  - 22 for SSH (type)
  - 6000 and 7000 for Custom TCP (Type)
  - 6001, 6002... for more connected Raspberries. <br>
    \* use the i.e. 6001 port instead of 6000 in the further configuration below if configuring another RPi

access vit ssh:
```
ssh -i "<certificate>.cer" <user>@<...>.eu-central-1.compute.amazonaws.com
```
*the ssh command can be found in aws console (EC2 -> instances -> \*instance -> connect)*<br>
*the certificate is created when setting up EC2 instance*

**steps to install frp on server:**
- download the appropriate and latest release binary <br>
  in our case we used [v0.49.0](https://github.com/fatedier/frp/releases/tag/v0.49.0) release, more specifically [frp_0.49.0_linux_amd64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.49.0/frp_0.49.0_linux_amd64.tar.gz) binary
  ```
    sudo bash
    cd /opt
    wget https://github.com/fatedier/frp/releases/download/v0.49.0/frp_0.49.0_linux_amd64.tar.gz
    tar -xzf frp_0.49.0_linux_amd64.tar.gz
    ln -s frp_0.49.0_linux_amd64 frp
  ```
- create `/etc/frps.ini` with the following content:
  ```
    [common]
    bind_port = 7000
  ```
- Create systemd service `/etc/systemd/system/frps.service` with the following content:
  ```
    [Unit]
    Description=frp reverse proxy server
    After=network.target
    
    [Service]
    User=ec2-user
    Group=ec2-user
    WorkingDirectory=/opt/frp
    ExecStart=/opt/frp/frps -c /etc/frps.ini
    
    [Install]
    WantedBy=multi-user.target
  ```
- Install systemd service
  ```
    sudo systemctl enable /etc/systemd/system/frps.service
  ```
- Start service
  ```
    sudo systemctl start frps.service
  ```


## Raspberry Pi

- port 22 needs to be open
  - install frp
    - download appropriate [release binary](https://github.com/fatedier/frp/releases)<br>
      in our case [frp_0.49.0_linux_arm.tar.gz](https://github.com/fatedier/frp/releases/download/v0.49.0/frp_0.49.0_linux_arm.tar.gz)
      ```
        sudo bash
        cd /opt
        wget https://github.com/fatedier/frp/releases/download/v0.49.0/frp_0.49.0_linux_arm.tar.gz
        tar -xzf frp_0.49.0_linux_arm.tar.gz
        ln -s frp_0.49.0_linux_arm frp
      ```
    - Create `/etc/frpc.ini` with the following content:
      ```
        [common]
        server_addr = <address_of_ec2_instance>
        server_port = 7000
      
        [ssh_<device_name>]
        type = tcp
        local_ip = 127.0.0.1
        local_port = 22
        remote_port = 6000
      ```
      **Warning:** if multiple devices are connected to frp, [ssh] needs to be unique per device. Name it i.e. [ssh_cpc1] 
    - Create systemd service `/etc/systemd/system/frpc.service` with the following content:
      ```
        [Unit]
        Description=frp reverse proxy client
        After=network.target
        
        [Service]
        User=cpc
        Group=cpc
        Restart=on-failure
        RestartSec=15s
        WorkingDirectory=/opt/frp
        ExecStart=/opt/frp/frpc -c /etc/frpc.ini
        
        [Install]
        WantedBy=multi-user.target
      ```
      *User and Group are specific for each RPi (`whoami` and `groups` commands)*
    - Install systemd service
      ```
        sudo systemctl enable /etc/systemd/system/frpc.service
      ```
    - Start service
      ```
        sudo systemctl start frpc.service
      ```

## Raspberry Pi access
follow [fracpete/rpi-remote-access](https://github.com/fracpete/rpi-remote-access#raspberry-pi-access) repository instructions