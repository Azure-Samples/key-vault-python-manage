---
page_type: sample
languages:
- python
products:
- azure
description: "This sample demonstrates how to manage key vaults in Azure using the Python SDK."
urlFragment: key-vault-python-manage
---

# Manage key vaults with Python

This sample demonstrates how to manage key vaults in Azure using the Python SDK.

**On this page**

- [Run this sample](#run)
- [What does example.py do?](#example)
    - [Create a key vault](#create)
    - [Delete a key vault](#delete)

<a id="run"></a>
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
## What is example.py doing?

This sample starts by setting up `ResourceManagementClient` and `KeyVaultManagementClient` objects using your subscription and credentials.

```python
#
# Create the Resource Manager Client with an Application (service principal) token provider
#
subscription_id = os.environ['AZURE_SUBSCRIPTION_ID']

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
print_item(resource_client.resource_groups.create_or_update(GROUP_NAME, resource_group_params))
```

Here, the `create_or_update` method returns a `ResourceGroup` object
after performing the appropriate operation,
and the supporting function `print_item` prints some of its attributes.

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
                'object_id': OBJECT_ID,
                'permissions': {
                    'keys': ['all'],
                    'secrets': ['all']
                }
            }]
        }
    }
)
```

The object ID is unique for a User or an Application. Find this number in the Azure Active Directory blade of the Azure portal:
* To find a User's object ID, navigate to "Users and groups" > "All users", search for the user name, and click it.
* To find an Application's object ID, search for the application name under "App registrations" and click it.

In either of these cases, you can then find the object ID in the Essentials box.

<a id="list"></a>
### List key vaults

This code lists some attributes of all available key vaults.

```python
for vault in kv_client.vaults.list():
    print_item(vault)
```

<a id="delete"></a>
### Delete a key vault

```python
delete_async_operation = resource_client.resource_groups.delete(GROUP_NAME)
delete_async_operation.wait()
print("\nDeleted: {}".format(GROUP_NAME))
```

Deleting a resource is an asynchronous operation which may take some time, so the object
returned from `delete` represents an operation in progress. Calling `wait` on it
forces the caller to wait until it finishes.
