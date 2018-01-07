# How to Set Up Slack Slash Command on Ubuntu 16.04

### Introduction
<!-- TODO:  Can you give an introduction to slash commands here? What are they and why would we want to build one? What kinds of things are they used for? Get us excited about this before we dive into the project. -->

<!-- Response: Added a quick introduction to slash commands below. -->

In [Slack](https://slack.com/), slash commands are a quick and easy way to perform actions in the message input box. For example, typing `/who` lists all users in the current channel. A complete list of built-in slash commands can be found at https://get.slack.help/hc/en-us/articles/201259356-Slash-commands. You will likely find that you need to add custom slash commands that members of your Slack workspace find useful, which is what we will cover in this tutorial.

<!-- TODO:  can you give an overview of how this all works at a high level? You type `/slash` into Slack. The request goes from Slack to your server, where your Flask application processes the request and returns a response to Slack" or something? -->

<!-- Response: Edited the paragraph below to incorporate your suggestions. -->

In this guide, you will set up a Slack slash command, which we will call `/slash`, on a Ubuntu 16.04 server using a [Flask](http://flask.pocoo.org/) app. This Flask app is served by a [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) application server and a [Nginx](https://nginx.org/) server that acts as a reverse proxy. By typing `/slash` in the message input box, the slash command can then be invoked from any Slack workspace in which you install the slash command as part of a Slack app. Invocation of the slash command sends information to the Flask app that processes the request and returns a response to Slack. For API documentation about Slack slash commands, visit https://api.slack.com/slash-commands.

## Prerequisites

To complete this tutorial, you will need:

* One Ubuntu 16.04 server set up by following [the Ubuntu 16.04 initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04), including a sudo non-root user and a firewall.
* An existing Flask application served with uWSGI running behind Nginx. Complete the tutorial [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04) to configure this on your server.
* A development Slack workspace. If you don't have one, create one at https://slack.com/create.

## Step 1 — Create and Install Slack App

We will first create a Slack app, which provides additional functionality to Slack, and install it in a development Slack workspace.  <!-- TODO:  explain what slack apps are? -->

<!-- Response: Slightly edited the paragraph above to state what Slack apps are used for. -->

To create a Slack app, visit https://api.slack.com/apps and click on the green **Create New App** button. Under **App Name**, we will use "DigitalOcean Slack slash command". Select the appropriate workspace under **Development Slack Workspace** and then click on the green **Create App** button. After the app is created, click on **Slash Commands** and then the **Create New Command** button. You should see the following page:

![Page for creating new command.](https://i.imgur.com/78vbP8z.png)

For this tutorial, the Slack slash command that will be used is `/slash`, which will send data via HTTP POST to a request URL that is `http://<^>server_domain_or_IP<^>/slash`. Therefore, we will fill in the following information:

- **Command**: `/slash`.
- **Request URL**: `http://<^>your_server_ip_or_domain_name<^>/slash`.
- **Short Description**: DigitalOcean Slack slash command.

    <!-- TODO:  I don't think the screenshot is necessary. Slack may change the UI and changing text later is easier than screenshots.  -->

    <!-- Response: I feel that the screenshot is useful for people who prefer not to read through a chunk of text to fill out a form. -->

The filled-in page now looks like:

![Filled-in page for creating new command.](https://i.imgur.com/5NEcWb1.png)

<$>[note]
**Note**: Ignore the "This doesn't seem like a proper link. Sorry!" message in the screenshot; you will not see this message as you are using a valid domain name or IP address.
<$>

Click on the green **Save** button to finish creating the slash command. Then install the app by clicking on **Install App**, followed by the green **Install App to Workspace** button and lastly the green **Authorize** button.

<!-- TODO:  Add a transition here. What are we going to do next? -->

<!-- Response: Added a transition paragraph below. -->

We have now created and installed a Slack app in the development Slack workspace. Next, we will create a Flask app that processes the slash command.

## Step 2 — Create Flask App for Slack Slash Command

<!-- TODO:  Introduce this step. What are we doing? "When we issue commands in our Slack workspace, they'll be sent to our server. Let's write the code that handles those requests by using the Flask application you created in the ....."  -->

<!-- Response: Added a quick introduction below to introduce this step. -->

We will now create a Flask app that will handle the infomation sent by the slash command and return a response to Slack.

After finishing the [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04) tutorial, your Flask app resides in `~/<^>myproject<^>/` and this directory contains the following files and directory:
  - `<^>myproject<^>.ini`
  - `<^>myproject<^>.py`
  - `wsgi.py`
  - `<^>myprojectenv<^>/`

Our Flask app in `<^>myproject<^>.py` acts on the data sent by the Slack slash command and returns a JSON response. As stated in https://api.slack.com/slash-commands, we should validate the slash command using the verification token, which should be private, provided by Slack. The verification token can be obtained by visiting https://api.slack.com/apps, clicking on our **DigitalOcean Slack slash command** app followed by **Basic Information**, and looking for **Verification Token**. We will save the verification token in a separate `.env` file that is private and not kept under version control, and then use the `python-dotenv` package to export the key-value pairs in `.env` as environment variables to be used in `<^>myproject<^>.py`.

First, activate the Python virtual environment by running:

```command
source <^>myprojectenv<^>/bin/activate
```

To confirm that the virtualenv is activated, you should see `(<^>myprojectenv<^>)` on the left-hand side of the Bash prompt. Secrets such as the verification token should not be stored under version control. To achieve this, we use the [`python-dotenv`](https://github.com/theskumar/python-dotenv) package that exports the secrets as environment variables. Using `pip`, we install the `python-dotenv` package:
<!-- TODO:  explain why we need this package. What will it do for us?  Also, link to its docs. -->

<!-- Response: Edited paragraph above to discuss what the package does and link to package documentation. -->

```command
pip install python-dotenv
```

Using nano or your favorite text editor, create the `.env` file:

```command
nano .env
```

Place the verification token in the `.env` file by ensuring that the file contains the following key-value pair: <!-- TODO:  be sure to explain what we are doing here and why. -->

<!-- Response: Edited sentence above to explain what is happening. -->

```
[label ~/myproject/.env]
VERIFICATION_TOKEN=<^>your_verification_token<^>
```

<!-- TODO:  before we open this file, let us know what we're going to do.  -->

<!-- Response: Added purpose of opening the file in the paragraph below. -->

During the development of a Flask app, it is sometimes convenient for the uWSGI server to reload automatically when we make changes to the app. To do this, first open `<^>myproject<^>.ini` in your editor:

```command
nano <^>myproject<^>.ini
```

Then, we modify `<^>myproject<^>.ini` to ensure that uWSGI automatically reloads when you `touch` or modify the Flask app in `<^>myproject<^>.py`: <!-- TODO:  why would we want to do this? Please explain. -->

<!-- Response: Added justification in two paragraphs above. -->

```python
[label ~/myproject/myproject.ini]
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true

touch-reload = <^>myproject<^>.py
```

<!-- TODO:  same here. What are we about to do? -->
We now open `<^>myproject<^>.py`:

```command
nano <^>myproject<^>.py
```

Modify `<^>myproject<^>.py` to respond to a Slack slash command by sending a text message that says "DigitalOcean Slack slash command is successful!":

<!-- TODO:  Can you clarify what's new here? You say modify the file, but do we add all this code? Or just parts of it?  -->

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

After we have modified and saved `<^>myproject<^>.py`, deactivate the Python virtual environment:

```command
deactivate
```

<!-- TODO:  why do we want to deactivate the env? Are we done? If we made a mistake and it doesn't work, we have to activate it again. -->

Restart the `<^>myproject<^>` systemd service:

```command
sudo systemctl restart myproject
```

<!-- TODO:  this is a fantastic example of telling us what we're doing before we do it. This is what I was looking for in the other spots.  -->
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

    location <^>/slash<^> {
        include uwsgi_params;
        uwsgi_pass unix:/home/<^>sammy<^>/<^>myproject<^>/<^>myproject<^>.sock;
    }
}
```

After making this change, check the Nginx configuration file for syntax errors:

```command
sudo nginx -t
```

If there are no syntax errors with the Nginx configuration file, restart the Nginx service:

```command
sudo systemctl restart nginx
```

<!-- TODO:  transition. What have we done so far, what's next? -->

## Step 3 — Test the Slack Slash Command

Visit your development Slack workspace and type `/slash` in any channel. You should see the following response:

![Slack slash command is successful!](https://i.imgur.com/TiHgJer.png)

<!-- TODO:  Is there more we can do in this tutorial to make this more useful? Maybe do something with a value that comes in? Because this step is so short, it could be combined with the previous step, which then makes this only a two-step tutorial.  It would be great if we could expand this just a little more. -->

## Conclusion

In this tutorial, you implemented a Slack slash command by setting up a Flask app that is served by a uWSGI application server and a Nginx reverse proxy server. To encrypt the connection for the slash command, you should look into using HTTPS for the request URL. You can do so by [installing a free SSL certificate issued by Let's Encrypt on the Nginx server](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04).

<!-- TODO:  what else can we do with this setup? Give us more examples and ideas for further exploration. -->
