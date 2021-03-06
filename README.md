# Terraform azurerm using azurestackcloud provider 
Terraform azurerm provider using local Azure stack for public and usgov endpoints.

## Local development environment
Repo running on WSL 2 Ubuntu 18.04

## Setup
### Configure
1. Update the environment *.env files to point to the full path of the *.json endpoint files.
1. Update the environment *.env files with the TF_VAR_SUBSCRIPTION_ID, TF_VAR_TENANT_ID and TF_VAR_LOCATION.
1. Update the TF_VAR_BACKEND_STORAGE_ACCOUNT_NAME since this is globally unique for each cloud instance.
1. You are now ready to set the environment variables from the appropriate env file.\
set -a && . environment/dev/dev.env && set +a

Note: using a Windows mounted path from WSL does not work in step 1.

### Deploy
Create the remote storage infrastructure. 
This is a one time initialization of the storage and the state is stored in teh local folder.
1. Navigate to the 01_init folder
1. $ terraform init
1. $ terraform plan
1. $ terraform apply --auto-approve

Create the remote state backed resource. A sample is provided in 02_bas0\1_net
1. Navigate to the 02_base\1_net folder\
2. Retrieve the storage account access key from:\
Resource group: azstack-remote-state\
Storage account: azstackstore\
Access key\
The access key can be added to the provider.tf, environment\dev\dev.env file or used in the init reconfigure below.
3. $ terraform init

Using the access key directly (not saved in the code)
```
terraform init -reconfigure \
    -backend-config="access_key=accessKeyFromAzureStorageAccountLength88Char==" \
    -backend-config="subscription_id=${TF_VAR_SUBSCRIPTION_ID}" \
    -backend-config="resource_group_name=${TF_VAR_BACKEND_RESOURCE_GROUP_NAME}" \
    -backend-config="storage_account_name=${TF_VAR_BACKEND_STORAGE_ACCOUNT_NAME}" \
    -backend-config="container_name=${TF_VAR_BACKEND_CONTAINER_NAME}" \
    -backend-config="key=02_base/01_net"
```

Using the dev.env environment variable
```
terraform init -reconfigure \
    -backend-config="access_key=${TF_VAR_ACCESS_KEY}" \
    -backend-config="subscription_id=${TF_VAR_SUBSCRIPTION_ID}" \
    -backend-config="resource_group_name=${TF_VAR_BACKEND_RESOURCE_GROUP_NAME}" \
    -backend-config="storage_account_name=${TF_VAR_BACKEND_STORAGE_ACCOUNT_NAME}" \
    -backend-config="container_name=${TF_VAR_BACKEND_CONTAINER_NAME}" \
    -backend-config="key=02_base/01_net"
```

4. $ terraform plan
5. $ terraform apply --auto-approve
6. Confirm resource creation and the remote state is saved in the storage account container

Note: The access key should be placed in Azure Key vault or passed in using the terraform init reconfigure command as provided below. This follows best security practices and keeps your secret safe and out of code/source control.


Confirm by accessing the Azure portal\
Resource group: azstack-remote-state\
Storage account: azstackstore\
Container: azstfrs\
Blob: 02_base/01_net (https://yourstorename.blob.core.windows.net/azstfrs/02_base/01_net)

### Destroy
To remove the resources, run this command in the each folder, in reverse order.
1. $ terraform destroy --auto-approve.

Notes: 
* If the environment is completely recreated, a new Access Key will be generated, update will be necessary.
* If changing the Access Key, the .terraform folder in that deployment modeule will have to be deleted before init.



