# Setting Up an Airgapped Terraform Cluster on AWS

## Deploy an AWS Terraform Cluster:

### Prerequisites:
- [Terraform](https://www.terraform.io/downloads.html) 0.11.x
- Sales Mesosphere License Key (via OneLogin)[https://mesosphere.onelogin.com/notes/56317]
- Sales Mesosphere Private and Public Key (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] (This is the standard Mesosphere SSH key, You may have this preconfigured already)
- Retrieve Mesosphere MAWS Commandline tool for access to AWS: https://github.com/mesosphere/maws/releases

Installing Terraform:
Download and install terraform from [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html) . (Or just use brew install terraform on a Mac)

### Configure Mesosphere MAWS 

```bash
# Download maws-darwin binary from https://github.com/mesosphere/maws/releases
chmod +x maws*
sudo mv ~/Downloads/maws* /usr/local/bin/maws
```
If you are using maws for the first time, you need to list the account you have access too and login to one of them

Use this command

````
maws ls
````
Your output should look something like this:

```
You will now be taken to your browser for authentication. URL https://aws.mesosphere.com/sso?maws=56014
2 available Accounts:
Team 03:
    110465657741_Mesosphere-PowerUser

Team 10:
    273854932432_Mesosphere-PowerUser
````
Now use maws to login to retrieve your temporary account credentials

```
maws login 110465657741_Mesosphere-PowerUser
```

Your output should look something like this
```
retrieved credentials writing to profile 110465657741_Mesosphere-PowerUser
export AWS_PROFILE=110465657741_Mesosphere-PowerUser
```

Note that you will have to refresh the credentials (use the maws login command) every 1 hour

### Configure SSH Private and Public Key for Terraform

Set your ssh agent locally to point to your pem key and public key

```bash
$ ssh-add /path/to/ssh_private_key.pem
```
