# FileBrowser Installation and Configuration on Fedora

This guide covers the installation, configuration, and customization of FileBrowser on Fedora, along with setting up an Apache reverse proxy to serve it without redirection.

## 1. Install FileBrowser

### Clone the Repository

```bash
git clone https://github.com/PRASSamin/filebrowser
```

Build the Binary:

```sh
make build
```

Move the Binary to a Global Location:

```sh
sudo mv filebrowser /usr/local/bin/
```

## 2. Create a Systemd Service for FileBrowser

Create a systemd service file:

```sh
sudo nano /etc/systemd/system/filebrowser.service
```

Add the following content:

```ini
[Unit]
Description={{ Service Name }}
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/filebrowser -a 0.0.0.0 -p 9696 -r {{ Root Directory || /home/pras/FileBrowserRoot }} -d {{ Database Location || /home/filebrowser.db }}
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload Systemd and Enable the Service:

```sh
sudo systemctl daemon-reload
sudo systemctl enable filebrowser
sudo systemctl start filebrowser
```

Check the Service Status:

```sh
sudo systemctl status filebrowser
```

## Fix Permission Issues (If Needed)

If you encounter permission errors, adjust the file permissions:

```sh
sudo chown root:root /home/filebrowser.db
sudo chmod 644 /home/filebrowser.db

sudo chown -R root:root {{ Root Directory || /home/pras/FileBrowserRoot }}
sudo chmod -R 755 {{ Root Directory || /home/pras/FileBrowserRoot }}

sudo chown root:root /usr/local/bin/filebrowser
sudo chmod 755 /usr/local/bin/filebrowser
```

Restart the service:

```sh
sudo systemctl restart filebrowser
```

##### Handle SELinux (For Fedora)

SELinux might block FileBrowser on Fedora. To temporarily disable SELinux:

```sh
sudo setenforce 0
```

Restart the service:

```sh
sudo systemctl restart filebrowser
```

If it works after disabling SELinux, you can permanently allow FileBrowser:

```sh
sudo chcon -t bin_t /usr/local/bin/filebrowser
sudo chcon -t var_t /home/filebrowser.db
sudo chcon -t var_t /home/pras/FileBrowserRoot
```

## 4. Set Up Apache Reverse Proxy

### Install Required Modules

```sh
sudo dnf install httpd -y
```

Enable necessary modules by adding the following lines to `/etc/httpd/conf/httpd.conf`:

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

### Configure Virtual Host

Create a new configuration file:

```sh
sudo nano /etc/httpd/conf.d/filebrowser.conf
```

Add the following content:

```apache
<VirtualHost *:80>
    ServerName hostname

    ProxyPass "/" "http://127.0.0.1:9696/"
    ProxyPassReverse "/" "http://127.0.0.1:9696/"


    ErrorLog /var/log/httpd/filebrowser_error.log
</VirtualHost>
```

Save and restart Apache:

```sh
sudo systemctl restart httpd
```

## 5. Firewall Configuration

Allow HTTP traffic:

```sh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

If using SELinux, allow proxy connections:

```sh
sudo setsebool -P httpd_can_network_connect on
```

## 4. Set Up Apache Reverse Proxy

Install Required Modules

```sh
sudo dnf install httpd -y
```

##### Enable Necessary Modules

Edit the Apache configuration file to load the required proxy modules:

```sh
sudo nano /etc/httpd/conf/httpd.conf
```

Add the following lines:

```sh
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

##### Configure Virtual Host

Create a new configuration file for the virtual host:

```sh
sudo nano /etc/httpd/conf.d/filebrowser.conf
```

Add the following content:

```apache
<VirtualHost *:80>
    ServerName hostname

    ProxyPass "/" "http://127.0.0.1:9696/"
    ProxyPassReverse "/" "http://127.0.0.1:9696/"

    ErrorLog /var/log/httpd/filebrowser_error.log
</VirtualHost>
```

Save and Restart Apache

```sh
sudo systemctl restart httpd
```

## 5. Firewall Configuration

###### Allow HTTP traffic through the firewall:

```sh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

###### Allow Proxy Connections with SELinux (If Using SELinux)

```sh
sudo setsebool -P httpd_can_network_connect on
```
