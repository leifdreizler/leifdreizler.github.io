---
title:  "Installing Cowrie (an SSH Honeypot) on Centos 7"
date:   2016-11-23 12:00:00
categories: [Honeypot]
tags: [CentOS 7, Cowrie]
---

As part of work for a future blog post I decided to install and monitor an SSH honeypot 🍯 . A honeypot is a decoy designed to attract and monitor hostile users. Honeypots can serve as a canary—allowing system administrators to improve defenses throughout their network, or be used as a way to study attackers.
<p align="center">
<img src="{{ "images/cowrie/hp.jpg" | prepend: site.baseurl }}" style="-webkit-filter: drop-shadow(3px 3px 3px #222); filter: drop-shadow(3px 3px 3px #222);">
</p>


It's powered by the following:

* [CentOS 7](https://www.centos.org/){:target="_blank"}
* [Cowrie](https://github.com/micheloosterhof/cowrie){:target="_blank"}

## Introduction ##

*"Cowrie is a medium interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker",* developed by Michel Oosterhof, and is an actively maintained fork of [Kippo](https://github.com/desaster/kippo){:target="_blank"}. 

[Cowrie's features](https://github.com/micheloosterhof/cowrie#user-content-features){:target="_blank"} include the ability to log an attacker's movement through a fake filesystem and save files that attack attempts to download.

I would strongly advise against running Cowrie on a server that is hosting things you care about. Cowrie could have it's own [security issues](https://github.com/micheloosterhof/cowrie/wiki/Frequently-Asked-Questions){:target="_blank"} and should be isolated from the rest of your environment. 

## References ##

* [My Baseline CentOS Setup](https://leifdreizler.com/2016/Base-Setup/){:target="_blank"}
* [How To Install Kippo, an SSH Honeypot, on an Ubuntu Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-install-kippo-an-ssh-honeypot-on-an-ubuntu-cloud-server){:target="_blank"}

## Getting Started ##

I'll assume you're starting with a CentOS 7 server that has firewalld enabled. If you don't have these configured you can follow my [previous guide](https://leifdreizler.com/2016/Base-Setup/){:target="_blank"} about setting up a CentOS 7 server in [DigitalOcean](https://m.do.co/c/d669cfd3f8d6){:target="_blank"}. Stop when you reach the "Let's Encrypt and Nginx" section.

If you have completed that section that's fine too—it won't impact the setup of the honeypot. However, I would strongly encourage you **not** to run a honeypot on the same host as things you care about, as mentioned above.

## Reconfiguring SSH and Firewalld ##

We'll need to move SSH to another port to make room for the honeypot. 

1. `sudo emacs /etc/ssh/sshd_config`
2. Uncomment and change `#Port 22` to `Port 222`
3. `sudo systemctl restart sshd`

In the future you will have to SSH into your server using `-p 222` appended, or edit your [SSH config file](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client){:target="_blank"} to specify a port for this host. We'll also have to make some changes to `firewalld`.

Cowrie is not supposed to run as root, so we'll need to redirect port 22 to a non-priveleged port. By default Cowrie runs on 2222, so we'll use that.

```
# Open up port for real SSH connection
$ sudo firewall-cmd --permanent --add-port=222/tcp
success
$ sudo firewall-cmd --zone=public --add-masquerade --permanent
success
$ sudo firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=2222 --permanent
success
$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces: 
  sources: 
  services: ssh
  ports: 222/tcp
  masquerade: yes
  forward-ports: port=22:proto=tcp:toport=2222:toaddr=
  icmp-blocks: 
  rich rules:
$ sudo firewall-cmd --reload
```

## Installing and Configuring Cowrie ##

You'll need to install a handful of dependcies and Python libraries to install and run Cowrie.

```
$ sudo yum -y upgrade
$ sudo yum install -y epel-release
$ sudo yum install -y gcc libffi-devel python-devel openssl-devel git python-pip pycrypto
$ sudo pip install configparser pyOpenSSL tftpy twisted==15.2.0 
$ sudo adduser cowrie
$ sudo passwd cowrie
$ su - cowrie
$ git clone https://github.com/micheloosterhof/cowrie.git
$ cd cowrie
$ mv cowrie.cfg.dist cowrie.cfg
```

Edit your `cowrie.cfg` file and uncomment `#listen_port = 2222`. In this file you will find tons of options, which you can explore on your own. 

#### Connecting to Cowrie ####

Start Cowrie and use `tail` to monitor the logfile. You can stop Cowrie buy running `./stop.sh`

```
./start.sh
Starting cowrie with extra arguments [] ...
$ tail -F log/cowrie.log
```

Open a local terminal tab, erase your server's SSH fingerprint, and then SSH to the honeypot using the username 'root' and any password except 'root' or '123456' as found in [/honeyfs/etc/passwd](https://github.com/micheloosterhof/cowrie/blob/master/honeyfs/etc/passwd){:target="_blank"} file. 

<pre class="highlight"><code># Replace with your server's hostname
local$ ssh-keygen -R <span style="color:red">138.68.62.198</span>
local$ ssh root@<span style="color:red">138.68.62.198</span>
</code></pre>

You can view any active connections by going back to the terminal tab running `tail`.

Any actions taken within the honeypot will be stored in the `log` directory for you to review in the future. I find the .log file easier to view in realtime, and the .json files easier to parse.

#### Conclusion ####

You should expect a few hundred to a few thousand login attempts per day, which creates plenty of chances to test the various configuration options of Cowrie on your uninvited guests. I highly encourage you to dive into the [Cowrie GitHub repo](https://github.com/micheloosterhof/cowrie){:target="_blank"} for an overview of additional features, and look at the codebase starting with the .cfg file. 
