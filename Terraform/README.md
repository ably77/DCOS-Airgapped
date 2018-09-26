# Setting Up an Airgapped Terraform Cluster on AWS

## Deploy an AWS Terraform Cluster:

### Prerequisites:
- Access to the Mesosphere Internal Github Repo
- [Terraform](https://www.terraform.io/downloads.html) 0.11.x
- Sales Mesosphere License Key (via OneLogin)[https://mesosphere.onelogin.com/notes/56317]
- Sales Mesosphere Private and Public Key (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] (This is the standard Mesosphere SSH key, You may have this preconfigured already)
- Retrieve Mesosphere MAWS Commandline tool for access to AWS: https://github.com/mesosphere/maws/releases

### Mesosphere Internal Github

If you do not have access to the Internal Mesosphere Github https://github.com/mesosphere then request access by opening up an IT Support ticket at http://desk.mesosphere.com

**2-Factor Authentication (2FA)**

Mesosphere Company policy requires 2FA to be enabled, please do so accordingly

**SSH Key**

Make sure that your Github account has SSH properly set up:

https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/

### Install Terraform on your Local Machine

If you're on a mac environment with homebrew installed, run this command.

```
brew install terraform
```

If you have terraform already installed, it is a good idea to update to the latest stable version of Terraform

```
brew update
brew upgrade terraform
```

If you want to leverage the terraform installer, feel free to check out https://www.terraform.io/downloads.html.

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

## Terraform Deployment

### Making changes by using the Terraform’s desired_cluster_profile -var-file

When reading the commands below relating to installing and upgrading, it may be easier for you to keep all these flags in a seperate file instead of the default example provided. This way you can make a change to the file and it will persist when you do other commands to your cluster in the future.

For example:

This command below already has the flags on what I need to install such has:
* DC/OS Version 1.11.4
* Masters 1
* Private Agents 1
* Public Agents 7
* Security Mode
* MAWS AWS Profile
* 1.11+ License Key


When we view the file, you can see how you can save your state of your cluster:

```
$ cat desired_cluster_profile
num_of_masters = "1"
num_of_private_agents = "7"
num_of_public_agents = "1"
dcos_security = "permissive"
dcos_version = "1.11.4"
aws_profile = "<INSERT_MAWS_AWS_ACCOUNT_HERE>"
dcos_license_key_contents = "<INSERT_LICENSE_KEY_HERE>"
```

*Note:**: The variables.tf file is also included here for you to view what inputs can be added to the desired_cluster_profile options file


## Build a DC/OS Cluster using Terraform

### Grab Terraform Scripts from Github Repo:

```
terraform init -from-module git@github.com:mesosphere/enterprise-terraform-dcos//aws
```

### Spin Up DC/OS Cluster with Desired Cluster Profile

```
terraform apply -var-file desired_cluster_profile
```
