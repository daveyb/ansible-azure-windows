# Ansible for Azure and Windows

Get started using Ansible from Windows to manage VMs in Azure

## Getting started

1. Clone this repo

```
git clone https://github.com/daveyb/ansible-azure-windows.git
```

2. Pull the Ansible container

```
docker pull davidbenedic/azure-ansible-container:latest
```

3. Pull Azure CLI container

```
docker pull straubt1/azhelper
```

## Create Azure Service Principal
- An Azure Service Principal, or SP, is an account in Active Directory, designed for an application, that is granted permission to access an Azure subscription or a subset of resources in that subscription.
- The Service Principal account is used by Ansible to:
    1. Stand up infrastructure
    2. Query the Azure API for information about that infrastructure (typically used for dynamic inventory and/or inventory scripts)

1. Launch the Azure CLI container, mapping the azure folder from your host OS into the container as a volume

```
docker run --rm -it -v C:\Users\<username>\.azure:/root/.azure straubt1/azhelper
```

2. Login to the Azure CLI (from within the container)

``` 
az login
```

Follow the instruction in the CLI to complete the login. Note that once you login, you won't need to login again as long as you map the host azure directory into the container at runtime.

3. Create an Application identity in Active Directory (you must be a co-owner of your Azure Subscription, or given explicit permission to do so by your Subscription owner).

```
az ad app create --display-name ansible-automation --homepage http://www.ansible.com --identifier-uris http://www.ansible.com
```

Please make note of the `appId` JSON property in the output of the above command

4. Create a Service Principal and assign it a password (aka "secret")

```
az ad sp create-for-rbac --name <appID> --password "<a good password>" 
```

5. Assign a Service Principal and access scope to the subscription (defaults to 'Contributor' role, which allows resource creation/deletion)

```
az role assignment create --assignee <appId from step 3> --role Contributor --scope /subscriptions/<subscription id>
```

If you need to find out your subscription id or tenant id...

```
az account show
```

6. Create an Azure Credentials file on the host

```
New-Item -Type Directory -Name ~\.azure

New-Item -Type File -Name ~\.azure\credentials
```

Fill out your `~\.azure\credentials` file as follows:

```
[default]
tenant=<Tenant Id>
subscription_id=<Subscription Id>
client_id=<App Id>
secret=<Serice Principal Password>
```

For more information about Azure Active Directory Service Principals, please refer to [https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

## Using the azure-ansible-container

1. Start the container, mounting the host azure credentials file _and_ the ansible-azure-windows repository into the container

```
docker run --rm -it -v C:\Users\<username>\.azure:/root/.azure -v C:\Users\<username>\<path-to-git-repo>:/ansible davidbenedic/azure-ansible-container:latest
```

2. Update the ansible with _your_ credentials by opening `infra.yml` and entering in values for `admin_username` and `admin_password`.

3. Create the Azure infrastructure

```
ansible-playbook infra.yml
```

4. Update the ansible hosts file with the `ansible_user`, `ansible_password`, and Public IP address of your VM (under the `[demo]` directive)

5. Start running playbooks against your host

```
ansible-playbook shell.yml -i hosts
```

