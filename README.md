# How to configure a WebDAV server using rclone


### Introduction 
[WebDAV](http://www.webdav.org/) is an extension of the `HTTP` protocol that allows you share files remotely. Most web server software such as `Apache HTTP`have built-in `WebDAV` modules but they usually require complex configuration steps.

One of the easiest ways to configure a `WebDAV` server is by using `rclone`.

[Rclone](https://rclone.org/) is a command line application used to manage files on cloud storage, but it also be used to configure a `WebDAV` server.

### Step 1
First, you'll need to install `rclone`.
~~~
sudo dnf install rclone -y
~~~
Here is the format for the command to run a`WebDAV` server:
~~~
rclone serve webdav remote:path [flags]
~~~
* The `remote:path` specifies the directory for the WebDAV server.
~~~
/media/webdav/
~~~
Configure the `addr`, `user`, and `pass` flags.

* The `addr` flag is used to specify the IP address and port.
~~~
--addr 192.168.4.200:8111
~~~
* The `user` flag is used to specify a username. Choose a name that doesn't exist on the server.
~~~
--user poochie
~~~
* The `pass` flag is the password for the `WebDAV` user.
~~~
--pass myRandomPassword
~~~
Here is the complete command to start the `WebDAV` server:
~~~
rclone serve webdav /media/webdav/ --addr 192.168.4.200:8111 --user poochie --pass myRandomPassword
~~~
This works great for testing but should be configured to run as a service.

## Step 2

We'll be creating a custom `systemd` unit file to start and stop the `WebDAV` server as a service.

First, we need to create a `bash` script that will run the `rclone serve` command.
~~~
sudo vi /usr/bin/rclone/rclone_webdav.sh
~~~
Here is the complete bash script:
~~~
#!/bin/bash
/usr/bin/rclone serve webdav /media/webdav/ --addr 192.168.4.200:8111 --user poochie --pass myRandomPassword
~~~
Make the script executable.
~~~
chmod +X /usr/bin/rclone/rclone_webdav.sh
~~~
Create a systemd unit file for the script.
~~~
vi /etc/systemd/system/rclone_webdav.service
~~~
Add the following lines:
~~~
[Unit]
Description=Rclone WebDAV Service
After=network.target

[Service]
ExecStart=/usr/bin/rclone/rclone_webdav.sh
Restart=always

[Install]
WantedBy=multi-user.target
~~~
Run the following `systemctl` commands to enable the service:
~~~
systemctl daemon-reload
systemctl start rclone_webdav.service
systemctl enable rclone_webdav.service
~~~
To verify the WebDAV server is running:
~~~
systemctl status rclone_webdav.service
~~~
