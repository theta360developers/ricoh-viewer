# Alternate Tutorial for Minimal Application

The RICOH360 Viewer requires two things to start:

1. token for viewer
2. contentId for the image you want to show

```javascript linenums="1" hl_lines="4 8" title="index.html"
// instantiate viewer object
const viewer = new RICOH360Viewer({
    divId: "viewer",
    onFetchToken: () => "{{token}}",
});
// start viewer with content
viewer.start({
    contentId: "{{contentId}}"
});
```

## Setting up a virtual environment on Python

Although not required, I recommend that you set up a virtual
environment on Python.  This avoids conflicting libraries on your main system
Python.

```text
python -m venv venv
source venv/bin/activate
```

You should now see a `(venv)` prompt.

```text
(venv) craig@craigs-air practice %
```

!!! tip inline end
    The Private Key and the Client Secret are not the same.  You must
    get the Client ID, Client Secret, and Private Key from RICOH.  The
    Private Key is for the Viewer.  The Client Secret is for the content.

## Viewer Token

To generate the viewer token, you need the following:

1. Client ID
2. Private Key

We will use PyJWT and cryptography to generate the viewer token with the Private Key.

### install PyJWT and cryptography

PyJWT is needed to generate the JSON Web Token that the
viewer needs.  The cryptography package is needed for the
RS256 encryption used to encode the token.

```text
pip install PyJWT cryptography
```

!!! tip
    You can check the Python packages installed in your environment
    with `pip freeze`

### Create `server.py` file

Use VSCode or equivalent to create a file, `server.py`.

At the top, include `import jwt`.

Below the import, add your `PRIVATE_KEY` and `CLIENT_ID`.
The Private Key is long.  Put it in triple quotes.

![private key](images/tutorial2/private_key.png)

The Client ID is shorter.

![client id](images/tutorial2/client_id.png)

With the `CLIENT_ID` and `PRIVATE_KEY` set in your Python
script, you can now generate the token for the RICOH360 Viewer.

```python linenums="34"  title="server.py"
# generate token for RICOH360 Viewer
payload = {"client_id": CLIENT_ID}
token = jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")
print(f"token for RICOH360 Viewer: {token}")
```

### run `server.py`

Test the RICOH360 Viewer token creation by running
`python server.py`.

Expected output is shown below.  The token is shortened
in the example.

```text
python server.py              
token for RICOH360 Viewer: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.XE4c2tlamFtaTQzbmZqcWM3YjhwNGxjcXAifQ...
...
...
```

## Content ID

We can now instantiate the viewer.  However, we won't be able to
see any image in the viewer until we supply it with a contentId.

```javascript linenums="1" hl_lines="8" title="index.html"
// instantiate viewer object
const viewer = new RICOH360Viewer({
    divId: "viewer",
    onFetchToken: () => "{{token}}",
});
// start viewer with content
viewer.start({
    contentId: "{{contentId}}"
});
```

### Requirements for contentId

You need the following to get a contentId:

1. RICOH THETA images loaded up into your account on the RICOH360 Cloud
1. RICOH360 Cloud token generated with AWS Cognito

In addition to the requirements above, you also need the following from RICOH
to generate a RICOH360 Cloud token.

1. Client ID
1. Client Secret

The Client ID is the same ID used to generate the RICOH360 Viewer token.

Add the `CLIENT_SECRET` below the `CLIENT_ID` in your `server.py` file.

![client secret](images/tutorial2/client_secret.png)

### install requests

```text
pip install requests
```

Import requests and base64 in your `server.py` file.

```python linenums="1" title="server.py"
import jwt
import requests
import base64

PRIVATE_KEY = """-----BEGIN PRIVATE KEY-----
...
...
```

### Make request to AWS Cognito for RICOH360 Cloud token

Use this code to get the RICOH360 Cloud token.  Place it at the bottom of
your `server.py` file

```python linenums="42" title="server.py"
# generate token for RICOH360 Cloud API
# Endpoint and authentication for AWS token
token_endpoint = "https://saas-prod.auth.us-west-2.amazoncognito.com/oauth2/token"  # noqa: E501
auth = base64.b64encode(f"{CLIENT_ID}:{CLIENT_SECRET}".encode()).decode("utf-8")  # noqa: E501
headers = {
    "Content-Type": "application/x-www-form-urlencoded",
    "Authorization": f"Basic {auth}",
}
body = {"grant_type": "client_credentials", "scope": "all/read"}

# Request AWS token
token_response = requests.post(token_endpoint, headers=headers, data=body)
token_object = token_response.json()
ricoh_cloud_access_token = token_object.get("access_token")
print(8 * "=")
print(f"RICOH360 Cloud Token\n {ricoh_cloud_access_token}")
```

### Test RICOH360 Cloud Token Creation

Run `python server.py` to test the RICOH360 Cloud token creation.

You should see your two tokens printed to the console.

![tokens](images/tutorial2/tokens.png)

Congratulations.  Now that you have the two tokens, you're almost
done with the setup.  You just need to get a Content ID for the image
you want to display.

### Use Cloud Token to Get Content

Remember that our goal is to send a `contentId` to the HTML file that will
display the image.  Prior to building the HTML page, we are getting
the content ID to send to the HTML file.

Although we are only building the `server.py` file at this point, let's
look at the JavaScript snippet again to understand our goal.

```javascript linenums="1" hl_lines="8" title="index.html"
// instantiate viewer object
const viewer = new RICOH360Viewer({
    divId: "viewer",
    onFetchToken: () => "{{token}}",
});
// start viewer with content
viewer.start({
    contentId: "{{contentId}}"
});
```

Add this code to the bottom of your `server.py` file.

```python linenums="60" title="server.py"

# get content from RICOH360 Cloud server
print("start process to contact RICOH360 Cloud server to get content")
# Fetch content using the token
content_headers = {"Authorization": f"Bearer {ricoh_cloud_access_token}"}
content_response = requests.get(
    "https://api.ricoh360.com/contents?limit=1", headers=content_headers
)
content_data = content_response.json()
print("got response from RICOH360 Cloud server")
print("RICOH360 Cloud Content")
print(content_data)
```

### Test contents API

run `python server.py`

!!! tip
    Make sure you have content in your RICOH360 Cloud account.

You should see the content listing.

![content listing](images/tutorial2/content_list.png)

In most editors, you can also click on the link to the thumbnail. On a Mac,
I am using CMD-click.

![thumbnail](images/tutorial2/thumbnail.png)

### parse content_id from JSON

```python title="server.py" linenums="72"
# parse content_id
print(8 * "=")
print("Get content_id from data")
content_id = content_data[0]["content_id"]
print(content_id)
```

You should see this output on the console.

```text
========
Get content_id from data
c6eac34b-9bdc-4ba1-81af-29470fdead79
```
