# Assignment3p2

### Task1

Make two new virtual servers (droplets):
- Make the droplets from digital ocean.
- Give each droplet the tag; web.
- Name the first droplet server1.
- Name the second droplet server2.

**Server1**

Go to Digital Ocean and click the green `Create` button at the top of the page.

Once you click it you should see the option called `Droplets`, click it.

Go through the set up for the droplet, with the following configuration steps;

- `Choose Region` = San Francisco
- `Datacenter` = SFO3
- `Choose an image` = Custom images, choose the arch linux image
- `Choose Size` = Basic
- `CPU options` = Regular, and 4/mo or 6/mo, either one is fine
- `Choose Authentication Method` = select your SSH key
- `Quantity` = 1 Droplet
- `Hostname` = server1
- `Tags` = web

Click `Create Droplet`.

**Server2**

Go through the set up for the droplet, with the following configuration steps;

- `Choose Region` = San Francisco
- `Datacenter` = SFO3
- `Choose an image` = Custom images, choose the arch linux image
- `Choose Size` = Basic
- `CPU options` = Regular, and 4/mo or 6/mo, either one is fine
- `Choose Authentication Method` = select your SSH key
- `Quantity` = 1 Droplet
- `Hostname` = server2
- `Tags` = web

Click `Create Droplet`.

### Task2

#### Create a Load balancer

1. Go to Digital Ocean
2. Go to the Create button at the top.
3. Click on load balancer.

**Load Balancer**

The majority of settings should remain as default.

Load balancer type should be Regional.

Datacenter should be San Francisco 3.

Network visibility should be external and public.

Under **Connect Droplets**, enter the tag `web`.

This should match the tag you gave each droplet.

Once you do every droplet with that tag will be added to the load balancer.

Leave the rest of the settings as the defaults and click `Create Load Balancer`.

> It will take some time to set up the load balancer.

### Task3

**Setup environment for server1 and server2**

Open your terminal.

Login into server1 using SSH.

```code
ssh -i .ssh/ssh-key arch@droplet-ip
```

`.ssh/ssh-key`, is the path to your private SSH key.

`droplet-ip`, is the ip address of server1.

Open another tab in your terminal and login into server2.

```code
ssh -i .ssh/ssh-key arch@droplet-ip
```

`.ssh/ssh-key`, is the path to your private SSH key.

`droplet-ip`, is the ip address of server2.

**Inside server1 and server2;**

- Update the serevr.
- Download vim.
- Download git.
- Download tree.
- Download nginx.

Update the server.

```bash
sudo pacman -Syu
```

Download vim.

```bash
sudo pacman -S vim
```

Download git.

```bash
sudo pacman -S git
```

Download tree.

```bash
sudo pacman -S tree
```

Download nginx.

```bash
sudo pacman -S nginx
```

Run the following command to get the startup files.

```bash
git clone https://git.sr.ht/~nathan_climbs/2420-as3-p2-start
```

### Task4

**Complete the steps bellow for both server1 and server2**

#### Step1

Create a new system user with the following:
- Home directory located at `var/lib/webgen`.
- System user login shell.
- Sub directories bin and HTML.
- The generate_index file inside `webgen/bin`.
- The index.html file inside `webgen/HTML`.
- The files, file-one and file-two inside `webgen/documents`.
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

The following command will create three sub directories called `bin`, `HTML` and `documents` inside the webgen directory.

```bash
sudo mkdir -p /var/lib/webgen/{bin,HTML,documents}
```

To move the `generate_index` file inside the `webgen/bin` directory, run the following.

```bash
sudo mv generate_index /var/lib/webgen/bin
```

Now put an `index.html` file inside the HTML directory.

```bash
sudo touch /var/lib/webgen/HTML/index.html
```

Now put `file-one` and `file-two` inside `documents` directory.

```bash
sudo touch /var/lib/webgen/documents/{file-one,file-two}
```

**Inside `file-one` and `file-two` add text.**

```bash
sudo vim /var/lib/webgen/documents/file-one
```
Add the following text.

```code
This is file-one from server1.
```

In server2;

```code
This is file-one from server2.
```

**Now for file-two.**

```bash
sudo vim /var/lib/webgen/documents/file-two
```
Add the following text.

```code
This is file-two from server1.
```
In server2;

```code
This is file-two from server2.
```

To give ownership of the webgen directory and any files or sub directories inside it to webgen, run;

```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

Run tree command to make sure your files follow the same directory tree.

```bash
tree /var/lib/webgen/
```

```bash
/var/lib/webgen/
├── bin
│   └── generate_index
├── documents
│   ├── file-one
│   └── file-two
└── HTML
    └── index.html
```

#### Step2

Make service and timer unit files:
- Service file that runs the generate_index script located at `/var/lib/webgen/bin/generate_index`.
- Timer unit file that runs the service file.
- The files will be put into the `/etc/systemd/system` directory.
- The files are named; `generate-index.service` and `generate-index.timer`.
- The timer file needs to activate the service at 5am everday.

cd into `/etc/systemd/system`.

```bash
cd /etc/systemd/system
```

run `sudo touch {generate-index.service,generate-index.timer}`.

```bash
sudo touch {generate-index.service,generate-index.timer}
```

#### Service file:

```bash
sudo vim generate-index.service
```

Put the following configuration inside.

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

```bash
sudo vim generate-index.timer
```

Put the following configuration inside.

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
sudo vim /etc/nginx/nginx.conf
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
sudo vim /etc/nginx/sites-available/webgen.conf
```

```bash
server {
        listen 80;
        listen [::]:80;

        server_name 144.126.213.143;

        root  /var/lib/webgen/HTML;
        index index.html;

        location / {

         try_files $uri $uri/ =404;

}

 location /documents {
        root /var/lib/webgen;
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
    }

}
```

`location /documents`, allows us to access the documents folder.

To make the site available we make a symlink that is stored in `sites-enabled`.

```bash
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/webgen.conf
```

To generate our html page run the `generate-index.serivce` file.

First we should make it executable.

```bash
sudo chmod +x /var/lib/webgen/bin/generate_index
```

Now we run it.

```bash
sudo systemctl start generate-index
```

Run `sudo nginx -t` to see if the `nginx.conf` file is running properly.

```bash
sudo nginx -t
```

Start the service

```bash
sudo systemctl start nginx
```

>Reload nginx if it does not run after start.
>
>Reload the service
>
```bash
sudo systemctl reload nginx
```

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