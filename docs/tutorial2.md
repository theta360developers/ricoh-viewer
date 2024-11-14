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

We will use PyJWT to generate the viewer token with the Private Key.

### install PyJWT

```text
pip install PyJWT
```

### Create `server.py` file

Use VSCode or equivalent to create a file, `server.py`.

At the top, include `import jwt`.

Below the import, add your `PRIVATE_KEY` and `CLIENT_ID`.
The Private Key is long.  Put it in triple quotes.

![private key](images/tutorial2/private_key.png)

The Client ID is shorter.

![client id](images/tutorial2/client_id.png)
