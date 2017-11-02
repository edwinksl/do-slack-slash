# How to Set Up Slack Slash Command on Ubuntu 16.04

### Introduction

In this guide, you will to set up a [Slack](https://slack.com/) slash command on a Ubuntu 16.04 server using a [Flask](http://flask.pocoo.org/) app. This Flask app is served by a [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) application server and a [Nginx](https://nginx.org/) server that acts as a reverse proxy. This slash command can then be invoked from any Slack workspace in which you install the slash command as part of a Slack app. For API documentation about Slack slash commands, visit https://api.slack.com/slash-commands.

## Prerequisites

Before you begin this tutorial, you will need the following:

1. Set up a non-root user with sudo privileges on a Ubuntu 16.04 server: [Initial Server Setup with Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).
2. Set up uWSGI and Nginx: [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04).

## Step 1 — Create and Install Slack App

We will first create a Slack app and install it in a development Slack workspace. If you do not have a development Slack workspace, create one at https://slack.com/create. To create a Slack app, visit https://api.slack.com/apps and click on the green **Create New App** button. Under "App Name", we will use "DigitalOcean Slack slash command". Select the appropriate workspace under "Development Slack Workspace" and then click on the green **Create App** button. After the app is created, click on **Slash Commands** and then the **Create New Command** button. You should see the following page:

![Page for creating new command.](https://i.imgur.com/78vbP8z.png)

For this tutorial, the Slack slash command that will be used is `/slash`, which will send data via HTTP POST to a request URL that is `http://<^>server_domain_or_IP<^>/slash`. Therefore, we will fill in the following information:

- **Command**: `/slash`
- **Request URL**: `http://<^>server_domain_or_IP<^>/slash`
- **Short Description**: DigitalOcean Slack slash command

The filled-in page now looks like:

![Filled-in page for creating new command.](https://i.imgur.com/5NEcWb1.png)

<$>[note]
**Note**: Ignore the "This doesn't seem like a proper link. Sorry!" message in the screenshot; you will not see this message as you are using a valid domain name or IP address.
<$>

Click on the green **Save** button to finish creating the slash command. Now, we will install the app by clicking on **Install App**, followed by the green **Install App to Workspace** button and lastly the green **Authorize** button.

## Step 2 — Create Flask App for Slack Slash Command

After finishing the [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04) tutorial, your Flask app resides in `~/<^>myproject<^>/` and this directory contains the following files and directory:
  - `<^>myproject<^>.py`
  - `wsgi.py`
  - `<^>myprojectenv<^>/`

Our Flask app in `<^>myproject<^>.py` acts on the data sent by the Slack slash command and returns a JSON response. As stated in https://api.slack.com/slash-commands, we should validate the slash command using the verification token, which should be private, provided by Slack. The verification token can be obtained by visiting https://api.slack.com/apps, clicking on our **DigitalOcean Slack slash command** app followed by **Basic Information**, and looking for **Verification Token**. We will save the verification token in a separate `.env` file that is private and not kept under version control, and then use the `python-dotenv` package to export the key-value pairs in `.env` as environment variables to be used in `<^>myproject<^>.py`.

We first activate the Python virtual environment by running:

```command
source <^>myprojectenv<^>/bin/activate
```

To confirm that the virtualenv is activated, you should see `(<^>myprojectenv<^>)` on the left-hand side of the Bash prompt. Using `pip`, we will install the `python-dotenv` package:

```command
pip install python-dotenv
```

Using nano or your favorite text editor, create the `.env` file:

```command
nano .env
```

Ensure that the `.env` file contains the following key-value pair:

```
[label ~/myproject/.env]
VERIFICATION_TOKEN=<^>your_verification_token<^>
```

We now open `<^>myproject<^>.py`:

```command
nano <^>myproject<^>.py
```

We modify `<^>myproject<^>.py` to respond to a Slack slash command by sending a text message that says "DigitalOcean Slack slash command is successful!":

```python
[label ~/myproject/myproject.py]
#!/usr/bin/env python

import dotenv
import os
from flask import Flask, jsonify, request

app = Flask(__name__)

dotenv_path = os.path.join(os.path.dirname(__file__), '.env')
dotenv.load_dotenv(dotenv_path)
verification_token = os.environ['VERIFICATION_TOKEN']


@app.route('/slash', methods=['POST'])
def slash():
    if request.form['token'] == verification_token:
        payload = {'text': 'DigitalOcean Slack slash command is successful!'}
        return jsonify(payload)


if __name__ == '__main__':
    app.run()
```

Because our request URL is `http://<^>server_domain_or_IP<^>/slash`, we need to change `location` in `/etc/nginx/sites-available/<^>myproject<^>` from `/` to `/slash`. We first open `/etc/nginx/sites-available/<^>myproject<^>`:

```command
sudo nano /etc/nginx/sites-available/<^>myproject<^>
```

Ensure that `location` is changed to `/slash` in `/etc/nginx/sites-available/<^>myproject<^>`:

```nginx
[label /etc/nginx/sites-available/myproject]
server {
    listen 80;
    server_name <^>server_domain_or_IP<^>;

    location /slash {
        include uwsgi_params;
        uwsgi_pass unix:/home/<^>sammy<^>/<^>myproject<^>/<^>myproject<^>.sock;
    }
}
```

After making this change, we check the Nginx configuration file for syntax errors:
```command
sudo nginx -t
```

If there are no syntax errors with the Nginx configuration file, we restart the Nginx service:

```command
sudo systemctl restart nginx
```

## Step 3 — Test Slack Slash Command

Visit your development Slack workspace and type `/slash` in any channel. You should see the following response:

![Slack slash command is successful!](https://i.imgur.com/TiHgJer.png)

## Conclusion

In this tutorial, you have implemented a Slack slash command by setting up a Flask app that is served by a uWSGI application server and a Nginx reverse proxy server. To encrypt the connection for the slash command, you should look into using HTTPS for the request URL. You can do so by installing on the Nginx server a SSL certificate issued for free by Let's Encrypt, and an awesome tutorial for this can be found at https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04.
