---
services: keyvault
platforms: python
author: lmazuel
---

# Manage key vaults with Python

This sample demonstrates how to manage key vaults in Azure using the Python SDK.

**On this page**

- [Run this sample](#run)
- [What does example.py do?](#example)
    - [Create a key vault](#create)
    - [Delete a key vault](#delete)

<a id="run"/>
## Run this sample

1. If you don't already have it, [install Python](https://www.python.org/downloads/).

2. We recommend using a [virtual environment](https://docs.python.org/3/tutorial/venv.html) to run this example, but it's not mandatory. You can initialize a virtual environment this way:

    ```
    pip install virtualenv
    virtualenv mytestenv
    cd mytestenv
    source bin/activate
    ```

3. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/key-vault-python-manage.git
    ```

4. Install the dependencies using pip.

    ```
    cd key-vault-python-manage
    pip install -r requirements.txt
    ```

5. Create an Azure service principal, using 
[Azure CLI](http://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal-cli/),
[PowerShell](http://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal/)
or [Azure Portal](http://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/).

6. Export these environment variables into your current shell. 

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    ```

7. Run the sample.

    ```
    python example.py
    ```

<a id="example"></a>
## What is example.rb doing?

This sample starts by setting up ResourceManagementClient and KeyVaultManagementClient objects using your subscription and credentials.

```python
#
# Create the Resource Manager Client with an Application (service principal) token provider
#
subscription_id = os.environ.get(
    'AZURE_SUBSCRIPTION_ID',
    '11111111-1111-1111-1111-111111111111') # your Azure Subscription Id
credentials = ServicePrincipalCredentials(
    client_id=os.environ['AZURE_CLIENT_ID'],
    secret=os.environ['AZURE_CLIENT_SECRET'],
    tenant=os.environ['AZURE_TENANT_ID']
)
kv_client = KeyVaultManagementClient(credentials, subscription_id)
resource_client = ResourceManagementClient(credentials, subscription_id)
```

It registers the subscription for the "Microsoft.KeyVault" namespace
and creates a resource group and a storage account where the media services will be managed.

```python
# You MIGHT need to add KeyVault as a valid provider for these credentials
# If so, this operation has to be done only once for each credentials
resource_client.providers.register('Microsoft.KeyVault')

# Create Resource group
resource_group_params = {'location': WEST_US}
resource_client.resource_groups.create_or_update(GROUP_NAME, resource_group_params)
```

There is a supporting function (`print`) that print a resource group and it's properties.
With that set up, the sample lists all resource groups for your subscription, it performs these operations.

<a id="create"></a>
### Create a key vault

```python
vault = kv_client.vaults.create_or_update(
    GROUP_NAME,
    KV_NAME,
    {
        'location': WEST_US,
        'properties': {
            'sku': {
                'name': 'standard'
            },
            'tenant_id': os.environ['AZURE_TENANT_ID'],
            'access_policies': [{
                'tenant_id': os.environ['AZURE_TENANT_ID'],
                'object_id': os.environ['AZURE_TENANT_ID'],
                'permissions': {
                    'keys': ['all'],
                    'secrets': ['all']
                }
            }]
        }
    }
)
```

<a id="list"></a>
### List key vaults

This code lists the first 5 key vaults.

```python
vaults = keyvault_client.vaults.list(5)
```

<a id="delete"></a>
### Delete a key vault

```python
delete_async_operation = resource_client.resource_groups.delete(GROUP_NAME)
delete_async_operation.wait()
print("\nDeleted: {}".format(GROUP_NAME))
```
