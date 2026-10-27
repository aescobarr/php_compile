# How to download and compile php from source

## Download
```
# Download and extract
wget https://www.php.net/distributions/php-8.5.12.tar.gz
tar zxvf php-8.5.12.tar.gz
```
## Build 
```
# Build
cd php-8.5.12
./buildconf --force
# Will install to directory /opt/php-8.5.12
# Add extensions curl, zlib, openssl
./configure \
	--prefix=/opt/php-8.5.12 \
	--with-config-file-path=/opt/php-8.5.12/etc \
	--with-config-file-scan-dir=/opt/php-8.5.12/etc/conf.d \
	--enable-cli \
	--enable-fpm \
	--with-zlib \
	--with-curl \
	--with-openssl \
	--with-mysqli=mysqlnd \
	--with-pdo-mysql=mysqlnd \
	--with-fpm-user=www-data \
	--with-fpm-group=www-data
make -j"$(nproc)"
# Install
sudo make install
```

## Set default config
```
sudo mkdir -p /opt/php-8.5.12/etc/conf.d
sudo cp php.ini-production /opt/php-8.5.12/etc/php.ini
```

# Configure php-fpm

## Create default config
```
cd /opt/php-8.5.12/etc
sudo cp php-fpm.conf.default php-fpm.conf
sudo cp php-fpm.d/www.conf.default php-fpm.d/www.conf
```

## Adjust default config

Add content to www.conf:
```
[www8_5_12]
user = www-data
group = www-data

listen = /run/php-8.5.12.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

# Configure PHP-FPM to run in the foreground

## Create service config file
```
# Test config file
sudo /opt/php-8.5.12/sbin/php-fpm \
    --fpm-config /opt/php-8.5.12/etc/php-fpm.conf \
    --test
```
    
```
# Create service definition
sudo nano /etc/systemd/system/php-8.5.12-fpm.service
```

Add content to file:
```
[Unit]
Description=Custom PHP-FPM 8.5.12 (/opt/php-8.5.12)
After=network.target

[Service]
Type=simple

ExecStart=/opt/php-8.5.12/sbin/php-fpm --nodaemonize --fpm-config /opt/php-8.5.12/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 $MAINPID
KillSignal=SIGQUIT

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

## Operate service

```
# Reload daemons to register config
sudo systemctl daemon-reload
```

```
# Enable
sudo systemctl enable php-8.5.12-fpm
# Start
sudo systemctl start php-8.5.12-fpm
# Status
sudo systemctl status php-8.5.12-fpm
```

## Verify socket

```
ls -l /run/php-8.5.12.sock
```

The location of the socket can be found at /opt/php-8.5.12/etc/php-fpm.d at the appropiate named config file, at the line:
```
listen = /run/php-8.5.12.sock
```