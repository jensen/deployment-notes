# Deployment and Monitoring

We have a [web application](https://github.com/jensen/redux-notes/). Now what? How do we share it with the world. Heroku, AWS Elastic Beanstalk and other managed cloud application platforms will help with this. If you are curious about how to setup a self-managed server then another option will be necessary. We use a VPS (Virtual Private Server) instance to do this.

Even though I did this work with DigitalOcean, the process is still going to be similar for whatever host you choose.

- Create Instance
- Login to VPS
- Add non-root super user account
- Install git and setup remote repo with hooks
- Install application dependencies
- Install pm2 to ensure server restarts itself
- Install nginx to allow connections from outside to be proxied to your application
- Open up ports on your firewall if necessary
- Push from local repo to remote repo to deploy

## Choosing a VPS

There are a lot of options when choosing a host.  Today we used DigitalOcean.

- [DigitalOcean](https://www.digitalocean.com/)
- [Amazon Web Services](https://aws.amazon.com/)
- [Linode](https://www.linode.com/)
- [SSD Nodes](https://www.ssdnodes.com/)
- ...

## Setting up the VPS

Each of these VPS providers will have a control panel that allows you to create instances. For DigitalOcean these are called `droplets`. In order to login to the server provided by DigitalOcean we can use ssh. In order to make this easy we will use a public ssh key that we will share with DigitalOcean. If you don't have a public key then you can create one with `ssh-keygen -t rsa`. If you already have one, then you can find it by typing `cat ~/.ssh/id_rsa.pub`.

More details can be found in the [DigitalOcean Documentation](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets).

Once you have created a droplet and chosen the os, location and instance name you will be given an IP address.

### Gaining Acces

If you have setup the SSH key correctly then you will be able to login with `ssh root@<ip>`. It is now possible to start configuring the VPS for your own use.

### Avoiding Root

We avoid using the `root` account when doing this type of work. We will need it initially, but once we are able to login to the server then the first thing you would do is create a new super user.

More details can be found in the [DigitalOcean Documentation](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart).


```
adduser <username>
passwd <username>
usermod -aG sudo <username>
su - <username>

mkdir ~/.ssh
chmod 700 ~/.ssh

copy public key to ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
```

## Installing Software

This section is likely going to be slightly different unless your needs are exactly the same as mine. Some things that I didn't install that you may need instead.

- PostgreSQL, MySQL
- Ruby, Python, PHP
- Apache

The app that we are going to deploy is a Redux/React app with an Express server that runs in Node. It stores it's data in memory. It can be found at [https://github.com/jensen/redux-notes/](https://github.com/jensen/redux-notes/).

### git

If you have to install `git` then use the package manager provided by the OS running on your VM. We want to create a remote repo on the VPS and link our local repo. First thing to do is create a bare repo.

On the remote server type init the bare repot with `git init --bare chatty-redux.git` locally you can connect to the remote repot with `git remote add production ssh://kjensen@<ip>/home/<username>/chatty-redux.git`.

Before you push, you will want to setup the `post-receive` hook to automatically checkout the latest version after deploying. Create a file called `post-receive` in the `/home/<username>/chatty-redux.git/hooks` dir. A basic example would checkout the latest version after every push.

`chatty-redux.git/hooks/post-receive`
```
#!/bin/sh
git --work-tree=/var/www/domain.com --git-dir=/var/repo/site.git checkout -f
```

Once you have created this file, you can make it executable `chmod +x post-receive`.

More details can be found in the [DigitalOcean Documentation](https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps).

### nvm

It is a good idea to install `nvm` rather than `node` first. This will allow you to manage different versions of `node` on the same machine. Follow the [instructions](https://github.com/creationix/nvm) provided by the `nvm` developers. If you type `nvm` and it doesn't work you likely have to restart the terminal session to load it up. The `nvm` instructions that get ouput after install, explain ths. Once nvm is installed install node `nvm install v8.7.0` (or whatever version you want).

Once `node` is install you can install the dependencies with `npm i` and then build the production assets with `npm run prod:build`. A simple way to run the server is with `NODE_ENV=production node server` from the chatty-redux root directory. The server is now listening on port 3000.

### nginx

The server is listening, but it is behind a firewall. We want to setup a webserver to listen for connections on port 80 and proxy them to port 3000. We can use nginx for this.

Based on the OS selection that we made (Ubuntu 16.04) we can use apt to install the software.

```
sudo apt-get update
sudo apt-get install nginx
```

More details can be found in the [DigitalOcean Documentation](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04).

> You may need to open up some ports on your server for nginx to work. Some providers are a lot more strict and close everything.

Once nginx is installed you will want to setup the proxy for the node application. More details can be found in the [DigitalOcean Documentation)[https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04].

Modify `/etc/nginx/sites-available/default` to include the proxy configuration for your app. Make sure the port is correct.

```
location / {
  proxy_pass http://localhost:3000;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

You should test your configuation changes before restarting the server. In a production environment, this could take your site down for minutes. You can do this by running `sudo nginx -t`. If everything is good after the test then run `sudo systemctl restart nginx` to restart nginx. Navigate to your address in order to see the nginx welcome page or your running app if you setup the proxy.

### pm2

As soon as the outside world can connect to the server you will want to start using `pm2` to manage the process. Webservers crash. When they do we want to have them automatically rebooted. To run the `chatty-redux` application in production you would execute `NODE_ENV=production pm2 start server --name 'chatty-redux'`.

Useful pm2 commands:

- `pm2 start <script> --name '<name>'`
- `pm2 stop <name>`
- `pm2 delete <name>`
- `pm2 restart <name>`
- `pm2 list`
- `pm2 logs`

In addition to restarting your server when it crashes, pm2 can provide monitoring of your application. The company responsible for [pm2](http://pm2.keymetrics.io/) also provides monitoring software through [https://keymetrics.io/](https://keymetrics.io/). In order to link your running process with the keymetrics dashboard you would run a command on the server similar to `pm2 link a1b2c3d4e5f6g7 z9y8x7w6v5u4t3 <machine name>`, this information can be found on the dashboard.


## Bonus

We can use the `pmx` library from keymetrics to add metrics and actions to the dashboard. [Metrics](http://docs.keymetrics.io/docs/pages/custom-metrics/) are things like `realtime users`, `realtime messages` and `messages per minute`. [Actions](
http://docs.keymetrics.io/docs/pages/custom-actions/
) are things like `send:notification` that would be able to send a message to all connected users.

The [feature/pm2 branch](https://github.com/jensen/redux-notes/blob/feature/pm2/server/profiling.js) has some examples of this in the `server/profiling.js` file.


