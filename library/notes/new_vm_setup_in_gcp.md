### Steps

- select a sutiable distribution, I generally go with ubuntu 20.04, boot disk size of 30-40gb and cpu with 4gb ram. location asia-south. 
- setup static ip for the VM -> [link](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address#console)
- *have os-login enabled* *(this allows us to use my gcloud setup to ssh into my VM, that way the email with which gcloud is setup becomes the username we get), otherwise a diff directory is created with the username with which u sshed into this VM* and I become that user & with sudo permission. ,y gcloud config is set as *service acc mail*!? {maybe bcuz my email is a user of that service account, i am allowed to get in?}   **read this up in detail** ----- with os-login enabled i become my email-derived username with sudo permission.
	  
	*then maybe the only difference really is that we dont have to maintain diff ssh keys for different user etc, we can use gcloud login the use gcloud cli commands to ssh etc.*
	
- ssh into the vm using -> 
```
gcloud compute ssh personalized-classifier-platform-dev
```
- set password using 
```
sudo passwd # -> bq2rQOi93QEOr2l
```
- 
	Now do the following 
```
cd /..
mkdir fmt && cd fmt

# install basics
apt-get update && apt-get install ffmpeg libsm6 libxext6 -y

# install miniconda via https://docs.anaconda.com/free/miniconda/index.html#quick-command-line-install

# setup automatic git login by using the .netrc file
put in credential like so ->
vi ~/.netrc
'''
paste this ->	
machine github.com
  login ******
  password ghp_***************************
'''


# git clone the repo
git clone https://github.com/FarMart-Tech/content_moderation.git

# create conda env
export ENV=dev
conda create -n $ENV python=3.9
conda activate $ENV

# install libraries

	#create a env.sh file which will contain the AUTH_USER and AUTH_PASSWORD
	vi ~/env.sh
	'''
	export AUTH_USER=*******
	export AUTH_PASSWORD=ghp_************************************
	'''
	source ~/env.sh

git checkout $ENV
cd content_moderation
pip install .
pip install -r requirements.txt
pip install -r cm_app/requirements.txt
```


##### Install nodejs and pm2
NodeJS Installation

[https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-debian-10](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-debian-10)

[https://linuxize.com/post/how-to-install-node-js-on-debian-10/](https://linuxize.com/post/how-to-install-node-js-on-debian-10/)

```
apt install nodejs npm
npm install pm2 -g
```

Use **NVM** to manage your Node installation.

```
# to install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
source ~/.bashrc

# to list all versions
nvm list-remote

# install your preferred version 
nvm install v20.9.0
```

When using NVM to install node, the node package isn’t installed in the `usr/bin` directory by default.(check usr/bin) 
To do so, 

```
 sudo ln -s "$(which node)" /usr/bin/node
 sudo ln -s "$(which npm)" /usr/bin/npm
```

##### Install and setup nginx
NGINX is utilized as the web server to handle HTTP/HTTPS requests efficiently. Server blocks within NGINX configurations have been customized to route traffic to the port on which the server is running. In addition, Certbot is being used for managing the SSL certificates and hence enable HTTPS.

**Steps to set up NGINX on a VM**
To install NGINX on your VM follow [this](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/) guide.

1. `cd /etc/nginx/`
2. `vi sites-available/cm_backend_app` and paste this ->
```
server {
	listen 80;
	listen [::]:80;

	root /var/www/<put-in-external-static-ip-here>/html;
	index index.html index.htm index.nginx-debian.html;

	server_name <put-in-external-static-ip-here>;

	location / {

			proxy_pass http://localhost:3000;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection 'upgrade';
			proxy_set_header Host $host;
			proxy_cache_bypass $http_upgrade;

	}
}
```

- The above defined bash code is called a **[server block](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04).** A server block enables us to expose   multiple websites from the same machine. For example, if I want to deploy both the dev and stage environments of the gateway server from the same virtual machine, I will create two files inside the `sites-available` folder: `gateway-dev` and `gateway-stage`. Each of them will point to two different URLs and they will forward traffic to the ports which are running the `dev` and `stage` environments respectively. Here it just forwards the port 3000 to port 80 of our local!
    
3. In order to enable this server-block, run the following command   
```
sudo ln -s /etc/nginx/sites-available/cm_app_backend /etc/nginx/sites-enabled/ 
```
 
 This will create a [symlink](https://www.freecodecamp.org/news/symlink-tutorial-in-linux-how-to-create-and-remove-a-symbolic-link/) from the files in the `sites-available` directory to the `sites-enabled` directory, which NGINX actually uses to serve HTTP requests. An advantage of using this config structure for NGINX is that you can selectively disable/enable the websites you want to host on a particular machine. 
    
The `sites-available` directory will store _all_ of your virtual host configurations, whether or not they're currently enabled. When you want to enable a particular host, use the above defined command to add a symlink from that configuration file to the `sites-enabled` directory. In case you want to disable a particular website for whatever reason, simply remove the symlink,  do this also when updating ur nginx serverblocks.

```
# remove symlink file in sites-enabled directory
rm symlink_name
```

 4. After you’re done making changes to your config files in the `sites-available` directory, run the following command to make sure there are no syntax errors.

```
sudo nginx -t
```

5. if everything is okay, restart nginx -> 
```
sudo systemctl start nginx
```
##### start cm backend server

- cd into cm_app folder at root_dir of content_mod repo
```
cd /home/fmt/content_moderation/cm_app

pm2 start app.py --watch --name cm_app_backend --interpreter python3 -- --port 3000
# app.py me the port is hardcoded!

```


### Note ->

> if ure root dont do `cd ~/../fmt `
> external ip for personalized-classifier-service-dev: 35.200.195.23

#### git pull and reload file ->
```
cd content_moderation/cm_app
echo "inside"
pwd
git checkout dev
git pull
conda env list
conda activate pcp_${ENV}
echo "raN CONDA on $ENV"
pm2 list
pm2 reload cm_app_backend
echo  "RAN PM2 reload"
pm2 list
```


#### note - tech debt : 
- in line `output=$(gcloud compute ssh personalized-classifier-platform-dev` ue env variables in workflow file for trigger_pcp_vm
- increase size of vm. setup ci-cd pipeline and infra using cloud run (we will try and shift to cloud run sometime later as it offers containerized solution, as discussed with Deepak.) (build image from source and using config, then use container to deploy via cloud run service in gcp. replicability is important (scratch se infra setup in ci-cd is apparently the standard practise.))





main passwd -> IMBK1CLYDErsq7x