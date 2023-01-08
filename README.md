# rclone-scripts-azue-oci
rclone scripts to migrate content from azure to oci

This repo contents some useful scripts to migrate data from Azure Blob to Oracle Cloud Infrastructure Object Storage Service (OCI-OSS)

## Table of Contents 

1. Create OCI Object Storage Service buckets as per provieded storage accounts.
2. RClone Based Migration Scripts

## What you will need ?

1. OCI or Azure VM with 24OCPUs / VCPs and with At-least 128GB RAM and 1 TB HDD
2. az cli
3. oci cli
4. rclone

## Let's Do This

### Linux System Changes Required (I used Oracle Linux and Ubuntu and providing examples for ubuntu)

You will must have following limits on /etc/security/limits.conf

```
*          soft    nproc     4096
root       soft    nproc     unlimited
root soft     nproc          65000    
root hard     nproc          65000   
root hard     nofile         65000
```

Also read following links as well

https://project-sunbird.atlassian.net/browse/ED-462
https://github.com/project-sunbird/sunbird-devops/discussions/3605
(Yes I am originally doing this for sunbird 🦖 )

### Setting up Rclone 

To install rclone on Linux/macOS/BSD systems, run:

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

Follwing Should be installed under root (sudo su - or sudo -i ) and please make sure to delete your compute after migration done.

### Install Azure CLI

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
and continue as per the installation instructions. Just press enter until the end if you are new to this or no need any customizations :-D

### Install OCI CLI

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```
and continue as per the installation instructions. Just press enter until the end if you are new to this or no need any customizations :-D

## Configuration

### 1. Configure az cli and oci cli

run following command to login to az cli and then do as per console instructions. command will exit upon the successful login. 

```
az login
```

rul following command to setup oci cli

```
oci setup config
```

run following command to login to az cli and then do as per console instructions. command will exit upon the successful login. Or read following to setup instance principal https://www.ateam-oracle.com/post/calling-oci-cli-using-instance-principal to the instance (Only applicable if you are creating the instance on OCI and not valid for azure)







