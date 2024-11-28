# Assignment3p2

### Task1

Make two new virtual servers (droplets):
- Make the droplets from digital ocean.
- Give them each the tag web.
- Follow the configuration below for each droplet.

#### Step1

Create a new system user with the following:
- Home directory located at `var/lib/webgen`.
- System user login shell.
- Sub directories bin and HTML.
- The generate_index file inside `webgen/bin`.
- The index.html file inside `webgen/HTML`.
- Ownership of home directory and any files and sub directories within.

```bash
sudo useradd -r webgen -d /var/lib/webgen -s /usr/bin/nologin
```

`-r` : specifies that it is a system user.

`-d` : specifies the location of the home directory.

`-s` : specifies the shell of the user. System users usually have non-login shells.

Since the user's home directory does not exist yet, run the following command.

```bash
sudo mkdir -p /var/lib/webgen
```

`-p` : Makes directories within directories.

The following command will create two sub directories called `bin` and `HTML` inside the webgen directory.

```bash
sudo mkdir -p /var/lib/webgen/{bin,HTML}
```

To move the `generate_index` file inside the `webgen/bin` directory, run the following.

```bash
sudo mv generate_index /var/lib/webgen/bin
```

Now put an `index.html` file inside the HTML directory.

```bash
sudo touch /var/lib/webgen/HTML/index.html
```

To give ownership of the webgen directory and any files or sub directories inside it to webgen, run;

```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

#### Step2

Make service and timer unit files:
- Service file that runs the generate_index script located at `/var/lib/webgen/bin/generate_index`.
- Timer unit file that runs the service file.
- The files will be put into the `/etc/systemd/system` directory.
- The files are named; `generate-index.service` and `generate-index.timer`.
- The timer file needs to activate the service at 5am everday.

cd into `/etc/systemd/system`.

run `sudo touch {generate-index.service,generate-index.timer}`.

#### Service file:

```code
[Unit]
Description=Runs the generate_index script.
After=network-online.target
Requires=network-online.target

[Service]
User=webgen
Group=webgen
Type=oneshot
ExecStart=/var/lib/webgen/bin/generate_index

[Install]
WantedBy=multi-user.target
```

`After` and `Requires` ensure that the serivce will only run after `network-online.target` is already running.

`ExecStart` is the location of the script the service needs to run.

#### Timer file:

```code
[Unit]
Description=Run generate-index.service at 5am daily

[Timer]
OnCalendar=*-*-* 5:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

To active the timer run `sudo systemctl start generate-index.timer`.

To see if the timer will execute the service file, run `systemctl status generate-index.timer`.

#### Step3

Modfiy the main `nginx.conf` file:
- To make the server run as `webgen`.
- Make a seperate server block for nginx to serve `index.html` on port 80.

Delete the default config file.

```bash
sudo rm /etc/nginx/nginx.conf
```

To make a new one. Copy the following configuration.

**configuration:**

```bash
user http;
worker_processes auto;
worker_cpu_affinity auto;

events {
    multi_accept on;
    worker_connections 1024;
}

http {
    charset utf-8;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    log_not_found off;
    types_hash_max_size 4096;
    client_max_body_size 16M;

    # MIME
    include mime.types;
    default_type application/octet-stream;

    # logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    # load configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

Now paste what you copied into the new file.

```bash
sudo nvim /etc/nginx/nginx.conf
```

To make the server run as webgen:

Edit the `/etc/nginx/nginx.conf` file from `user http;` to `user webgen;`.

```bash
user webgen;
```

Run the following command to make the two directories, `sites-available` and `sites-enabled`.

```bash
sudo mkdir -p /etc/nginx/{sites-available,sites-enabled}
```

Inside the `sites-available` directory, make a `webgen.conf` file.

```bash
sudo touch /etc/nginx/sites-available/webgen.conf
```

In the `webgen.conf` file put the following.

```bash
server {
        listen 80;
        listen [::]:80;

        server_name 146.190.124.7;

        root  /var/lib/webgen/HTML;
        index index.html;

        location / {

         try_files $uri $uri/ =404;

}

}
```

To make the site available we make a symlink that is tored in `sites-enabled`.

```bash
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/webgen.conf
```

Run `sudo nginx -t` to see if the `nginx.conf` file is running properly.

Start the service

```bash
sudo systemctl start nginx
```

Reload the service

```bash
sudo systemctl reload nginx
```
Now start the service again.

Why is it important to use a separate server block file instead of modifying the main 
nginx.conf file directly?

We can configure multiple sites using several server blocks.

How can you check the status of the nginx services and test your nginx configuration?

Run `sudo nginx -t` to see if the `nginx.conf` file is running properly.

#### Step4

Set up a firewall with ufw:
- Install ufw.
- Allow ssh and http.
- Enable ssh rate limiting.
- Check firewall status.

Install the ufw firewall package with;

```bash
sudo pacman -S ufw
```

The firewall is by default configured to deny everthing coming in, so we will allow `http` and `ssh`.

**Allow SSH**

```bash
sudo ufw allow ssh
```

We also want to limit ssh, so that when too many attempts are made in a short period of time the firewall will block them.

```bash
sudo ufw limit ssh
```

**Allow HTTP**

```bash
sudo ufw allow http
```

**Enable the firewall**

Check the status of the firewall to make sure that all the configurations block and allow as intended.

```bash
sudo ufw status verbose
```

Enable the firewall.

```bash
sudo ufw enable
```

### Task2

### Task3

### Task4