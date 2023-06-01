
followed instructions in [fracpete/rpi-remote-access](https://github.com/fracpete/rpi-remote-access) repository

## Server
AWS EC2 instance with Amazon Linux used as server

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