
# Table of Content
- [Table of Content](#table-of-content)
- [CPD Platform](#cpd-platform)
    - [CPD internal service URL](#cpd-internal-service-url)
    - [Generate CPD token](#generate-cpd-token)
- [OpenScale](#openscale)
    - [Get list of (accessible) OpenScale instances](#get-list-of-accessible-openscale-instances)
- [Spaces](#spaces)
    - [Get deployment space ID by name](#get-deployment-space-id-by-name)
    - [Get asset from space by name (WML Client)](#get-asset-from-space-by-name-wml-client)

# CPD Platform

### CPD internal service URL

```
import os
internal_cpd_url = os.environ['RUNTIME_ENV_APSX_URL']
```
`https://internal-nginx-svc:12443`

### Generate CPD token
```py
import os

# From a notebook, there is no need to generate 
# token manaully
cpd_token = os.environ['USER_ACCESS_TOKEN']

# Or generate with username and API key
def get_cpd_token(cpd_username, cpd_apikey, cpd_url):
    url = '{}/icp4d-api/v1/authorize'.format(cpd_url)
    payload = {
        'username': cpd_username,
        'api_key': cpd_apikey
    }
    response = requests.post(url, headers=headers, json=payload, verify=False)
    json_data = response.json()
    access_token = json_data['token']
    return access_token

cpd_token = get_cpd_token(cpd_username, cpd_apikey, cpd_url)
``` 

# OpenScale

### Get list of (accessible) OpenScale instances
```py
from ibm_watson_openscale.utils.client_utils import get_my_instance_ids
my_instances = get_my_instance_ids(cpd_url, cpd_username, apikey=cpd_api_key)
print(my_instances)
```

`['00000000-0000-0000-0000-000000000000 (openscale-defaultinstance)']`


# Spaces
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