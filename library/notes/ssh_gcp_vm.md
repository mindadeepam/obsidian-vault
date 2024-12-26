
when os-login is enabled ->

1. To connect using the **CLI** and **glcoud command** you need to enter the following command. 
	`gcloud compute ssh --project=data-science-test-343606  --zone=us-central1-a personalized-classifier-platform`





## pcp vm


  

1. ImportError: libGL.so.1: cannot open shared object file: No such file or directory

`apt-get update && apt-get install ffmpeg libsm6 libxext6 -y`

2. command to run backend on pm2 (u must be inside *cm_app* folder)

`pm2 start app.py --watch --name cm_app_backend --interpreter python3 -- --port 3000`

3. **setup ssh keys** ->

`ssh -i /Users/deepam_minda_farmart_co/.ssh/pcp_ssh deepam_minda_farmart_co@34.27.121.30`

  

setup:

1. run on ur local machine

`ssh-keygen -t rsa -f ~/.ssh/pcp_ssh -C deepam.minda@farmart.co -b 2048`

2. Add the public key to the VM instance’s SSH keys section on the GCP Console. (entire string)

3. we have os login enabled. hence run

```gcloud compute os-login ssh-keys add \

--key-file=./fmt-ds-dev-gateway.pub \

--project=data-science-test-343606```

4. change permissions of pvt and pub file:

- `chmod 600 ~/.ssh/pcp_ssh.pub`

- `chmod 600 ~/.ssh/pcp_ssh`

5. ssh using

`ssh -i /Users/deepam_minda_farmart_co/.ssh/pcp_ssh deepam_minda_farmart_co@34.27.121.30`

  

4. nginx setup follow [this](https://data-science.getoutline.com/doc/gateway-server-deployment-strategy-HvqjA7LOoi#h-22-nginx) doc from outline.

  
  
  

5.

you must have conda installed *pcp_{ENV}*, cd content_moderation && pip install -r requirements, install and setup pm2 ..

- mkdir fmt
- sudo chmod -R a+rwx fmt/
- cd fmt
- git clone https://github.com/FarMart-Tech/content_moderation.git
- cd content_moderation
- git checkout dev
- git pull
- conda activate pcp_${ENV}
- cd cm_app
- pm2 reload cm_app_backend
	(pm2 start app.py --watch --name cm_app_backend --interpreter python3 -- --port 3000)



root passwd - RIMbcMgwhdZE57g