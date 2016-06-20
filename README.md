# patreon-python
Interact with the Patreon API via OAuth.

Get the egg from [PyPI](https://pypi.python.org/pypi/patreon)


Step 1. Get your client_id and client_secret
---
Visit the [OAuth Documentation Page](https://www.patreon.com/oauth2/documentation)
while logged in as a Patreon creator to register your client.

This will provide you with a `client_id` and a `client_secret`.


Step 2. Use this library
---
e.g., in a Flask route
```python
import patreon
from flask import request
...

client_id = None      # Replace with your data
client_secret = None  # Replace with your data
creator_id = None     # Replace with your data

@app.route('/oauth/redirect')
def oauth_redirect():
    oauth_client = patreon.OAuth(client_id, client_secret)
    tokens = oauth_client.get_tokens(request.args.get('code'), redirect_uri)
    access_token = tokens['access_token']

    api_client = patreon.API(access_token)
    user_response = api_client.fetch_user()
    user = user_response['data']
    included = user_response.get('included')
    if included:
      pledge = next((obj for obj in included
        if obj['type'] == 'pledge' and obj['relationships']['creator']['data']['id'] == creator_id), None)
      campaign = next((obj for obj in included
        if obj['type'] == 'campaign' and obj['relationships']['creator']['data']['id'] == creator_id), None)
    else:
      pledge = nil
      campaign = nil

    # pass user, pledge, and campaign to your view to render as needed
```


Step 3. (Optional) Customize your usage
---
`patreon.API` instances have four methods:
* `fetch_user(includes=None, fields=None)`
* `fetch_campaign(includes=None, fields=None)`
* `fetch_campaign_and_patrons(includes=None, fields=None)`
* `fetch_page_of_pledges(campaign_id, page_size, cursor=None, includes=None, fields=None)`

The `includes` and `fields` arguments to these methods specify
the [related resources](http://jsonapi.org/format/#fetching-includes)
and the [resource attributes](http://jsonapi.org/format/#fetching-sparse-fieldsets)
you want returned by our API, as per the [JSON:API specification](http://jsonapi.org/).
The lists of valid `includes` and `fields` arguments are provided on `patreon.schemas`.
For instance, if you wanted to request the total amount a patron has ever paid to your campaign,
which is not included by default, you could do:
```python
api_client = patreon.API(patron_access_token)
patron_response = api_client.fetch_user(None, {
  'pledge': patreon.schemas.pledge.default_attributes + [patreon.schemas.pledge.Attributes.total_historical_amount_cents]
})
```