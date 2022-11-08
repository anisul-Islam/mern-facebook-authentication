# facebook-authentication
- step1: get facebook client id 
    - https://developers.facebook.com/
    - register as developer
    - create new app as consumer
    - `npm install react-facebook-login`
    - change in package.json `"start": "HTTPS=true react-scripts start",`


```js
// Facebook.js
import React from "react";
import FacebookLogin from "react-facebook-login/dist/facebook-login-render-props";
import { loginWithFacebook } from "../services/AuthService";

const Facebook = () => {
  const responseFacebook = async (response) => {
    try {
    // 1. send the userID and accessToken to the server
      const result = await loginWithFacebook({
        userID: response.userID,
        accessToken: response.accessToken,
      });
      console.log("Facebook signin success: ", result);
    } catch (error) {
      console.log(error);
      console.log("Facebook signin error: ", error.response.data.message);
    }
  };

  return (
    <div className="pb-3">
      <FacebookLogin
        appId="3326779317563993"
        autoLoad={false}
        callback={responseFacebook}
        render={(renderProps) => (
          <button
            onClick={renderProps.onClick}
            className="btn btn-danger btn-lg btn-block"
          >
            <i className="fab fa-facebook p-2"></i>Login With Facebook
          </button>
        )}
      />
    </div>
  );
};

export default Facebook;


```

```js
export const loginWithFacebook = async (data) => {
  const response = await axios.post(
    `http://localhost:3030/api/facebook-login`,
    data
  );
  return response.data;
};
```
