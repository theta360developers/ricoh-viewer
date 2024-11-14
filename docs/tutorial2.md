# Alternate Tutorial for Minimal Application

The RICOH360 Viewer requires two things to start:

1. token for viewer
2. contentId for the image you want to show

```javascript linenums="1" hl_lines="4 8" title="index.html snippet"
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

```python linenums="34"  title="server.py snippet"
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

## contentId

We can now instantiate the viewer.  However, we won't be able to
see any image in the viewer until we supply it with a contentId.

```javascript linenums="1" hl_lines="8" title="index.html snippet"
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

```python
import jwt
import requests
import base64

PRIVATE_KEY = """-----BEGIN PRIVATE KEY-----
...
...
```
