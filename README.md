# facebook-authentication
- [doc](https://dev.to/quod_ai/how-to-integrate-facebook-login-api-into-your-react-app-33de)
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
- backend part
```js
// facebook
authRoutes.post("/facebook-login", handleFacebookLogin);
```

- controller part
- `npm install node-fetch@2` compatible for require
```js
const fetch = require("node-fetch");
const handleFacebookLogin = async (req, res) => {
  try {
    const { userID, accessToken } = req.body;

    // create the url for requesting userInfo
    const url = `http://graph.facebook.com/v2.11/${userID}/?fields=id,name,email&access_token=${accessToken}`;

    // make the request with fetch api
    const response = await fetch(url);
    const data = await response.json();
    const { name, email, id } = data;

    // check user with this email already exist or not
    const exsitingUser = await User.findOne({ email });
    if (exsitingUser) {
      console.log("user exist");
      // create a token for user
      const token = jwt.sign(
        { _id: exsitingUser._id },
        String(dev.app.jwtSecretKey),
        {
          expiresIn: "7d",
        }
      );

      // step 7: create user info
      const userInfo = {
        _id: exsitingUser._id,
        name: exsitingUser.name,
        email: exsitingUser.email,
        phone: exsitingUser.phone,
      };

      return res.json({ token, userInfo });
    } else {
      // if user does not exist create a new user
      // let create a dummy password
      let password = email + dev.app.jwtSecretKey;
      const newUser = new User({
        name,
        email,
        password,
      });

      const userData = await newUser.save();
      if (!userData) {
        return res.status(400).send({
          message: "user was not created with google",
        });
      }

      // if user is created
      const token = jwt.sign(
        { _id: userData._id },
        String(dev.app.jwtSecretKey),
        {
          expiresIn: "7d",
        }
      );

      // step 7: create user info
      const userInfo = {
        _id: userData._id,
        name: userData.name,
        email: userData.email,
        phone: userData.phone,
        isAdmin: userData.isAdmin,
      };

      return res.json({ token, userInfo });
    }

    res.send("facebook login successful");
  } catch (error) {
    res.status(500).send({
      message: error.message,
    });
  }
};
```
