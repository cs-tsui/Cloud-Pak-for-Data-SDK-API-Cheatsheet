### Filter Deployment Spaces by Name and get ID

### CPD Internal Service URL

```
import os
internal_cpd_url = os.environ['RUNTIME_ENV_APSX_URL']
```
'https://internal-nginx-svc:12443'

### CPD Token Generation
```
import os

# From a notebook, there is no need to generate 
# token manaully
cpd_token = os.environ['USER_ACCESS_TOKEN']

# Or generate with username and API key
def get_cpd_token(parms):
    url = '{}/icp4d-api/v1/authorize'.format(parms['url'])
    payload = {
        'username': parms['username'],
        'api_key': parms['apikey']
    }
    response = requests.post(url, headers=headers, json=payload, verify=False)
    json_data = response.json()
    access_token = json_data['token']
    return access_token

cpd_parms = {
    'username': <username>,
    'apikey': <apikey>,
    'url': <cpd_url>,
}
cpd_token = get_cpd_token(cpd_parms)
```

### Returns the first space_id that matches the name
```
def get_space_id_by_name(wml_client, deployment_space)
    filtered_spaces = [space['metadata']['id'] for space in wml_client.spaces.get_details()['resources'] if
                           space['entity']['name'] == deployment_space]
    if len(filtered_spaces) > 0:
        space_id = filtered_spaces[0]
    else:
        space_id = None
    print(space_id)
    return space_id
```
