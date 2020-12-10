---
title:  "Building this Site"
date:   2016-10-28 18:00:00
categories: [my site]
tags: [CentOS 7, NGINX, Let's Encrypt, Jekyll]
---

In my first blog post I'm going to walk you through the process of building this blog.
<p align="center">
<img src="{{ "images/building/meta.jpg" | prepend: site.baseurl }}">
</p>

It's powered by the following:

* [CentOS 7](https://www.centos.org/){:target="_blank"}
* [NGINX](https://www.nginx.com/){:target="_blank"}
* [Let's Encrypt](https://letsencrypt.org/){:target="_blank"}
* [Jekyll](https://jekyllrb.com/){:target="_blank"}

## Getting Started ##

I'm a huge fan of [DigitalOcean](https://www.digitalocean.com){:target="_blank"}, and use them to host a few projects including this site. If you don't have a Digital Ocean account, use my [referral code](https://m.do.co/c/d669cfd3f8d6){:target="_blank"} to get $10 🙃 

I have attempted to highlight differences in your <span style="color:red">variables, users, etc</span> in red, so watch out for those!

## Setting up CentOS ##

I have consolidated my CentOS setup process into [this blog post](https://leifdreizler.com/2016/Base-Setup/){:target="_blank"}. It covers setting up users, firewalld, automatic updates, Nginx, Let's Encrypt, etc. I'll assume you have completed that blog post, or are comfortable making adjustments as needed.

Later in the guide you'll need to access port 4000/tcp, so we'll open that up now.

```
$ sudo firewall-cmd --permanent --add-port=4000/tcp
success
$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces: 
  sources: 
  services: http https ssh
  ports: 4000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
$ sudo firewall-cmd --reload
```

## About Jekyll ##

Jekyll is written in Ruby and can easily turn blog posts written in markdown into a simple, yet powerful website. It's similar to [Hugo](https://gohugo.io/){:target="_blank"} or [ghost](https://ghost.org/){:target="_blank"}. Jekyll is used to power [GitHub Pages](https://pages.github.com/){:target="_blank"}, which can be used in lieu of a bunch of things I set up manually as part of this blog post—and is also free 🤑

Most themes follow the same basic architecture. The two most important parts of a theme are the `_config.yml` file and `_posts` folder. The config file lets you change some basic information about the site including your name, links to social media, and things like the markdown parser or syntax highlighter.

## Installing Jekyll ##
```
# some of these may be uneccessary, but can save you some headache later with gem installs
$ sudo yum install -y ruby ruby-devel rubygems build-essential git zlib-devel gcc
$ sudo gem install jekyll bundler
```

## Setting up Jekyll ##

Find a Jekyll theme online and go to its [GitHub page](https://github.com/joshgerdes/jekyll-uno){:target="_blank"} and follow the installation instructions. If none are provided, try adapting the directions in the theme I used. Deal with the inevitable dumpster fire involved with installing ruby gems 🔥

<pre class="highlight"><code># Navigate to the directory you want your site to live in, I chose ~/site
$ mkdir <span style="color:red">~/site</span> && cd <span style="color:red">~/site</span>
# You could also 'fork' the parent repo and then clone
$ git clone https://github.com/<span style="color:red">joshgerdes/jekyll-uno</span>.git
$ cd <span style="color:red">jekyll-uno</span>
$ bundle install
</code></pre>

Customizing your Jekyll install is mostly done within your `_config.yml` file. I adjusted title, description, url, baseurl and the social media links. You can see my [config file on GitHub](https://github.com/leifdreizler/leifdreizler.com/blob/master/_config.yml){:target="_blank"}.

<pre class="highlight"><code>$ bundle exec jekyll serve --host=<span style="color:red">138.68.44.75</span></code></pre>

Visit <span style="color:red">138.68.44.75</span>:4000 in your browser just to make sure that everything is running properly, then exit Jekyll with Ctrl+C.


📸	&nbsp; Take a snapshot once your Jekyll theme is working

## Reconfiguring nginx ##

We'll need to make some minor edits to our existing nginx configuration to accomodate the Jekyll site.

<pre class="highlight"><code>$ sudo usermod -a -G <span style="color:red">leif</span> nginx
$ chmod 710 /home/<span style="color:red">leif</span>
</code></pre>


When you edit `$ sudo emacs /etc/nginx/conf.d/ssl.conf`, either copy mine (below) or edit the following directives manually:

<pre class="highlight"><code>root /home/<span style="color:red">leif/site/jekyll-uno</span>/_site;
index index.html
autoindex off;
</code></pre>

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
	root /home/<span style="color:red">leif/site/jekyll-uno</span>/_site;
	index index.html
	autoindex off;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
}
</code></pre>

Check your nginx configuration for errors and restart:

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

## Finishing Touches ##

Confirm that you can access and navigate throughout your site by visiting https://<span style="color:red">leifdreizler.com</span>

```
# Remove the now unnecessary 4000/tcp port 
$ sudo firewall-cmd --permanent --remove-port=4000/tcp
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
```

You're all done! You now have a Jekyll blog with a custom theme served by NGINX running on CentOS 7 protected by a Let's Encrypt SSL certificate 🙌

📸	&nbsp; Take a snapshot, and optionally delete intermediate snapshots

## YOUR First Post ##

<pre class="highlight"><code>$ cd ~/<span style="color:red">site/jekyll-uno</span>/_posts
# follow the YYYY-MM-DD-name-of-post.md naming convention
$ cp <span style="color:red">existingpost.md newpost.md</span>
# Use your favorite text editor to adjust the contents, tags, etc.
$ emacs newpost.md
$ bundle exec jekyll build
</code></pre>

When writing blog posts you may want to consider previewing them using an online markdown parser before posting them to your site. You can also use `$ bundle exec jekyll build --watch` to automatically post blog updates whenever you save the markdown file. 
