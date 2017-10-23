# Deployment and Monitoring

Digital Ocean (AWS, Linode, etc.)

Generate key if one doesn't exist.

ssh-keygen -t rsa

View the public key, so you can share it with DigitalOcean.

cat ~/.ssh/id_rsa.pub

Provide your server with

cat ~/.ssh/id_rsa.pub | ssh root@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"

Avoid Root

adduser <username>
passwd <username>

usermod -aG sudo <username>

su - <username>

setup ssh for user

mkdir ~/.ssh
chmod 700 ~/.ssh

copy public key into ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

Software & Webservers

git

git init --bare chatty-redux.git

git remote add production ssh://kjensen@104.236.153.174/home/kjensen/chatty-redux.git

nvm

[https://github.com/creationix/nvm](https://github.com/creationix/nvm)

install nvm

once nvm is installed install node `nvm install v8.7.0`


pm2 & keymetrics

npm install pm2 -g

nginx

[https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)

sudo apt-get update

sudo apt-get install nginx

sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw status
sudo ufw status

systemctl status nginx

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04

location / {
  proxy_pass http://localhost:3000;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}

sudo nginx -t

sudo systemctl restart nginx

setup site to run

npm i (for dependencies)



pm2 custom metrics and actions

http://docs.keymetrics.io/docs/pages/custom-metrics/

