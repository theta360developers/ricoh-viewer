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

Now that goal of the Python server code is top in your mind,
let's move back to our Python file and focus on achieving
the goal of getting a `contentId` that we can pass to the
viewer in the `index.html` file.

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

## HTML File

Now that we have the `contentId` and the viewer `token`, we
can now create the HTML file that will hold the browser.

In a new index.html file, put the following code:

```html title="index.html" linenums="1"
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://r360pf-prod-static.s3.us-west-2.amazonaws.com/viewer/v0.15.0/ricoh360-viewer.js"></script>
  </head>
  <body>
    <div
      style="
        position: relative;
        width: 100%;
        height: 650px;
        background-color: #2a303c;
      "
    >
      <div id="viewer"></div>
    </div>
    <script>
      const viewer = new RICOH360Viewer({
        divId: "viewer",
        onFetchToken: () => "{{token}}",
      });
      viewer.start({
        contentId: "{{contentId}}",
      });
    </script>
  </body>
</html>
```

## Connect `server.py` and `index.html`

We just need to connect the backend `server.py` file
with the frontend `index.html` file.  To connect
the files, we will use Flask.

`pip install Flask`

In the `server.py` file, import the Flask packages.

```python title="server.py" linenums="1"
from flask import Flask, render_template
```

Specify how to start Flask and let the application know
that the `index.html` file is in the same folder as the `server.py`
file.

Below the section where you specified `CLIENT_SECRET`.

```python title="server.py"
app = Flask(__name__)
app.template_folder = "."
```

### in `server.py` create route to `index.html`

At the bottom of your `server.py` file, add the route to `index.html`

```python title="server.py"
@app.route("/")
def index():
    print(f"contentId: {content_id}")
    return render_template("index.html",  token=token, contentId=content_id)
```

### start flask server

At the bottom of `server.py`, add this:

```python title="server.py"
app.run(port=3000, debug=True)
print("Open browser at http://localhost:3000 or http://127.0.0.1:3000")
```

## run application

`python server.py`

![open browser console](images/tutorial2/open_browser.png)

![browser](images/tutorial2/browser.png)

Congratulations!  You have now configured the RICOH360 Viewer.

Please see the other tutorials for customization of the browser
and connection with the powerful RICOH360 Cloud APIs.

## bonus challenges

### change image

First, load multiple images into your RICOH360 Cloud account.

In `server.py` file, change the number of images you get from the
RICOH360 Cloud server from 1 to 5.

```python
content_response = requests.get(
    "https://api.ricoh360.com/contents?limit=5", headers=content_headers
)
```

When you get the `content_id`, change the index of the content from 0 to 1.

```python
content_id = content_data[1]["content_id"]
```

reload browser.

You should see the next image.

![browser image 2](images/tutorial2/browser_image2.png)

Change the index to 2.

```python
content_id = content_data[2]["content_id"]
```

Reload the browser to see the next image.

![image 3](images/tutorial2/browser_image_3.png)

### enhance image

In `index.html`, apply `transform: 'enhancement'` below
the contentId.

```javascript
viewer.start({
  contentId: "{{contentId}}",
  transform: 'enhancement'
});
```

The image is now enhanced.

![enhancement](images/tutorial2/enhancement.png)

### annotation

Set up a constant for the viewer ui.  In the toolbar,
add the annotation button.

Place the `ui` constant after the `onFetchToken` callback.

```javascript linenums="21" hl_lines="11"
const ui = {
  toolbar: {
    annotationButton: {
      isHidden: false
    }
  }
}
const viewer = new RICOH360Viewer({
  divId: "viewer",
  onFetchToken: () => "{{token}}",
  ui
});
```

![annotation](images/tutorial2/annotation.png)

### drawing

```javascript
const ui = {
  toolbar: {
    annotationButton: {
      isHidden: false
    },
    drawingButton: {
      isHidden: false
    }
  }
}
```

![drawing](images/tutorial2/drawing.png)

### split-screen

The split-screen control is in the `verticalPanel`.

```javascript
const ui = {
  toolbar: {
    annotationButton: {
      isHidden: false
    },
    drawingButton: {
      isHidden: false
    }
  },
  verticalPanel: {
    isHidden: false,
    topMargin: 25,
  }
}
```

![split-screen](images/tutorial2/split-screen.png)

### control of right pane

![right pane enhance](images/tutorial2/right-pane-enhance.png)

1. create a function to switch scene
1. in the function, add the pane "1", which is the right pane
1. create a button to run the function to switch the scene

```javascript
const viewer = new RICOH360Viewer({
    divId: "viewer",
    onFetchToken: () => "{{token}}",
    ui
  });
viewer.start({
  contentId: "{{contentId}}",
});
const enhanceRightPane = () => {
  viewer.switchScene(
  {
    contentId: "{{contentId}}",
    transform: 'enhancement'
  }, "1"
  );
}
```

Below the viewer, add this:

```html
<button type="button" onclick="enhanceRightPane()">enhance right</button>
```
