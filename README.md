# pennbot

pennbot is a chat bot built on the [Hubot][hubot] framework. It was
initially generated by [generator-hubot][generator-hubot], and is being deployed
on a CentOS7 linux server.

[hubot]: http://hubot.github.com
[generator-hubot]: https://github.com/github/generator-hubot

### Running pennbot Locally

You can test your hubot by running the following, however some plugins will not
behave as expected unless the [environment variables](#configuration) they rely
upon have been set.

You can start pennbot locally by running:

    % bin/hubot

You'll see some start up output and a prompt:

    [Sat Feb 28 2015 12:38:27 GMT+0000 (GMT)] INFO Using default redis on localhost:6379
    pennbot>

Then you can interact with pennbot by typing `pennbot help`.

    pennbot> pennbot help
    pennbot animate me <query> - The same thing as `image me`, except adds [snip]
    pennbot help - Displays all of the help commands that pennbot knows about.
    ...

### Configuration

A few scripts (including some installed by default) require environment
variables to be set as a simple form of configuration.

Each script should have a commented header which contains a "Configuration"
section that explains which values it requires to be placed in which variable.
When you have lots of scripts installed this process can be quite labour
intensive. The following shell command can be used as a stop gap until an
easier way to do this has been implemented.

    grep -o 'hubot-[a-z0-9_-]\+' external-scripts.json | \
      xargs -n1 -I {} sh -c 'sed -n "/^# Configuration/,/^#$/ s/^/{} /p" \
          $(find node_modules/{}/ -name "*.coffee")' | \
        awk -F '#' '{ printf "%-25s %s\n", $1, $2 }'

How to set environment variables will be specific to your operating system.
Rather than recreate the various methods and best practices in achieving this,
it's suggested that you search for a dedicated guide focused on your OS.

### Scripting

An example script is included at `scripts/example.coffee`, so check it out to
get started, along with the [Scripting Guide][scripting-docs].

For many common tasks, there's a good chance someone has already one to do just
the thing.

[scripting-docs]: https://github.com/github/hubot/blob/master/docs/scripting.md

### external-scripts

There will inevitably be functionality that everyone will want. Instead of
writing it yourself, you can use existing plugins.

Hubot is able to load plugins from third-party `npm` packages. This is the
recommended way to add functionality to your hubot. You can get a list of
available hubot plugins on [npmjs.com][npmjs] or by using `npm search`:

    % npm search hubot-scripts panda
    NAME             DESCRIPTION                        AUTHOR DATE       VERSION KEYWORDS
    hubot-pandapanda a hubot script for panda responses =missu 2014-11-30 0.9.2   hubot hubot-scripts panda
    ...


To use a package, check the package's documentation, but in general it is:

1. Use `npm install --save` to add the package to `package.json` and install it
2. Add the package name to `external-scripts.json` as a double quoted string

You can review `external-scripts.json` to see what is included by default.

##### Advanced Usage

It is also possible to define `external-scripts.json` as an object to
explicitly specify which scripts from a package should be included. The example
below, for example, will only activate two of the six available scripts inside
the `hubot-fun` plugin, but all four of those in `hubot-auto-deploy`.

```json
{
  "hubot-fun": [
    "crazy",
    "thanks"
  ],
  "hubot-auto-deploy": "*"
}
```

**Be aware that not all plugins support this usage and will typically fallback
to including all scripts.**

[npmjs]: https://www.npmjs.com

### hubot-scripts

Before hubot plugin packages were adopted, most plugins were held in the
[hubot-scripts][hubot-scripts] package. Some of these plugins have yet to be
migrated to their own packages. They can still be used but the setup is a bit
different.

To enable scripts from the hubot-scripts package, add the script name with
extension as a double quoted string to the `hubot-scripts.json` file in this
repo.

[hubot-scripts]: https://github.com/github/hubot-scripts

##  Persistence

If you are going to use the `hubot-redis-brain` package (strongly suggested),
you will need to install Redis on your server.

```
sudo yum install redis
sudo systemctl start redis
```

By default, Redis will be listening on port 6379 and bound to localhost, meaning
it will be able to accept connections only from a hubot running on the same computer it
is running. The conf file is default located in `/etc/redis.conf`.

To verify that Redis is running you can try `redis-cli ping` which should 
reply `PONG`. You can also try `sudo systemctl status redis`. 

If you don't need any persistence feel free to remove the `hubot-redis-brain`
from `external-scripts.json` and you don't need to worry about redis at all.


## Adapters

Adapters are the interface to the service you want your hubot to run on, such
as Campfire or IRC or Mattermost. There are a number of third party adapters that the
community have contributed. Check [Hubot Adapters][hubot-adapters] for the
available ones.

If you would like to run a non-Campfire or shell adapter you will need to add
the adapter package as a dependency to the `package.json` file in the
`dependencies` section.

Once you've added the dependency with `npm install --save` to install it you
can then run hubot with the adapter.

    % bin/hubot -a <adapter>

Where `<adapter>` is the name of your adapter without the `hubot-` prefix.

Pennbot is using the `hubot-mattermost` adapter found [here][hubot-mattermost].

[hubot-adapters]: https://github.com/github/hubot/blob/master/docs/adapters.md
[hubot-mattermost]: https://github.com/renanvicente/hubot-mattermost

### Deploying with nginx

Messages to the server from mattermost will need to be sent over HTTPS listening on port 443
and passed onto hubot, running on localhost (127.0.0.1) listening on port 8080. Below is an example server block used by nginx.
In this example, the source code for hubot will live in the directory `/var/www/example.com`.
```
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name  example.com;
    root /var/www/example.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

  location / {
    proxy_pass http://127.0.0.1:8080/incoming_messages/;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
    # logging
    access_log /var/log/nginx/bots.log;
    error_log  /var/log/nginx/error_bots.log;

}
```

### Deploying to UNIX or Windows

If you would like to deploy to either a UNIX operating system or Windows.
Please check out the [deploying hubot onto UNIX][deploy-unix] and [deploying
hubot onto Windows][deploy-windows] wiki pages.

[heroku-node-docs]: http://devcenter.heroku.com/articles/node-js
[deploy-heroku]: https://github.com/github/hubot/blob/master/docs/deploying/heroku.md
[deploy-unix]: https://github.com/github/hubot/blob/master/docs/deploying/unix.md
[deploy-windows]: https://github.com/github/hubot/blob/master/docs/deploying/windows.md



### Creating a service with systemd

You can use systemd on CentOS to keep hubot running in the background. Below
is an example file `/etc/systemd/system/examplebot.service`.

```
[Unit]
Description=Examplebot
Requires=network.target
After=network.target

[Service]
Type=simple
WorkingDirectory=/var/www/example.com
User=examplebot
Group=examplebot

Restart=always
; RestartSec=10

; Configure Hubot environment variables, use quotes around vars with whitespace as shown below.
;Environment="HUBOT_aaa=xxx"
;Environment="HUBOT_bbb='yyy yyy'"
Environment="MATTERMOST_INCOME_URL=https://example_mattermost_server.com/hooks/your_webhook"
Environment="MATTERMOST_ENDPOINT=/incoming_messages/"
Environment="MATTERMOST_HUBOT_USERNAME=examplebot"
Environment="MATTERMOST_ICON_URL=http://example.com/icon.jpg"
Environment="MATTERMOST_TOKEN=aaaaaaasdffffffffsss"

ExecStart=/var/www/example.com/bin/hubot --adapter mattermost

[Install]
WantedBy=multi-user.target
```

Start the service with `sudo systemctl start examplebot.service`

## Restart the bot

Use `sudo systemctl restart examplebot.service`.

