---
title:  "Baseline CentOS Snapshot"
date:   2016-11-03 18:00:00
categories: [my site]
tags: [CentOS 7, NGINX, Let's Encrypt]
---

In the process of writing a few blog posts, I've realized that I use the same few Digital Ocean guides to get everything setup, and then make a few additional changes. Instead of repeating work across blog posts, and jumping back and forth between DO guides I'm going to walk you through creating a solid baseline DO snapshot which can be used to create new droplets in the future.
<p align="center">
<img src="{{ "images/centos/logo.png" | prepend: site.baseurl }}">
</p>

It's powered by the following:

* [CentOS 7](https://www.centos.org/){:target="_blank"}
* [NGINX](https://www.nginx.com/){:target="_blank"}
* [Let's Encrypt](https://letsencrypt.org/){:target="_blank"}

## Getting Started ##

I'm a huge fan of [DigitalOcean](https://www.digitalocean.com){:target="_blank"}, and use them to host a few projects including this site. If you don't have a Digital Ocean account, use my [referral code](https://m.do.co/c/d669cfd3f8d6){:target="_blank"} to get $10 🙃 

Part of why I love DigitalOcean is because of their extensive [community guides](https://www.digitalocean.com/community/tutorials/){:target="_blank"}. They cover a range of topics covering various permutations of operating systems, webservers, programming languages, etc.

At this point I'll assume you have already registered a domain, pointed the appropriate DNS records to your CentOS droplet's IP address, are at least generally familiar with linux, and can make minor adjustments to the guide based off differences in your environment.

I'm also a big fan of DO's 'snapshot' feature, which creates a point-in-time copy of your droplet which you can later restore from. I use it frequently when trying out new things as a way to restore to a previous working condition in case I mess something up.


## References ##

This guide is an attempt to walk you through my additions or changes to the DO CentOS setup process, but won't go into in depth explanations. If you need more context, please review:

* [Initial Server Setup with CentOS 7](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7){:target="_blank"}
* [Additional Recommended Steps for New CentOS 7 Servers](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-centos-7-servers){:target="_blank"}
* [How To Secure Nginx with Let's Encrypt on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7){:target="_blank"}

## Setting up and Securing CentOS ##

Throughout the guide:

```
# This is a comment about what I'm doing
$ This is a command to run
```

I have attempted to highlight differences in your <span style="color:red">variables, users, etc</span> in red, so watch out for those!

Log into your server using the ssh key you provided DO during the setup process. If you didn't setup a public key, do so at this time.

<code class="highlighter-rouge">local$ ssh <span style="color:red">root</span>@<span style="color:red">138.68.44.75</span></code> 

#### Users and SSH ####

<pre class="highlight"><code>$ sudo yum install -y epel-release 
# Leave out emacs if you prefer a different editor
$ sudo yum install -y ntp certbot nginx <span style="color:red">emacs</span>
$ adduser <span style="color:red">leif</span>
$ passwd <span style="color:red">leif</span>
$ gpasswd -a <span style="color:red">leif</span> wheel
$ su - <span style="color:red">leif</span>
$ mkdir .ssh
$ chmod 700 .ssh
$ touch .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
$ exit
$ cat .ssh/authorized_keys > /home/<span style="color:red">leif</span>/.ssh/authorized_keys 
</code></pre>

Next we'll make some changes to your SSH config to help protect your droplet 🇨🇳 &nbsp; 🇷🇺

1. Edit your ssh config `$ emacs /etc/ssh/sshd_config` 
2. Change `#PasswordAuthentication Yes` to `PasswordAuthentication No`
3. Change `#PermitRootLogin Yes` to `PermitRootLogin No`
4. Reload sshd: `$ systemctl reload sshd`

Open up a new terminal window on your local machine and test ssh: <code class="highlighter-rouge">$ ssh <span style="color:red">leif</span>@<span style="color:red">138.68.44.75</span></code>. If that works, close your root console tab and continue using the account you just created.

I highly recommend adding an [entry to your local SSH config](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client){:target="_blank"} to make logging into your server easier in the future.

#### Firewall ####

For `firewalld` you should enable http and https. SSH should already be enabled.

```
$ sudo systemctl start firewalld
$ sudo firewall-cmd --permanent --add-service=http
success
$ sudo firewall-cmd --permanent --add-service=https
success
$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces: 
  sources: 
  services: http https ssh
  ports: 
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
$ sudo firewall-cmd --reload
$ sudo systemctl enable firewalld
```

#### Timezones, NTP, Swap File, and Automatic Updates ####

<pre class="highlight"><code>$ sudo timedatectl list-timezones
$ sudo timedatectl set-timezone <span style="color:red">America/Los_Angeles</span>
$ sudo timedatectl
$ sudo systemctl start ntpd
$ sudo systemctl enable ntpd
$ sudo fallocate -l 1G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
$ sudo sh -c 'echo "/swapfile none swap sw 0 0" &gt;&gt; /etc/fstab'
</code></pre>

Next is to turn on automatic updates, which will help keep your site safe from the latest security threats.

```
# Install yum-cron
$ sudo yum install -y yum-cron
# Configure yum-cron (replace emacs with editor of your choice)
$ sudo emacs /etc/yum/yum-cron.conf
```
Edit `apply_updates = yes`

```
$ sudo systemctl enable yum-cron
$ sudo systemctl start yum-cron
```

The basic CentOS setup is complete, now we'll move onto Let's Encrypt and Nginx.

📸	&nbsp; If you're newer to linux, or just cautious you may want to take a snapshot at this point.

### Let's Encrypt and Nginx ###

<pre class="highlight"><code>$ echo -e "location ~ /.well-known {\n\tallow all;\n}" | sudo tee /etc/nginx/default.d/le-well-known.conf
$ sudo systemctl restart nginx
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
# Enter your email and agree to the Terms
$ sudo certbot certonly -a webroot --webroot-path=/usr/share/nginx/html -d <span style="color:red">leifdreizler.com</span> -d <span style="color:red">www.leifdreizler.com</span>
</code></pre>

`$ sudo emacs /etc/nginx/conf.d/ssl.conf` and paste in the following config:

<pre class="highlight"><code>server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        server_name <span style="color:red">leifdreizler.com www.leifdreizler.com</span>;

        ssl_certificate /etc/letsencrypt/live/<span style="color:red">leifdreizler.com</span>/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/<span style="color:red">leifdreizler.com</span>/privkey.pem;

        ssl_protocols TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
	ssl_session_tickets off;
        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;
	## verify chain of trust of OCSP response using Root CA and Intermediate certs
    	ssl_trusted_certificate /etc/letsencrypt/live/<span style="color:red">leifdreizler.com</span>/fullchain.pem;

	

        location ~ /.well-known {
                allow all;
        }

        # The rest of your server block
        root /usr/share/nginx/html;
        index index.html index.htm;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
}
</code></pre>

Setup a redirect to make sure all traffic goes over HTTPS:

```
$ echo -e "return 301 https://$host$request_uri;" | sudo tee /etc/nginx/default.d/ssl-redirect.conf
$ sudo nginx -t
$ sudo systemctl restart nginx
$ sudo systemctl enable nginx
$ sudo certbot renew
$ sudo crontab -e
```

Press `i` to enter edit mode and paste the following into crontab:

```
30 2 * * 1 /usr/bin/certbot renew >> /var/log/le-renew.log
35 2 * * 1 /usr/bin/systemctl reload nginx
```

Quit crontab using by typing: Esc `:wq` Enter

Visit <span style="color:red">leifdreizler.com</span>, if you see the default Nginx page, you're all done!

📸	&nbsp; Take a snapshot! This is a working config that can be reverted back to in the future. You can also spin up new droplets based off of this config instead of starting over when you want to do a new project.

I'll be going over some options in future blog posts, so stay tuned! &nbsp; 📺 
