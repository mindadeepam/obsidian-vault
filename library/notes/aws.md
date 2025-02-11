## login
The data science team is migrating from GCP(Google Cloud services) to AWS(Amazon Web Services). To start using it locally you need to be authenticated with AWS.

### Log in to the AWS Console

1. Firstly, to login to the AWS console on the browser go to: [https://farmart.awsapps.com/start/#/?tab=accounts](https://farmart.awsapps.com/start/#/?tab=accounts).
    
    1. If `email_id = deepam.minda@farmart.co` then, `user_name = deepam.minda`
        
    2. Set up a new password by clicking on forgot password.
        

2. Once logged in, you can select the `developer`, then select the `fmt-data-science-dev` role.

![](media/aws-access-portal.png)


### Log in via the terminal

If you select the `access keys` option besides the role, you can see the following

![](media/aws-access-keys-ss.png|400x300)

These are the various ways to authenticate your local device with aws so that you can programmatically access things like s3 buckets, containers, etc.

Option: 1 set aws environment variable is the shortest option, but those credentials expire in 1 hour, and you will have to manually reset those variables.

Hence we will go with the recommended option which allows us to be authenticated for longer.

3. Firstly to download aws-cli for mac-os, run the following in the terminal:
    

```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

For other OS refer to [this](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) document.

4. Then run `aws configure sso` and follow the following video.
    

![](media/aws-configure-sso-sr.mov)

#### Under the hood

This stores the default configurations in `~/.aws/config`, which looks something like this:


![](media/aws-config-ss.png)

Here the SSO-Session part **authenticates** you with AWS and the **profile** configuration authorizes you with certain roles and policies.

#### Longevity of sign-in

Although the login time is more than 1 hour via SSO login, it is still only 8 hours by default (it can be set to 12 max). So you may have to run these every day.

```
   # Login with default profile
   aws sso login

   # OR login with specific profile
   aws sso login --profile your-profile-name
```

### References

5. [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
    
6. [https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
## login to a different role in cli
```bash
aws sso login --profile fmt-gcp-to-aws-policy-378990335490
```

## ECR
- login and docker pull from ecr
```bash
aws ecr get-login-password --region ap-south-1 --profile fmt-data-science-dev-378990335490 | docker login --username AWS --password-stdin 378990335490.dkr.ecr.ap-south-1.amazonaws.com

aws ecr get-login-password --region ap-south-1 --profile fmt-gcp-to-aws-policy-378990335490 | docker login --username AWS --password-stdin 378990335490.dkr.ecr.ap-south-1.amazonaws.com

```
- create repo
```bash
aws ecr create-repository --repository-name "$REPOSITORY_NAME" --region ap-south-1 --profile fmt-gcp-to-aws-policy-378990335490
```


