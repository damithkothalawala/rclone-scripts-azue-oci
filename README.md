# rclone-scripts-azue-oci
rclone scripts to migrate content from azure to oci

This repo contents some useful scripts to migrate data from Azure Blob to Oracle Cloud Infrastructure Object Storage Service (OCI-OSS)

Table of Contents 

1. Create OCI Object Storage Service buckets as per provieded storage accounts.
2. RClone Based Migration Scripts

What you will need ?

1. OCI or Azure VM with 24OCPUs / VCPs and with At-least 128GB RAM and 1 TB HDD
2. az cli
3. oci cli
4. rclone


Linux System Changes Required

You will must have following limits on /etc/security/limits.conf

```
*          soft    nproc     4096
root       soft    nproc     unlimited
root soft     nproc          65000    
root hard     nproc          65000   
root hard     nofile         65000
```

