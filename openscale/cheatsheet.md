### Filter Deployment Spaces by Name and get ID


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
