# Moving Azure Blobs to Oracle Cloud Infrastructure Object Storage Service using RClone (Azure Blob to OCI OSS)
rclone scripts to migrate content from azure to oci

This repo contents some useful scripts to migrate data from Azure Blob to Oracle Cloud Infrastructure Object Storage Service (OCI-OSS)

## Table of Contents 

- [rclone-scripts-azue-oci](#rclone-scripts-azue-oci)
  * [Let's Do This](#let-s-do-this)
    + [Linux System Changes Required (I used Oracle Linux and Ubuntu and providing examples for ubuntu)](#linux-system-changes-required--i-used-oracle-linux-and-ubuntu-and-providing-examples-for-ubuntu-)
    + [Setting up Rclone](#setting-up-rclone)
    + [Install Azure CLI](#install-azure-cli)
    + [Install OCI CLI](#install-oci-cli)
  * [Configuration](#configuration)
    + [1. Configure az cli and oci cli](#1-configure-az-cli-and-oci-cli)
    + [2. Configure RClone](#2-configure-rclone)
    + [3. Creating Migration Scripts](#3-creating-migration-scripts)
      - [3.1 Automate OCI Bucket Creation](#31-automate-oci-bucket-creation)
      - [3.2 Automate Migration for Larger Objects](#32-automate-migration-for-larger-objects)
        * [Example Usage](#example-usage)

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


### 2. Configure RClone

RClone can be configured using following command but I am adding the required config for rclone v1.61.1 just to make your life easy

```
rclone config
```

make sure to have following alike config under the path "/root/.config/rclone/rclone.conf" (yes need root account)

```vim
[azure]
type = azureblob
env_auth = true

[oci]
type = oracleobjectstorage
namespace = axdTRUNCATEDixb
compartment = ocid1.compartment.oc1..aaaaaaaahnnivgp2mTRUNCATED6itp2cwmkhb7doa
region = ap-mumbai-1
```
basically I have given azure and oci as the names for the 2 CSP configurations. You may add any blocks as you may need.

### 3. Creating Migration Scripts

#### 3.1 Automate OCI Bucket Creation

You need to create buckets on OCI Side before doing the migration. So what I have done is to create a script to list containers inside given set of azure storage account and create those on oci oss.

Please remember that OCI do not have storage account concept and you cannot have duplicate name for buckets even inside oci compartments. So you will have to handle that issue with prefixing storage account name to the oci side bucket. (Need only if you have 2 containers with same name amoung multiple storage accounts. For ex production and testing storage accounts can have a container named config)

```bash
#!/bin/bash
#set path for az / oci binary
export PATH=$PATH:/root/bin/
# To Get a list of storage accounts execute following command 
#storage_accounts=$(az storage account list --output tsv --query '[].name')

#Now select required storage accounts 
storage_accounts="production dev testing"

#Provide OCI Compartment to Create new buckets
oci_compartment_id='ocid1.compartment.oc1..aaaaaaaahnnivTRUNCATeD'

# Iterate through each storage account
for storage_account in $storage_accounts; do
  # Get a list of all containers in the storage account
  containers=$(az storage container list --account-name $storage_account --num-results "*" --output tsv --query '[].name')

  # Iterate through each container
  for container in $containers; do
    #create oci bucket as per the template [Storage_Account_Name]-[Container_Name]
    oci os bucket create --name "$storage_account-$container"  --compartment-id $oci_compartment_id 2>&1 >> oci_os_creation.log
  done
done
```

I have tested above to create 196000+ buckets and it took around 2 days. This is thanks to the az cli / api list limits "single listing call is 5000."

#### 3.2 Automate Migration for Larger Objects

This is simple but please make sure to have a `screen` session to run script

follwing script designed to take az storage account container and oci oss buckets as command line arguments 


```bash
#!/bin/bash
export PATH=$PATH:/root/bin/
#azure to oci sync 
export AZURE_STORAGE_ACCOUNT_NAME=$1
time rclone --progress --stats-one-line --multi-thread-streams 10000 --multi-thread-cutoff 1Mi --multi-thread-streams 10000 --transfers 1000 --buffer-size 2048Mi --azureblob-chunk-size 100Mi --azureblob-list-chunk 5000 --max-stats-groups 5000 --oos-chunk-size 100Mi --oos-upload-concurrency 10000 copy azure:$2 oci:$3
```

##### Example Usage

```bash
#./azure_to_oci_sync.sh [azure_storage_account] [azure-container] [oci-bucket-name]
./azure_to_oci_sync.sh production config-container production-config-container
```

