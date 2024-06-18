# Project Structure

To use the viewer, build a simple server to provide routes
for the UI that you embed the viewer in.

This example uses  [Express](https://expressjs.com/) for the server.
Let's say you create two routes, `/viewer` and `/content`.  The
first route to `/viewer` loads the React app that then
loads the RICOH Viewer.  The React app will also load  content with
a HTTP GET request to  `/content`.

In the code example `views/viewer.ejs`, this snippet goes to the
`/content` route.

```typescript
const res = await fetch("/content", {
          method: "GET",
          headers: {
            "Content-Type": "application/x-www-form-urlencoded",
          },
        });
```

When the app calls `/content`, Express will run
a method that you created. In this example, the method is called
`getContent()`.

![browser process](images/structure/browser_process.png)

Prior to actually getting the content, you must first generate a token.
The function `getContent()` will first generate a token, then it will
get the content from the RICOH360 Content API.  The token is put into
the Authorization header.

The example below uses the JavaScript [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) package.

```typescript
fetch("https://api.ricoh360.com/contents", {
    method: "GET",
    headers: {
      Authorization: "Bearer " + tokenObject.access_token,
    },
  });
```

- **views/**: Folder that holds view/HTML files (EJS markdown files)
  - **index.ejs**: Home/landing page template
  - **viewer.ejs**: Viewer page template
- **index.js**: File where server side functions and API endpoints to these functions are held

## Tokens

The demo uses two different tokens.  There is a token for the viewer and a token for the content API. It can be confusing.  Don't worry, we'll
walk you through it.

The token for the Content API is generated with Amazon Cognito with
the Client ID and Client Secret.

The token for the RICOH Viewer is generated with the Client ID
and Private Key
using [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken). 

The Private Key and the Client Secret are not the same.

We provide examples for both tokens and show how to use them
in your API calls.

## Server-side functions

### createToken()

```typescript
const privatekey = `-----BEGIN PRIVATE KEY-----
  <client private key>
-----END PRIVATE KEY-----`;

const createToken = () => {
  const payload = {
    client_id: <your client id>,
  };

  const accessToken = jwt.sign(payload, privatekey, {
    algorithm: "RS256",
    expiresIn: "60m",
  });

  return accessToken;
};
```

- `createToken()` uses a payload object with your clientID and optionally your groupID
- Using [jsonwebtoken package](https://www.npmjs.com/package/jsonwebtoken), you create and return an access token with an encryption algorithm that expires in a certain time (60 minutes)
- This function will be called client side via the API endpoint for this function and used in a callback function for Ricoh's viewer API

### getContent()

```typescript
const clientId = "<your client id>";
const clientSecret = "<your client secret>";

const getContent = async () => {
  const tokenEndpoint =
    "https://saas-prod.auth.us-west-2.amazoncognito.com/oauth2/token";

  const auth = Buffer.from(`${clientId}:${clientSecret}`).toString("base64");

  const requestData = {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
      Authorization: `Basic ${auth}`,
    },
    body: new URLSearchParams({
      grant_type: "client_credentials",
      scope: "all/read",
    }),
  };

  const tokenResponse = await fetch(tokenEndpoint, requestData);
  const tokenObject = await tokenResponse.json();

  const res = await fetch("https://api.ricoh360.com/contents", {
    method: "GET",
    headers: {
      Authorization: "Bearer " + tokenObject.access_token,
    },
  });
  const data = await res.json();

  return data;
};
```

- This function takes your clientID and clientSecret, turns it into base64 string format and sends a post request to `https://saas-prod.auth.us-west-2.amazoncognito.com/oauth2/token` (set as tokenEndpoint) to get a token to use for Ricoh content API
- After getting the token response, it creates another post request to Ricoh content API with the token as authorization and returning the data

## Server-side endpoints

### /

```typescript
app.get("/", (req, res) => {
  res.render("index");
});
```

This endpoint returns the EJS template file (index.ejs) in ./views

### /viewer

```typescript
app.get("/viewer", (req, res) => {
  res.render("viewer");
});
```

Calls up the main viewer with the split view, transformations, and
content.

### /token

```typescript
app.get("/token", (req, res) => {
  let token = createToken();
  res.status(200).send(token);
});
```

This endpoint calls on the `createToken()` server-side function and returns the token

### /content

```javascript
app.get("/content", async (req, res) => {
  let test = await getContent();
  res.status(200).send(test);
});
```

This endpoint calls on the `getContent()` async function and returns the content data which is needed by the viewer.

## Client-side (in ./views/index.ejs)

### fetchContent

```typescript
const fetchContent = async () => {

  // fetch content from /content endpoint and store data in const data
  const res = await fetch("/content", {
    method: "GET",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
  });
  const data = await res.json();

  // populate buttons on left pane list
  const leftList = document.getElementById("leftList");
  for (let i = 0; i < data.length; i++) {
    if (data[i].status === "uploaded") {
      console.log(data[i]);
      const listItem = document.createElement("li");
      listItem.innerHTML = `<div>
        <img src="${
          data[i].thumbnail_url
        }" onclick="viewer.switchScene({ contentId: '${
        data[i].content_id
      }'},${0})">
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'enhancement'},${0})">
          Enhance
        </button>
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'b_person'},${0})">
          Blur People
        </button>
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'p_cubic'},${0})">
          Cubic View
        </button>
      </div>`;
      leftList.append(listItem);
    }
  }

  // populate buttons on right pane list
  const rightList = document.getElementById("rightList");
  for (let i = 0; i < data.length; i++) {
    if (data[i].status === "uploaded") {
      console.log(data[i]);
      const listItem = document.createElement("li");
      listItem.innerHTML = `<div>
        <img src="${
          data[i].thumbnail_url
        }" onclick="viewer.switchScene({ contentId: '${
        data[i].content_id
      }'},${1})">
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'enhancement'},${1})">
          Enhance
        </button>
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'b_person'},${1})">
          Blur People
        </button>
        <button type="button" onclick="viewer.switchScene({ contentId: '${
          data[i].content_id
        }',transform:'p_cubic'},${1})">
          Cubic View
        </button>
      </div>`;
      rightList.append(listItem);
    }
  }
};
```

`fetchContent()` makes a async get request to the backend endpoint /content and returns content data and populates listing of images with transform buttons

### fetchToken()

```javascript
const fetchToken = async () => {
  const res = await fetch("/token", {
    method: "GET",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
  });
  const data = await res.text();
  return data;
};
```

`fetchToken()` makes a async get request to the backend endpoint /token and returns a token

### ui

```typescript
const ui = {
  cropping: {
    horizontalMargin: 15,
    verticalMargin: 75,
  },
  toolbar: {
    isHidden: false,
    cancelButton: {
      label: "Cancel",
    },
    checkButton: {
      label: "Done",
    },
    autoRotateButton: {
      isHidden: false,
      label: "Auto Rotate",
    },
    croppingButton: {
      isHidden: false,
      enterLabel: "Enter Cropping",
      exitLabel: "Exit Cropping",
    },
    annotationButton: {
      isHidden: false,
      enterLabel: "Use keyboard to annotate",
      exitLabel: "Exit keyboard annotation",
    },
    drawingButton: {
      isHidden: false,
      enterLabel: "Enter Drawing",
      exitLabel: "Exit Drawing",
    },
    fullScreenButton: {
      isHidden: false,
      enterLabel: "Enter Full Screen",
      exitLabel: "Exit Full Screen",
    },
    homePositionButton: {
      isHidden: false,
      label: "Home Position",
    },
    zoomControlButtons: {
      isHidden: false,
      zoomInLabel: "Zoom In",
      zoomOutLabel: "Zoom Out",
    },
  },
  tooltip: {
    isHidden: false,
  },
  verticalPanel: {
    isHidden: false,
    topMargin: 25,
  },
};
```

- Const ui is a parameter that is passed into the viewer object
- Allows definition and hiding/showing of ui buttons and functionalities

### onCropped

```typescript
const onCropped = async (url) => {
  const image = await fetch(url);
  const imageBlob = await image.blob();
  const imageURL = URL.createObjectURL(imageBlob);
  const downloadLink = document.createElement('a');
  downloadLink.href = imageURL;
  downloadLink.download = 'cropped_image.jpeg';
  document.body.appendChild(downloadLink);
  downloadLink.click();
  document.body.removeChild(downloadLink);
};
```

In the viewer, there is a crop function, that crop function passes in the cropped image URL to this callback function and downloads the cropped image to your computer.

### viewer

```typescript
const viewer = new RICOH360Viewer({
  divId: "viewer",
  onFetchToken: fetchToken,
  isCubemapEnabled: true,
  ui,
  onSelected: (index) => {
    console.log("selected", index);
  },
});

viewer.start({
  contentId: "<Content ID from fetchContent>",
  transform: "enhancement",
});

```

- Viewer is Ricoh embedded viewer API
- Viewer is initialized by specific a divId to inject the viewer UI into, an onFetchToken function for authorization and a CubeMapEnabled
- Viewer is started with a contentID (call `fetchContent()` function to get contentIDs) and a transform value ie. "enhancement" or "undefined"
