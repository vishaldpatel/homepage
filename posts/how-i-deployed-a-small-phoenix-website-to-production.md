# From Dev to Production for a small Phoenix project
## Published: 2024-10-28

I followed the following steps to deploy this website to production.

## Dev

If it is a simple app, then use Sqlite. It will save us from having to setup and configure a more robust database like Postgres.

## VPC

Go to Linode. Creating an account is very simple. I created the smallest, cheapest shared Linode with Arch Linux.

The output should be a root user account and an IP address that we can SSH into.

## Setting up Linux

Using an SSH client, log into the newly created Linux machine as root.

1. Create a personal user account in order to not have to login as root again.
2. Add the newly created user to the wheel group, and the sudoers list.
3. Install the following packages (`sudo pacman -S <package name>`): `git`, `elixir`, `sqlite`, `make`, `gcc`, `certbot`, `nginx`

## Website

Git clone the project, then follow instructions found in the [Introduction to Deployment](https://hexdocs.pm/phoenix/deployment.html) doc to create a production environment and start the phoenix server.

Start the website synchronously (without backgrounding it) and test to see if it worked by going to the ipaddress:port for your project.

## DNS

Register a domain if you don't already have one.

Edit the domain's DNS records and create an A-record to point to the newly created server's IP address.

## Setting up Nginx

It is possible to serve multiple websites (serving multiple domains) from this one tiny server.

In order to facilitate this, each website will have to run off a different port and Nginx should be configured to act as a proxy to forward requests to the corresponding project.

- Our `nginx.conf` should look like [this](https://gist.github.com/vishaldpatel/1aa05edfe345087e035b9fe00d779f6f).
- Our project-specific nginx config will look like [this](https://gist.github.com/vishaldpatel/d9fb76bf2d027e6d2cd0e2a0886ac29d).
- Don't forget to ensure that your project's nginx config file has the correct websocket related settings. See: [WebSocket proxying](https://nginx.org/en/docs/http/websocket.html).
- Be sure to `chmod +x` all the way to the webroot to make sure that nginx is able to send requests to your project, or you'll end up seeing errors in your nginx log file.
- Be sure to run `nginx -t` to make sure your configs are error free.

## Certbot

Run Certbot to get a free SSL certificate from the wonderful people at Let's Encrypt. We can have it automatically update nginx's settings with the correct SSL info, but I sometimes have to go in and clean things up, so be careful.

## Test the domain

Start the Phoenix app. Visit the domain. We should have a working `https://...` and our project should light right up.

## Run the project in background mode!

Start the server in background mode.

In order to restart it, for now, I kill the existing process.

## Setting up the first user

Start iex using `MIX_ENV=prod iex -S mix` to create any users I needed.

Next, I will try to figure out how to make proper use of Phoenix / Elixir deploys, but for now, I have a working website in production, and so can you! =)

## Manual release management

If you have a Phoenix server running:

1. `cd` into the production directory.
2. `git pull`
3. Run migrations.
4. Start another one on a different port.
5. Update nginx settings to point to the new one.
6. Test nginx to make sure that your settings are valid.
7. Restart nginx for the settings to take place.
8. Kill the old phoenix server's PID.
