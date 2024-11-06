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

## viewer token creation

```python
# Function to create a JWT token for the viewer API
def create_token():
    payload = {"client_id": CLIENT_ID}
    token = jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")
    # Decode to UTF-8 if necessary

    return token if isinstance(token, str) else token.decode("utf-8")
```
