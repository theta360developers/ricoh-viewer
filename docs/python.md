# Python Demo with Flask and Jinja

![home screen](images/python/home_screen.png)

An example Python Flask server is in the [python_server.py](https://github.com/theta360developers/oppkey-ricoh-viewer-demo-basic/blob/main/python_server.py)
file.

Python is an alternative to JavaScript on the backend.  If your
backend infrascture uses JavaScript, you do not need to use Python.

The backend can be built with any language.  Python is used to
illustrate a simple example of using any backend framework.

To run the Python demo, follow the step below.

```text
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python python_server.py
```

You will then need to open a web browser at the URL provided.

```text
python python_server.py
 * Serving Flask app 'python_server'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:3000

```

## viewer token creation using PyJWT

The demo uses [PyJWT](https://pyjwt.readthedocs.io/en/stable/) to create the web token.  To create the token, you must use the Private Key
obtained from RICOH.  This demo does not come with a private key.

You must have the private key in your  `secrets.env` file.

```python
# Function to create a JWT token for the viewer API
def create_token():
    payload = {"client_id": CLIENT_ID}
    token = jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")
    # Decode to UTF-8 if necessary

    return token if isinstance(token, str) else token.decode("utf-8")
```

## viewer HTML template with Jinja2

The HTML file to display the viewer is in [`views/flask_viewer.html`](https://github.com/theta360developers/oppkey-ricoh-viewer-demo-basic/blob/main/views/flask_viewer.html).

The viewer is imported with this line:

`<script src="https://r360pf-prod-static.s3.us-west-2.amazonaws.com/viewer/v0.15.0/ricoh360-viewer.js"></script>`

### Viewer instantiation

The fetchToken method returns the token generated in the Python
file. The Python variable that was passed into the HTML is in
double quotes `{{token}}`.

```javascript
const fetchToken = () => "{{token}}";

const viewer = new RICOH360Viewer({
    divId: "viewer",
    onFetchToken: fetchToken,
    isCubemapEnabled: true,
    ui,
onSelected: (index) => {
    console.log("index: ", index);
    },
    onCropped: onCropped,
}
);
```

## Image Selection Demo

The left pane is index 0.  The right pane is index 1.

![gui demo](images/python/python_gui.png)

This is an example creating the list of image for selection.
The `contentId` and `thumbnail_url` are parsed in JavaScript from the data sent
from Python.

In actual use, you may want to parse the data from the RICOH Cloud API
in the Python server-side code in order to improve performance and reduce
the data transmitted over the network.

```javascript
 const leftList = document.getElementById("leftList");
    for (let i = 0; i < data.length; i++) {
    if (data[i].status === "uploaded") {
        // console.log(data[i]);
        const listItem = document.createElement("li");
        listItem.innerHTML = `<div class="p-1">
        <img style="cursor: pointer;" src="${
            data[i].thumbnail_url
        }" onclick="viewer.switchScene({ contentId: '${
        data[i].content_id
        }'},${0})">
        <button type="button" class = "btn btn-primary mt-1" onclick="viewer.switchScene({ contentId: '${
            data[i].content_id
        }',transform:'enhancement'},${0})">
            Enhance
        </button>
        <button type="button" class = "btn btn-primary mt-1" onclick="viewer.switchScene({ contentId: '${
            data[i].content_id
        }',transform:'b_person'},${0})">
            Blur People
        </button>
        <button type="button" class = "btn btn-primary mt-1" onclick="viewer.switchScene({ contentId: '${
            data[i].content_id
        }',transform:'p_cubic'},${0})">
            Cubic View
        </button>
        </div>`;
        leftList.append(listItem);
    }
```
