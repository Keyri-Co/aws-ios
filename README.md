---
title: Amazon Cognito
slug: EsjZ-amazon-cognito
createdAt: 2022-11-22T19:18:01.000Z
updatedAt: 2022-11-23T14:32:44.000Z
---

Incorporating Keyri QR login into a Cognito-based application involves sending the ID token and access token in your mobile app as the payload in the Keyri SDK once the QR code is scanned. Once the Keyri Widget on your web app emits the payload object, save the ID token and access token in browser storage (cookie, LocalStorage, etc.).

Usage notes:

1.  Keyri QR login will not require users to complete separate MFA steps on the web platform, since it is extending the logged-in session already on the user's mobile app that has previously completed MFA

2.  Your user needs to be already logged into the Banno platform in your app when the Keyri function is called once the QR code is scanned by your app

3.  You must have a Keyri account with a service registered under the domain name on which you will be showing the login QR code. Register for a Keyri account here: <https://app.keyri.com/sign-up>

# Web

First, add the Keyri Widget iFrame to your login page in a div of your choosing:&#x20;

1.  Serve KeyriQR.html (available [here](https://raw.githubusercontent.com/Keyri-Co/library-keyri-connect/main/KeyriQR.html)) from the same origin as your login page (e.g., a /public directory)

2.  *RECOMMENDED*: serve everything on your login's origin with the header X-Frame-Options: SAMEORIGIN (examples of how to do so [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options#examples))[﻿](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options#examples)﻿

3.  Embed an iframe in your login page in the desired DOM element with the path to your copy of KeyriQR.html as its `src`

```html
 ...
      <iframe 
      src="./KeyriQR.html"
      id="qr-frame" 
      frameborder="0"
      height="300" 
      width="300" 
      scrolling="no" 
      style="border: solid 5px white;" 
    ></iframe>
    ...
```

Now add handling for the payload, triggered by an event listener tuned for the Keyri Widget:

```javascript
  window.addEventListener("message", (evt) => {   
    // Ensure that the event is coming from the Keyri Widget 
    if(evt.data.keyri && evt.data.data && document.location.origin == evt.origin){

      const {data} = evt;

      if(!data.error){
        // Extract the ID token and access token strings from the payload
        const payload = JSON.parse(data.data)
        const idToken = payload.idToken;
        const accessToken = payload.accessToken;
        // Save them into, for example, LocalStorage
        localStorage.setItem("idToken", idToken);
        localStorage.setItem("accessToken", accessToken)
        // assuming your web app has a listener for the user's authorization state,
        // reload to advance the user to the logged-in view
        document.location.reload();
      } else {
        showErrorModal({title: "Uh Oh",body: "Helpful Error Message"});
      }
    }
  });
```

## Mobile

First, create a new iOS/Android application, and install the Amplify CLI using AWS's standard docs: [https://docs.amplify.aws/lib/project-setup/prereq/q/platform/](https://docs.amplify.aws/lib/project-setup/prereq/q/platform/ios/)

### Configure Amplify

In terminal, navigate to your project directory, then run the following commands:&#x20;

1.  `amplify init`

2.  `amplify add auth`

3.  `amplify push`

For all 3 steps, using the default options will be fine.

### Add Amplify and Keyri SDKs

Add the Amplify Swift Package: [https://github.com/aws-amplify/[amplify-swift](https://github.com/aws-amplify/amplify-swift)](https://github.com/aws-amplify/amplify-swift)

Utilize Amplify, and the **AWSCognitoAuthPlugin **plugins to bring in the required dependencies[](https://github.com/aws-amplify/amplify-swift)

For adding Keyri, add it via Cocoapods or SPM using our documentation

### Code

Below is a quick code snippet which displays how to get the credentials from Amplify and pass them through Keyri to the browser

```swift
do {
    let session = try await Amplify.Auth.fetchAuthSession()

    // Get user sub or identity id
    if let identityProvider = session as? AuthCognitoIdentityProvider {
        let usersub = try identityProvider.getUserSub().get()
        let identityId = try identityProvider.getIdentityId().get()
        print("User sub - \(usersub) and identity id \(identityId)")
    }

    // Get AWS credentials
    if let awsCredentialsProvider = session as? AuthAWSCredentialsProvider {
        let credentials = try awsCredentialsProvider.getAWSCredentials().get()
        // Do something with the credentials
        Keyri().easyKeyriAuth(username: [username], appKey: [AppKeyFromDevPortal], payload: credentials.secretKey)
    }

    // Get cognito user pool token

} catch let error as AuthError {
    print("Fetch auth session failed with error - \(error)")
} catch {
}
```

###

###
