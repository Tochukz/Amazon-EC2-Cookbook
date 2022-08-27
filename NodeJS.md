## Configure Node.JS Server on Ubunut using PM2 and Nginx Reverse Proxy
__Installing Node.js using NVM__  
Inastall NVM on ubuntub using a shell script obtained via curl
```
$ curl  https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
```
Load the environment created by the downloaded NVM installer
```
$ source ~/.profile  
```
Now install noe using the nvm
```
$ nvm install node
```
Other userful NVM commands
```
nvm install 12.18.3
nvm ls
nvm ls-remote
nvm use 12.18.3
nvm run default -version
nvm exec 12.18.3 server.js
```
For details see [How to Install NVM on Ubuntu 20/04](https://tecadmin.net/how-to-install-nvm-on-ubuntu-20-04/)

__Installing PM2__    
```
$ npm install pm2@latest -g
```
Useful PM2 commands
```
$ pm2 start app.js --name myapp
$ pm2 restart myapp
$ pm2 reload myapp
$ pm2 stop myapp
$ pm2 delete myapp
```
Useful PM2 options
```
--name <app_name>
--log <log_path>
```
Load Balance 4 instances of api.js:
```
$ pm2 start api.js -i 4
```
Monitor in production:
```
$ pm2 monitor
```
Make pm2 auto-boot at server restart:
```
$ pm2 startup
```
[PM2 Getting started](https://pm2.keymetrics.io/docs/usage/quick-start/)  
[PM2 Key metrics](https://pm2.keymetrics.io/)


__Setting Up Nginx as a Reverse Proxy Server__   
Edit your website configuration file
```
$ sudo nano /etc/nginx/sites-available/example.com
```
Update your server block as follows
```
server {
...
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
...
}
```
Check for Nginx configuration file syntax correctness
```
$ suo nginx -t
```
Restart Nginx
```
$ sudo systemctl restart nginx
```
For detail see [Digital Ocean: Setup Node.js for Production on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-20-04)  
