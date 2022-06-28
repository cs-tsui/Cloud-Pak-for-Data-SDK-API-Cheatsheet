
# Table of Content
- [Table of Content](#table-of-content)
- [CPD Platform](#cpd-platform)
    - [CPD internal service URL](#cpd-internal-service-url)
    - [Generate CPD token](#generate-cpd-token)
- [OpenScale](#openscale)
    - [Get list of (accessible) OpenScale instances](#get-list-of-accessible-openscale-instances)
    - [Instantiate OpenScale Client](#instantiate-openscale-client)
    - [Printing Subscription and Related Info for Debugging](#printing-subscription-and-related-info-for-debugging)
  - [Custom Metrics](#custom-metrics)
    - [Patch/Update Custom Metric Monitor's Integrated System ID](#patchupdate-custom-metric-monitors-integrated-system-id)
- [WML](#wml)
    - [WML (Minimal) Payload Format Req for Python Function](#wml-minimal-payload-format-req-for-python-function)
    - [Instantiate WML Client](#instantiate-wml-client)
    - [Get deployment space ID by name](#get-deployment-space-id-by-name)
    - [Get asset from space by name (WML Client)](#get-asset-from-space-by-name-wml-client)

# CPD Platform

Below is a collection of snippets for many commmon (and not so common) operations programmatically with the platform and its various services.

### CPD internal service URL

```py
import os
internal_cpd_url = os.environ['RUNTIME_ENV_APSX_URL']
```
`https://internal-nginx-svc:12443`

### Generate CPD token
```py
import os

# From a notebook, there is no need to generate token manaully
cpd_token = os.environ['USER_ACCESS_TOKEN']

# Or generate with username and API key
def get_cpd_token(cpd_username, cpd_apikey, cpd_url):
    url = '{}/icp4d-api/v1/authorize'.format(cpd_url)
    payload = {
        'username': cpd_username,
        'api_key': cpd_apikey
    }
    response = requests.post(url, json=payload, verify=False)
    json_data = response.json()
    access_token = json_data['token']
    return access_token
```

```py
cpd_token = get_cpd_token(cpd_username, cpd_apikey, cpd_url)
``` 

# OpenScale

### Get list of (accessible) OpenScale instances

This uses a util function to get instances, we can also call the platform API
manaully go get instances.

```py
from ibm_watson_openscale.utils.client_utils import get_my_instance_ids
my_instances = get_my_instance_ids(cpd_url, cpd_username, apikey=cpd_api_key)
print(my_instances)
```

`['00000000-0000-0000-0000-000000000000 (openscale-defaultinstance)']`


or

```py
def getServiceInstances(cpd_username, cpd_token, cpd_url):
    
    headers = {
            "Content-Type": "application/json",
            "Authorization": "Bearer {}".format(cpd_token)
    }
    
    resp =requests.get('{0}/zen-data/v3/service_instances?limit=100'.format(cpd_url), verify=False, headers=headers)
    service_instances = resp.json()['service_instances']
    # print(service_instances)
    return service_instances

svc_instances = getServiceInstances(cpd_username, cpd_token, cpd_url)
svc_instances
```

### Instantiate OpenScale Client
```py
from ibm_cloud_sdk_core.authenticators import CloudPakForDataAuthenticator
from ibm_cloud_sdk_core.authenticators import BearerTokenAuthenticator
from ibm_watson_openscale import APIClient

os_client = None
authenticator = BearerTokenAuthenticator(bearer_token=cpd_token)
os_client = APIClient(service_url=cpd_url, authenticator=authenticator)
```

### Printing Subscription and Related Info for Debugging

```py
# prefetch integrated systems for ease of lookup by ID
# Remember the value in the dict is the integrated system object
integrated_systems = IntegratedSystems(wos_client).list().result.integrated_systems
integrated_systems_dict = {}

for iss in integrated_systems:
    integrated_systems_dict[iss.metadata.id] = iss
integrated_systems_dict

# Target subscription to print
subscription_name_target = <your subscription name>
subscriptions = wos_client.subscriptions.list().result.subscriptions

for subscription in subscriptions:
    # Subscription Dict will hold more information after it's fetched
    sub_dict = subscription.to_dict()
    print("====================================================================================")
    print("| Subscription: ", subscription.entity.asset.name)
    print("====================================================================================")
#     print(isinstance(subscription, object))
#     print(subscription)
#     print(type(subscription))
    
    # Filter on name if subscription name exists
    if subscription_name_target is not None:
        if subscription.entity.asset.name != subscription_name_target:
            print("Subscription name doesn't match. Skipping fetch")
            continue
        else:
            print("Found subscription with name: ", subscription_name_target)
    
    print(subscription)
    
    # Get subscription ID from the object - alternatively convert to dict first
    sub_id = subscription.metadata.id
    
    # Get monitored instances for the subscription
    monitor_instances = wos_client.monitor_instances.list(target_target_id=sub_id)

    print("====================================================================================")
    print("| Monitor Instances ({}) for ".format(len(monitor_instances.result.monitor_instances)), subscription.entity.asset.name)
    print("====================================================================================")
    
    
    # TODO: get openpages integrated system details
    # from integration system
    # 
    #  'parameters': {'custom_metrics_provider_id': 'b7d80d11-d9d7-47e7-833c-45b863c1e0ad',
    #    'custom_metrics_wait_time': 120},

    for mon_instance in monitor_instances.result.monitor_instances:
        
        mon_def_id = mon_instance.entity.monitor_definition_id
        print("====================================================================================")
        print("| Monitor Instance: ", mon_def_id)
        print("====================================================================================")
        print(mon_instance)
        
        print(mon_instance.entity.parameters)
        # For each monitor instance, if custom metric with provider, fetch custom metric provider information
        if hasattr(mon_instance.entity, 'parameters') and 'custom_metrics_provider_id' in mon_instance.entity.parameters:
            custom_metrics_provider_id = mon_instance.entity.parameters['custom_metrics_provider_id'] # also references the integrated system ID
            
            print("====================================================================================")
            print("| Custom Metric - has integration ", mon_def_id)
            print("====================================================================================")
            print("TODO")
            if custom_metrics_provider_id in integrated_systems_dict:
                int_sys_obj = integrated_systems_dict[custom_metrics_provider_id]
                print(int_sys_obj.to_dict())
            else:
                # TODO: throw error instead of just print
                print("Custom Metric Provider/Integrated System not found for ID ", custom_metrics_provider_id)
        
            # Get monitor definition
            print("====================================================================================")
            print("| Monitor Definition for", mon_def_id)
            print("====================================================================================")
            mon_def = wos_client.monitor_definitions.get(monitor_definition_id=mon_instance.entity.monitor_definition_id).result
            print(mon_def)
```


## Custom Metrics

### Patch/Update Custom Metric Monitor's Integrated System ID
```py
# From the output above we need get the ID of the custom metrics provider
integrated_systems = IntegratedSystems(wos_client).list().result
print(integrated_systems)
custom_integrated_system_id = <correct integrated system id>

# We also need the custom monitor instance ID - use the show below, or other methods like subscription print to get the monitor instance ID
wos_client.monitor_instances.show(10)
custom_monitor_instance_id = <the monitor instance ID for our subscription>


payload = [
    {
       "op": "replace",
       "path": "/parameters",
       "value": {
           "custom_metrics_provider_id": custom_integrated_system_id
           "custom_metrics_wait_time":   120 
       }
    }
]
payload

# run the patch
response = wos_client.monitor_instances.update(custom_monitor_instance_id, payload, update_metadata_only = True)
result = response.result
result
```


# WML

### WML (Minimal) Payload Format Req for Python Function

`fields` key can also be omitted, shown here just for convenience.

```json
{
   "input_data":[
      {
         "values":{},
         "fields":null
      }
   ]
}
```

### Instantiate WML Client
```py
from ibm_watson_machine_learning import APIClient

wml_client = None
wml_credentials = {
    "token": cpd_token,
    "instance_id": "openshift",
    "url": cpd_url,
    "version": "4.0"
}

wml_client = APIClient(wml_credentials)
```


###  Get deployment space ID by name
```py
def get_space_id_by_name(wml_client, deployment_space):
    filtered_spaces = [space['metadata']['id'] for space in wml_client.spaces.get_details()['resources'] if
                           space['entity']['name'] == deployment_space]
    if len(filtered_spaces) > 0:
        space_id = filtered_spaces[0]
    else:
        space_id = None
    print(space_id)
    return space_id
```

### Get asset from space by name (WML Client)

```py
def get_asset_from_space_by_name(wml_client, deployment_space):
    wml_client
```


