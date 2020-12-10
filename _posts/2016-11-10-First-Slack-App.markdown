---
title:  "Writing your first Slack /slash Command, Powered by Flask"
date:   2016-11-21 10:00:00
categories: [Slack App]
tags: [CentOS 7, NGINX, Let's Encrypt, Gunicorn, Flask, Slack]
---

This guide will walk you through setting up a very basic Slack application powered by Flask. It will use the [SMMRY API](http://smmry.com/){:target="_blank"} to take an article, summarize it and return text back into Slack. 
<p align="center">
<img src="{{ "images/slack/slack.png" | prepend: site.baseurl }}" style="-webkit-filter: drop-shadow(3px 3px 3px #222); filter: drop-shadow(3px 3px 3px #222);">
</p>

It's powered by the following:

* [CentOS 7](https://www.centos.org/){:target="_blank"}
* [NGINX](https://www.nginx.com/){:target="_blank"}
* [Let's Encrypt](https://letsencrypt.org/){:target="_blank"}
* [Gunicorn](http://gunicorn.org/){:target="_blank"}
* [Flask](http://flask.pocoo.org/){:target="_blank"}
* [Slack](https://slack.com/){:target="_blank"}


## Flask Setup ##

If you're using something like [Heroku](https://www.heroku.com/){:target="_blank"}, skip down to Slack [Slack Setup](#slack-setup).

This guide will assume you're starting with my [baseline CentOS setup](https://leifdreizler.com/2016/Base-Setup/){:target="_blank"}. 

Later in the guide you'll need to access ports 5000/tcp and 8000/tcp, so we'll open that up now.

```
$ sudo firewall-cmd --permanent --add-port=5000/tcp
success
$ sudo firewall-cmd --permanent --add-port=8000/tcp
success
$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces: 
  sources: 
  services: http https ssh
  ports: 5000/tcp 8000/tcp
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
$ sudo firewall-cmd --reload
```

Next, complete this [How To Serve Flask Applications with Gunicorn and Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-centos-7){:target="_blank"} from Digitial Ocean. The only difference is that under the "Configuring Nginx to Proxy Requests" you will need to edit `/etc/nginx/conf.d/ssl.conf` instead of `/etc/nginx/nginx.conf`.

Here is my `ssl.conf` file for reference:

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

        location / {
            proxy_set_header Host $http_host;   
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://unix:/home/<span style="color:red">leif/slack/summary.sock</span>;
    }
}
</code></pre>

## Slack Setup ##

[This guide](http://www.programmableweb.com/news/how-to-use-slack-api-to-build-slash-commands-powered-google-app-engine-and-go/how-to/2015/09/16){:target="_blank"} does a good job of walking you through the basics of registering a Slack /slash command. When you get to the section "Creating your Project in Google Cloud" come back to this guide.

## Flask and Slack Setup ##

While testing the application, I run the flask application on port 5000 as you saw in the Digital Ocean guide, with debug mode on. When testing using this method you will need to go back into your Slack configuration and change the URL which your /slash command POSTs to.

<p align="center">
<img src="{{ "images/slack/slack2.png" | prepend: site.baseurl }}">
</p>

Here is the initial Flask application, replace the `host=` with your server's IP address!

``` python
from flask import Flask
application = Flask(__name__)

@application.route("/", methods=['POST'])
def hello():
    print request.get_data()
    return Response("test", status=200)

if __name__ == "__main__":
    application.run(host='138.68.17.99', debug=True)
```

Run your Flask app by typing /summarize into slack. You should recieve a "test" message in Slack, and your server should recieve a chunk of information about the message you just sent, as well as your Slack team.

<pre class="highlight"><code>$ python <span style="color:red">summary.py</span>
 * Running on http://<span style="color:red">138.68.17.99</span>:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 827-866-054
token=[Secret Token]&amp;team_id=[Team ID]&amp;team_domain=[Team name]&amp;channel_id=C[Channel ID]&amp;channel_name=[Channel Name]&amp;user_id=[User ID]&amp;user_name=[User Name]&amp;command=%2Fsummarize&amp;text=&amp;response_url=[Response URL]
[Slack IP Address] - - [04/Nov/2016 13:44:48] "POST / HTTP/1.1" 200 -
</code></pre>

<p align="center">
<img src="{{ "images/slack/slack1.png" | prepend: site.baseurl }}">
</p>


## Slack and Flask Basics ##

#### Slack ####
Now that you've got Flask and Slack talking to each other, let's cover some basics. [This page](https://api.slack.com/slash-commands){:target="_blank"} covers some very useful information regarding Slack slash commands, and I highly recommend that you read through it. Here are some highlights:

* Slack stops waiting for a response after 3 seconds
* If you need longer than 3s you should reply with a 200 and reply later
* Data sent to your server will have a `content-type` of `application/x-www-form-urlencoded`
* Data sent back to Slack should have a `content-type` of `application/json`
* Message Formatting and Attachments
* "In Channel" vs. "Ephemeral" Messages

Most of this will through in the example, but I highly suggest reading the **How do commands work?** section.

#### Flask ####

Best practice dictates that if you have secret keys, you should set them as environment variables instead of leaving them in your Python files. This helps keep them out of places like GitHub. For the testing on 5000 you can simply export them to your `profile`. 

```
$ source summaryenv/bin/activate
$ export SMMRY_API_KEY=123456
```

To make them to be active when running outside of testing, you'll need to add them to the service you created using [How To Serve Flask Applications with Gunicorn and Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-centos-7){:target="_blank"}. There is an extended explanation in the comments section (thanks [jellingwood](https://www.digitalocean.com/community/users/jellingwood) for this help!)

1. `sudo emacs /etc/systemd/system/myproject.service`
2. Add a new line under the [Service] Section
3. `Environment="SMRRY_API_KEY=123456"
4. `sudo systemctl daemon-reload`
5. `sudo systemctl restart myproject.service`

To retrieve environment variables, use the following code snippet as an example:

``` python
try:
    api_key = os.environ['SMMRY_API_KEY']
except KeyError:
    print "SMMRY API key not set, please register at: http://smmry.com/partner and set SMMRY_API_KEY environment var"
```

## Communicating between Flask and Slack ##

To access fields sent from Slack, you will need to know the names of the fields that you're being sent. You printed these out earlier with `request.get_data()`. To access a specific field use: request.form.get('fieldname').

To send data back to Slack, format it using following pattern (some advanced examples are in [this article](https://www.viget.com/articles/how-to-build-your-own-slack-app-and-bot){:target="_blank"}).

Below is a basic example of returning a reply to yourself in Slack.

``` python
from flask import Flask, request, Response, jsonify
import os
import json

application = Flask(__name__)

try:
    api_key = os.environ['SMMRY_API_KEY']
    slack_token = os.environ['SLACK_TOKEN']
except KeyError:
    print "SMMRY API key not set, please register at: http://smmry.com/partner and set SMMRY_API_KEY environment var"

@application.route("/", methods=['POST'])
def parse_request():
    # Check to see if your team's token is the same, if not ignore the request
    if request.form.get('token') == slack_token:
      data = {
        "text": "It's 80 degrees right now.",
        "attachments": [
        {
          "text":"Partly cloudy today and tomorrow"
        }
        ]
      }

      return Response(response=json.dumps(data), status=200, mimetype="application/json")

    return 

if __name__ == "__main__":
    application.run(host='138.68.17.99', debug=True) 

```

## SMMRY API Call ##

This section of code will go over how to make a call to the [SMRRY API](http://smmry.com/api){:target="_blank"}, and format it in a way that will look good in Slack.

``` python
import json
import urllib2

api_url = 'http://api.smmry.com/'
try:
    api_key = os.environ['SMMRY_API_KEY']
except KeyError:
    print "SMMRY API key not set, please register at: http://smmry.com/partner and set SMMRY_API_KEY environment var"


def call_smmry():
    api_url = 'http://api.smmry.com/'
    num_sentences = '3'
    # article to summarize, later we will replace this with input from slack                                                                           
    article = "https://krebsonsecurity.com/2016/10/hackers-hit-u-s-senate-gop-committee/"
    url = api_url + "&SM_API_KEY=" + api_key + "&SM_LENGTH=" + num_sentences + "&SM_URL=" + article
    # use the requests library to call the SMMRY API                                                                                                   
    r = requests.post(url)
    # retrieve 'text' field from SMMRY response                                                                                                        
    # loading it as JSON allows you to retrieve fields as a python dictionary                                                                          
    summy = json.loads(r.text)

    # format message for slack                                                                                                                         
    # *bold* \n creates a newline                                                                                                                      
    data = {
    # this will display the message to all users in the channel instead of just you
    "response_type": "in_channel",
            "text":"*"+ summy["sm_api_title"]+"* \n" + summy["sm_api_content"]
        }
    # temporarily print the data just to verify it is correct                                                                                          
    print data

    return

call_smmry()
```

## Slightly Less Basic communication between Flask and Slack ##

You can use all of the normal [Slack formatting](https://api.slack.com/docs/message-formatting){:target="_blank"} when replying to Slack.

## Tying it all Together ##

This section of code will combine the previous code snippets, and explain how they work together.

```python
from flask import Flask, request, Response, jsonify
import os
import json

application = Flask(__name__)

try:
    api_key = os.environ['SMMRY_API_KEY']
    slack_token = os.environ['SLACK_TOKEN']
except KeyError:
    print "SMMRY API key not set, please register at: http://smmry.com/partner and set SMMRY_API_KEY environment var"

@application.route("/", methods=['POST'])
def parse_request():
  # get form data to pass to SMMRY
  request_data = request.form

  # need to reply w/ a 200 to slack, and then wait for SMMRY to parse 
  # and then do a 2nd response if you take longer than 3s to reply
  threading.Thread(target=real_response, args=[request_data]).start()

  return Response("", status=200)

    
def real_response(request_data):
  api_url = 'http://api.smmry.com/'
  num_sentences = '3' 
  # retrieve the message body from Slack
  article = request_data.get('text')
  url = api_url + "&SM_API_KEY=" + api_key + "&SM_LENGTH=" + num_sentences + "&SM_URL=" + article
  # use the requests library to call the SMMRY API  
  r = requests.post(url)
  # retrieve 'text' field from SMMRY response                                                                                                        
  # loading it as JSON allows you to retrieve fields as a python dictionary     
  summy = json.loads(r.text)

  # format message for slack                                                                                                                         
  # *bold* \n creates a newline 
  data = {
  # this will display the message to all users in the channel instead of just you
    "response_type": "in_channel",
      "text":"*"+ summy["sm_api_title"]+"* \n" + summy["sm_api_content"]
  }

  # use unquote to remove URL encoding from the URL provided by SLack
  response_url = urllib2.unquote(request_data.get('response_url'))
  header = {"content-type":"application/json"}
  requests.post(response_url, headers=header, data=json.dumps(data))
  return


if __name__ == "__main__":
    application.run(host='138.68.17.99', debug=True) 
```

You should now be able to type `/summarize link` and have get a 3 sentence summary of the article in Slack!

## Finishing Up ##

Remove `debug=true` from your Flask app and close your `firewalld` ports back up and you're all done!

```
# Remove the now unnecessary 5000 and 8000 ports
$ sudo firewall-cmd --permanent --remove-port=5000/tcp
success
$ sudo firewall-cmd --permanent --remove-port=8000/tcp
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

## What's next? ##

This was meant to be a small, but realistic example of how to create a /slash command in Slack. It covers some basic Python libraries in the process but isn't really meant to be production ready. It lacks even basic error handling and won't scale using the threading method I employed. 

If you're interested in continuing your work with Flask and Slack, [this guide](https://realpython.com/blog/python/getting-started-with-the-slack-api-using-python-and-flask/){:target="_blank"} would be a good next step. 

If you're interested in adding a production ready summarization tool to your Slack team, you should check out [CyMetica](http://cymetica.com/slack/sumbot/){:target="_blank"}. I haven't used it, but it looks like it has the desired functionality.
