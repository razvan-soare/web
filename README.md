# iProov Web SDK v2.0.0

## Introduction

The iProov Web SDK is the iProov client for web browser based authentication. It can be integrated in a number of ways to support your web journeys for onboarding and authentication.

Please note that to use the Web SDK you will require Service Provider credentials. You can set these up automatically after registering your company on our [portal](https://portal.iproov.com/login).

You will need to generate a token from your back-end to use with the iProov Web SDK. You can use the [API documentation](https://secure.iproov.me/docs.html) to make the relevant API calls and return the token to your front-end.

## Quick Links

- [iProov Web SDK v2.0.0](#iproov-web-sdk-v200)
  - [Introduction](#introduction)
  - [Quick Links](#quick-links)
  - [Supported Browsers](#supported-browsers)
  - [Integrating](#integrating)
    - [Backend](#backend)
    - [Frontend](#frontend)
  - [Android Web View](#android-web-view)
    - [Java Example](#java-example)
    - [Kotlin Example](#kotlin-example)
  - [Script Tag](#script-tag)
  - [NPM Package](#npm-package)
    - [Setup](#setup)
    - [Vanilla JavaScript](#vanilla-javascript)
    - [jQuery](#jquery)
    - [Angular v7](#angular-v7)
    - [React v16](#react-v16)
    - [Vue v2](#vue-v2)
  - [Events](#events)
    - [Details](#details)
    - [Listeners](#listeners)
  - [Customisation](#customisation)
    - [Slots](#slots)
    - [Options](#options)
      - [Allow Landscape](#allow-landscape)
      - [Assets URL](#assets-url)
      - [Base URL](#base-url)
      - [Colours](#colours)
      - [Custom Title](#custom-title)
        - [Examples](#custom-title-examples)
      - [Logo](#logo)
      - [Prefer App](#prefer-app)
      - [Show Countdown](#show-countdown)
      - [Kiosk Mode](#kiosk-mode)
    - [HTML](#html)
    - [JavaScript](#javascript)
    - [Language Support](#language-support)
      - [Default Language](#default-language)
      - [Custom Language](#custom-language)
    - [Language Code Examples](#language-code-examples)
      - [ES6 static JSON object](#es6-static-json-object)
      - [ES6 async/await fetch file from local or external source](#es6-asyncawait-fetch-file-from-local-or-external-source)
      - [React 16, Axios, ES6 async/await fetch file from local or external source](#react-16-axios-es6-asyncawait-fetch-file-from-local-or-external-source)
      - [Angular 7, HttpClient, ES6 fetch file from local filesystem](#angular-7-httpclient-es6-fetch-file-from-local-filesystem)
      - [Angular 7, HttpClient, ES6 fetch file from external source](#angular-7-httpclient-es6-fetch-file-from-external-source)

## Supported Browsers

iProov's Web SDK makes use of the following technologies:

- [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)
- [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [WebAssembly](https://webassembly.org/)

It also requires a front facing camera with permission granted to use it.

Developers can use the `IProovSupport` check component to ensure their users have the correct hardware and software to
use the Web SDK. If the user device is unsupported, the integrator can send the user down a separate, manual or
password-based journey.

`IProovSupport` is a slim and separate component to the main `IProovMe` web component.

To benefit from tree-shaking in a module-based build environment you can use the named import:

```ecmascript 6
// Just load the support component:
import { IProovSupport } from "@iproov/web"
const optionalLogger = console;
const supportChecker = new IProovSupport(optionalLogger);
```

For script tag integrations, `IProovSupport` is available on the window object once included:

```ecmascript 6
const supportChecker = new IProov.IProovSupport();
```

`IProovSupport` can check just for the required APIs on the user's browser, using either event or Promise based APIs:

```ecmascript 6
const supportChecker = new IProovSupport();
supportChecker.addEventListener("check", ({ supported, granted }) => {
    if (supported === false) {
         // go to fallback UX
    }
    if (supported && granted) {
        // full permission and granted, we can definitely iProov!
    }
    if (supported && granted === null) {
        // browser API support, but we haven't run a permission check (see checkWithPermission)
    }
    if (supported && granted === false) {
        // browser API support, but camera access denied - try again or advise user before proceeding
    }
});
const { supported, granted } = await supportChecker.check();
```

...or to carry out a complete lightweight check for camera access with user interaction, this can pre-set the required
permissions for iProoving, save some bandwidth, and provide a cleaner user journey if camera access isn't possible:

```ecmascript 6
const supportChecker = new IProovSupport();
document.querySelector("#check-button").addEventListener("click", async () => {
  const { supported, granted } = await supportChecker.checkWithPermission()
});
```

If `.checkWithPermission()` is run and permission is granted and cached within the browser, future interaction is often not
required and we can tell if permission has been granted using a soft `.check()`.

Note that browsers have varying regimes to protect against device fingerprinting and to ensure user privacy. Repeated
calls to `getUserMedia` or the Permissions API can result in prompt blockage, or the redaction of media devices, which
can return inaccurate results. Our advice is therefore to avoid running multiple checks, in quick succession, on the
same page. Therefore, please avoid automatic or accidental repeat calls to `check` or `checkWithPermission`, especially
without user interaction.

The following events can be emitted from `IProovSupport`:

```ecmascript 6
const supportChecker = new IProovSupport();
const onCheckResult = ({ supported, granted, tests }) => ({});
const onUnsupported = ({ supported, tests }) => ({});
const onPermissionWasGranted = ({ tests }) => ({});
const onPermissionWasDenied = ({ tests }) => ({});
supportChecker.addEventListener('check', onCheckResult);
supportChecker.addEventListener('unsupported', onUnsupported);
supportChecker.addEventListener('granted', onPermissionWasGranted);
supportChecker.addEventListener('denied', onPermissionWasDenied);
// The `tests` object consists of the following options:
// null if unchecked, true if supported, false if not supported:
const possibleTests = {
    videoInput: null,
    wasm: null,
    userMedia: null,
    mediaStreamTrack: null,
    frontCamera: null,
    fullScreen: null,
    webgl: null,
}
```

Using the support checker is the best and canonical way to detect whether a browser is supported.

The below list is a best-effort representation of minimum feature support across browsers:

### Desktop browser support:

| ![Chrome](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/chrome_1-11/chrome_1-11.svg) | ![Firefox](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/firefox_1.5-3/firefox_1.5-3.svg) | ![Opera](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/opera_10-14/opera_10-14.svg) | ![Edge](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/edge/edge.svg) | ![Safari](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/safari-ios/safari-ios.svg) | ![SamsungInternet](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/samsung-internet_5/samsung-internet_5.svg) |
| ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 57+ ✔                                                                                                       | 53+ ✔                                                                                                            | 43+ ✔                                                                                                      | 16+ ✔                                                                               | 11+ ✔                                                                                             | 8.2 ✔                                                                                                                              |

### iOS browser support

| ![Safari](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/safari-ios/safari-ios.svg) | ![Chrome](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/chrome_1-11/chrome_1-11.svg) | ![Firefox](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/firefox_1.5-3/firefox_1.5-3.svg) |
| ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 11+ ✔                                                                                             | Not supported, due no camera support. Bug tracker suggests coming soon.                                     | Not supported, due no camera support. Bug tracker suggests coming soon.                                          |

### Android browser support

| ![Chrome](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/chrome_1-11/chrome_1-11.svg) | ![Firefox](https://cdnjs.cloudflare.com/ajax/libs/browser-logos/51.0.17/archive/firefox_1.5-3/firefox_1.5-3.svg) |
| ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 80+                                                                                                         | 68+                                                                                                              |

> If the device attempting to iProov doesn't meet the minimum requirements, the `unsupported` event is emitted. See [#events](#events) for more details.

## Integrating

### Backend

To make use of this SDK you will require integration with the back-end iProov service. To access credentials and an integration guide, please visit [portal.iproov.com](https://portal.iproov.com).

When starting an iProov transaction (or claim), you would first need to generate an [enrolment](https://secure.iproov.me/docs.html#operation/userEnrolServerToken) or [verification](https://secure.iproov.me/docs.html#operation/userVerifyServerToken) token, which can be done as part of the page load or with AJAX. You would then need to pass the token to your frontend to initialise the iProov Web SDK.

After receiving the result from the SDK, you must then confirm its authenticity by calling the validate API from your backend on page submission or with AJAX.

> The REST API should always be called from your backend so as to never expose your secret to your client application.

> Always call the validate API from your backend after receiving a result; never trust the result provided directly from the HTML client.

### Frontend

Once an iProov token has been generated by your backend, you should pass it to the Web SDK using one of the [integration methods](#integrating) described below.

After a successful iProov, the result must be validated by the backend before granting the user any privileges.

In these examples, the token is generated and validated using AJAX (in the register/login demo) and form submission (for the integration examples) by passing the following data to the backend:

- **user_id** - the email address provided by the user
- **action** - the specific endpoint to call:
  - _token_ - generate a unique 64 character token for the client
  - _validate_ - check the validity of the result provided by the client

## Android Web View

The iProov Web SDK is compatible with Android webview based apps. For the component to work as expected, the app _must_ correctly allow fullscreen mode, otherwise the user interface is compromised.

Below are examples on how to ensure fullscreen is allowed and configured correctly inside your Android app:

### Java Example

```java
package com.iproov.webview;

import android.Manifest;
import android.content.pm.ActivityInfo;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.view.View;
import android.webkit.PermissionRequest;
import android.webkit.WebChromeClient;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.FrameLayout;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private static final int CAMERA_PERMISSIONS_REQUEST_CODE = 12345;
    private static final String URL = "https://demo.iproov.com";
    private WebView webView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        WebView.setWebContentsDebuggingEnabled(true);

        webView = findViewById(R.id.webView);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.getSettings().setDomStorageEnabled(true);
        webView.setWebViewClient(new WebViewClient());
        //This is really important part. Our custom implementation of chrome client will allow webView to go full screen,
        //and will restore the same state of host app after we exit from full screen mode.
        webView.setWebChromeClient(new WebChromeClientFullScreen());

        final int cameraPermission = ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA);
        if (cameraPermission == PackageManager.PERMISSION_GRANTED) {
            webView.loadUrl(URL);
        } else {
            requestPermissions();
        }
    }

    @Override
    public void onBackPressed() {
        if (webView.canGoBack()) {
            webView.goBack();
        } else {
            super.onBackPressed();
        }
    }

    private void requestPermissions() {
        ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.CAMERA},
                CAMERA_PERMISSIONS_REQUEST_CODE);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == CAMERA_PERMISSIONS_REQUEST_CODE && grantResults.length > 0) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED)
                webView.loadUrl(URL);
        }
    }

    private  class WebChromeClientFullScreen extends WebChromeClient {
        private View customView = null;
        private CustomViewCallback customViewCallback = null;
        int originalOrientation = ActivityInfo.SCREEN_ORIENTATION_UNSPECIFIED;
        int originalVisibility = View.INVISIBLE;

        @Override
        public void onPermissionRequest(PermissionRequest request) {
           if (request != null) request.grant(request.getResources());
        }

        /**
         * Callback will tell the host application that the current page would
         * like to show a custom View in a particular orientation
         */
        @Override
        public void onShowCustomView(View view, CustomViewCallback callback) {
            //If we have custom view, that means that we are already in full screen, and need to go to original state
            if (customView != null) {
                onHideCustomView();
                return;
            }
            //going full screen
            customView = view;
            //We need to store there parameters, so we can restore app state, after we exit full screen mode
            originalVisibility = getWindow().getDecorView().getWindowSystemUiVisibility();
            originalOrientation = getRequestedOrientation();
            ((FrameLayout)getWindow().getDecorView()).addView(customView, new FrameLayout.LayoutParams(-1, -1));
            getWindow().getDecorView().setSystemUiVisibility(3846 | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
        }

        /**
         * Callback will tell the host application that the current page exited full screen mode,
         * and the app has to hide custom view.
         */
        @Override
        public void onHideCustomView() {
            ((FrameLayout)getWindow().getDecorView()).removeView(customView);
            customView = null;
            //Restoring aps state, as it was before we go to full screen
            getWindow().getDecorView().setSystemUiVisibility(originalVisibility);
            setRequestedOrientation(originalOrientation);
            if (customViewCallback != null) customViewCallback.onCustomViewHidden();
            customViewCallback = null;
        }
    }
}

```

### Kotlin Example

```kotlin
package com.iproov.webview

import android.Manifest
import android.content.pm.ActivityInfo
import android.content.pm.PackageManager
import android.os.Bundle
import android.view.View
import android.webkit.PermissionRequest
import android.webkit.WebChromeClient
import android.webkit.WebView
import android.webkit.WebViewClient
import android.widget.FrameLayout
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import kotlinx.android.synthetic.main.activity_main.*
class MainActivityKotlin : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        WebView.setWebContentsDebuggingEnabled(true)

        webView.settings.javaScriptEnabled = true
        webView.settings.domStorageEnabled = true
        webView.webViewClient = WebViewClient()
        webView.webChromeClient = object : WebChromeClient() {
            var customView: View? = null
            var callback: CustomViewCallback? = null
            var originalOrientation: Int = ActivityInfo.SCREEN_ORIENTATION_UNSPECIFIED
            var originalVisibility: Int = View.INVISIBLE
            override fun onPermissionRequest(request: PermissionRequest?) {
                request?.grant(request.resources);
            }

            override fun onShowCustomView(view: View?, callback: CustomViewCallback?) {
                if (customView != null) {
                    onHideCustomView()
                    return
                }
                customView = view
                originalVisibility = window.decorView.systemUiVisibility
                originalOrientation = requestedOrientation
                (window.decorView as FrameLayout).addView(this.customView, FrameLayout.LayoutParams(-1, -1))
                window.decorView.systemUiVisibility = 3846 or View.SYSTEM_UI_FLAG_LAYOUT_STABLE

            }

            override fun onHideCustomView() {
                (window.decorView as FrameLayout).removeView(customView)
                customView = null
                window.decorView.systemUiVisibility = originalVisibility
                requestedOrientation = originalOrientation
                callback?.onCustomViewHidden()
                callback = null
            }
        }

        val cameraPermission: Int = ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.CAMERA
        )

        if (cameraPermission == PackageManager.PERMISSION_GRANTED)
            webView.loadUrl("https://demo.iproov.com/")
        else
            requestPermissions()
    }

    override fun onBackPressed() {

        if (webView.canGoBack()) {
            webView.goBack()
        } else {
            super.onBackPressed()
        }
    }

    fun requestPermissions() {

        ActivityCompat.requestPermissions(
            this,
            arrayOf(Manifest.permission.CAMERA),
            12345
        )
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)

        if (requestCode == 12345 && grantResults.size > 0) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                webView.loadUrl("https://demo.iproov.com/")
            }
        }
    }

}
```

## Script Tag

The simplest and fastest method of integrating with the iProov Web SDK is to reference our generated client javascript file within your page as shown below. You would then use either the [Vanilla JavaScript](#vanilla-javascript) or [jQuery](#jquery) methods depending on your page's setup.

```html
<script src="https://client.iproov.app/v2"></script>
```

## NPM Package

The npm package `@iproov/web` allows for integration of the iProov Web SDK. It makes use of the [Web Components](https://www.webcomponents.org/introduction) APIs which are supported by most modern browsers and uses the [Polymer Project](https://www.polymer-project.org) to add support where they are not yet available.

### Setup

Install the package as a dependency. You can use [Yarn](https://yarnpkg.com/lang/en/) or [NPM](https://www.npmjs.com/get-npm) to do this.

```
yarn add @iproov/web
```

```
npm i @iproov/web --save
```

After you have installed the `@iproov/web` package, you must then include the code into your project ideally at the root of your application:

```javascript
require("@iproov/web")
```

It's as simple as that to include the iProov Web SDK. You now need to inject the web component by one of the [integration methods](#integrating) shown below.

To include the iProov Web SDK on your page, complete one of the following steps.

### Vanilla JavaScript

Inject the `<iproov-me>` element into the page with your token:

```javascript
window.addEventListener("WebComponentsReady", function(event) {
  const iProovMe = document.createElement("iproov-me")
  iProovMe.setAttribute("token", "***YOUR_TOKEN_HERE***")
  document.getElementById("your-container-id").appendChild(iProovMe)
})
```

### jQuery

The HTML can also be injected directly onto the page as in this jQuery example:

```javascript
window.addEventListener("WebComponentsReady", function(event) {
  $("#your-container-id").append($('<iproov-me token="***YOUR_TOKEN_HERE***"></iproov-me>'))
})
```

### Angular v7

Integrating with an Angular project has a few steps that need to be implemented to work out of the box. You will first need to create a custom Angular component which will allow you to inject the iProov Web SDK into your project.

> Note that the iProov Web SDK has only been tested with Angular v7 and may not be compatible with earlier versions.

The example below shows how to use iProov within your application and an example integrating with some of the events available for you to use. [View the full list of events](#events)

`iproov-component.html`

```html
<!-- <iproov-me> web component is injected here-->
<div id="iproov-container"></div>
```

`iproov.component.ts`

```javascript
import { Component, OnInit } from "@angular/core"
import "@iproov/web"

@Component({
  selector: "app-iproov", // component name used to inject iproov into app <app-iproov>
  template: "./iproov-component.html", // iproov-component.html as shown above
})
export class IproovComponent implements OnInit {
  token = "***YOUR_TOKEN_HERE***" // replace with your generated token
  ngOnInit() {
    //- Wait for web components to be ready before trying to use them
    window.addEventListener("WebComponentsReady", () => {
      const iProovMe = document.createElement("iproov-me")

      //- Bind any events before injecting into the page (must be before to work)
      iProovMe.addEventListener("unsupported", () => {
        console.warn("unsupported event was fired!")
      })

      //- Set your generated token
      iProovMe.setAttribute("token", this.token)

      //- Inject any custom slots
      iProovMe.innerHTML = `
      <div slot="button">
          <button type="button">
              Scan Face
          </button>
      </div>`

      //- An alternative way of injecting a custom slots
      const buttonSlot = document.createElement("button")
      buttonSlot.setAttribute("slot", "button")
      buttonSlot.innerHTML = `Scan Face`
      iProovMe.appendChild(buttonSlot)

      //- Inject iproov element into your page
      document.getElementById("iproov-container").insertAdjacentElement("beforebegin", iProovMe)
    })
  }
}
```

You will then need to inject the component into your application and also tell Angular that we are using custom web components. The code below is from the `angular-cli` default generated app.

`app.module.ts`

```javascript
import { BrowserModule } from "@angular/platform-browser"
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from "@angular/core" // import Custom Elements from Angular

import { AppRoutingModule } from "./app-routing.module"
import { AppComponent } from "./app.component"
import { IproovComponent } from "./iproov.component" // import the iproov component we just created

@NgModule({
  declarations: [
    AppComponent,
    IproovComponent, // include the iproov component
  ],
  imports: [BrowserModule, AppRoutingModule],
  providers: [],
  bootstrap: [AppComponent],
  schemas: [
    CUSTOM_ELEMENTS_SCHEMA, // define the custom element schema
  ],
})
export class AppModule {}
```

We then need to reference our new component in our app. The code below is again from the `angular-cli` default generated app.

`app.component.html`

```html
<div style="text-align:center">
  <app-iproov></app-iproov>
  <h1>
    Welcome to {{ title }}!
  </h1>
</div>
```

### React v16

Integrating with React is pretty straight forward. First of all, include the `@iproov/web` package into your application and then include the <iproov-me> web component with your generated token. If you want to pass any [custom slots](#Customisation) in, use the same method as shown below. The example below is taken from the [create-react-app](https://github.com/facebook/create-react-app) tool.

```javascript
import React, { Component } from 'react';

require('@iproov/web'); // includes the @iproov/web client into your app

class App extends Component {
  render() {
    return (
      <div className="App">
        <!-- replace with your generated token -->
        <iproov-me token="***YOUR_TOKEN_HERE***">
            <!-- add any custom slots here -->
            <div slot="button">
                <button type="button">
                    My Custom Button Text
                </button>
            </div>
        </iproov-me>
      </div>
    );
  }
}

export default App;

```

### Vue v2

Integrating with Vue JS is super easy and can be done with just a few lines of code. The example below was taken from using [Vue create hello-world](https://cli.vuejs.org/guide/creating-a-project.html). Include the `<iproov-me>` web component and replace with your generated token. Then all you need to do is include the `@iproov/web` client.

```html
<template>
  <div class="hello">
    <iproov-me token="***YOUR_TOKEN_HERE***"
      ><!-- replace with your generated token -->
      <!-- add any custom slots here -->
      <div slot="button">
        <button type="button">Scan Face <small>Vue Example</small></button>
      </div>
    </iproov-me>
  </div>
</template>

<script>
  require("@iproov/web") // include the @iproov/web client
  export default {
    name: "HelloWorld",
    props: {
      msg: String,
    },
  }
</script>
```

> Note that JavaScript methods are run after the **WebComponentsReady** event to ensure that the client is ready to be injected.

## Events

There are a set of [CustomEvents](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent) that can be used to integrate bespoke functionality. Events are bound directly to the `<iproov-me>` element after the [WebComponentsReady](https://www.webcomponents.org/polyfills#the-webcomponentsready-event) event.

### Details

Every event has a [detail](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/detail) property which contains data relating to that event. The token is present for every event:

```json
{
  "token": "Your Generated Token"
}
```

The available events are detailed below with any extra properties that are supplied:

| Event                 | Extra Properties                 | Description                                                          |
| --------------------- | -------------------------------- | -------------------------------------------------------------------- |
| **ready**             | None                             | iProov has initialised successfully and has camera permission        |
| **started**           | None                             | The user has started iProov by launching into fullscreen             |
| **aborted**           | _feedback, reason_               | The user has aborted iProov by exiting fullscreen                    |
| **streamed**          | None                             | The user has finished streaming and the client has exited fullscreen |
| **progress**          | _percentage, message_            | iProov has published a progress update for the authentication        |
| **passed**            | _type, passed_                   | The authentication was successful so the result can now be validated |
| **failed**            | _type, passed, feedback, reason_ | The authentication was unsuccessful so the user needs to try again   |
| **error**             | _feedback, reason_               | iProov encountered an error while processing the authentication      |
| **unsupported**       | _feedback, reason_               | The browser does not support using iProov                            |
| **permission**        | None                             | Camera permission is unknown and not blocked, show permission screen |
| **permission_denied** | None                             | The user has blocked access to the camera                            |

All possible properties of the event's **detail** property are described below:

| Property             | Events                                | Description                                                |
| -------------------- | ------------------------------------- | ---------------------------------------------------------- |
| **token**            | All                                   | The token associated with the authentication attempt       |
| **type** (†)         | _passed, failed_                      | The type of authentication (enrol, verify or id_match)     |
| **passed**           | _passed, failed_                      | Boolean value whether the result passed or failed          |
| **percentage**       | _progress_                            | A percentage (between 0 and 100) representing the progress |
| **message**          | _progress_                            | A user-friendly description of the current progress stage  |
| **feedback**         | _aborted, failed, error, unsupported_ | A fixed feedback code for making logical decisions         |
| **reason**           | _aborted, failed, error, unsupported_ | An English description of the reason for the event         |
| **is_native_bridge** | All                                   | Boolean value if event originates from the native bridge   |

† - Not available when running with the `prefer_app` setting in native bridge mode.

In the case of the **aborted**, **failed**, **error** and **unsupported** events, the _feedback_ code can be used for dealing with special cases and the _reason_ can be displayed to the user. The following are some of the possible responses:

| Feedback                              | Reason                                                |         Event |
| ------------------------------------- | ----------------------------------------------------- | ------------: |
| **client_browser**                    | The browser is not supported                          | _unsupported_ |
| **fullscreen_change**                 | Exited fullscreen without completing iProov           |     _aborted_ |
| **ambiguous_outcome**                 | Sorry, ambiguous outcome                              |      _failed_ |
| **user_timeout**                      | Sorry, your session has timed out                     |      _failed_ |
| **lighting_flash_reflection_too_low** | Ambient light too strong or screen brightness too low |      _failed_ |
| **lighting_backlit**                  | Strong light source detected behind you               |      _failed_ |
| **lighting_too_dark**                 | Your environment appears too dark                     |      _failed_ |
| **lighting_face_too_bright**          | Too much light detected on your face                  |      _failed_ |
| **motion_too_much_movement**          | Please keep still                                     |      _failed_ |
| **motion_too_much_mouth_movement**    | Please do not talk while iProoving                    |      _failed_ |
| **client_config**                     | There was an error with the client configuration      |       _error_ |
| **client_api**                        | There was an error calling the API                    |       _error_ |
| **client_camera**                     | There was an error getting video from the camera      |       _error_ |
| **client_stream**                     | There was an error streaming the video                |       _error_ |
| **client_error**                      | An unknown error occurred                             |       _error_ |
| **server_abort**                      | The server aborted the claim before iProov completed  |       _error_ |
| **invalid_token**                     | The provided token has already been claimed           |       _error_ |
| **network_problem**                   | Sorry, network problem                                |       _error_ |

### Listeners

It is recommended that you listen for at least the **ready** and **unsupported** events as one of them will always be called so you can determine if iProov is supported or not:

```javascript
document.querySelector("iproov-me").addEventListener("ready", function(event) {
  console.log("iProov is ready with token " + event.detail.token)
})
document.querySelector("iproov-me").addEventListener("unsupported", function(event) {
  console.warn("iProov is not supported: " + event.detail.reason)
})
```

> The **unsupported** event will be called by older browsers that support [CustomEvents](https://caniuse.com/customevent) (browsers released in 2011 and after - e.g. IE9+). If you want to degrade nicely in even older browsers, you should implement you own test like [checking for CustomEvent](https://stackoverflow.com/questions/20956964/detect-working-customevent-constructor).

As all of the event **details** follow the same structure, they can be handled by a single function if desired:

```javascript
const iProovMe = document.querySelector("iproov-me")
iProovMe.addEventListener("ready", iProovEvent)
iProovMe.addEventListener("started", iProovEvent)
iProovMe.addEventListener("aborted", iProovEvent)
iProovMe.addEventListener("streamed", iProovEvent)
iProovMe.addEventListener("progress", iProovEvent)
iProovMe.addEventListener("passed", iProovEvent)
iProovMe.addEventListener("failed", iProovEvent)
iProovMe.addEventListener("error", iProovEvent)
iProovMe.addEventListener("unsupported", iProovEvent)
iProovMe.addEventListener("permission", iProovEvent)
iProovMe.addEventListener("permission_denied", iProovEvent)

function iProovEvent(event) {
  switch (event.type) {
    case "aborted":
    case "error":
    case "unsupported":
    case "permission":
    case "permission_denied":
      console.warn("iProov " + event.type + " - " + event.detail.reason)
      break
    case "progress":
      console.info(event.detail.message + " (" + event.detail.progress + "%)")
      break
    case "passed":
    case "failed":
      console.log("iProov " + event.detail.type + " " + event.type)
      break
    default:
      console.log("iProov " + event.type)
  }
}
```

> If using [jQuery](https://jquery.com), you can attach to all the events in one go:

```javascript
$("iproov-me").on("ready started aborted streamed progress passed failed error unsupported", iProovEvent)
```

Further examples of event usage can be found in [examples.js](./examples.js) and [script.js](./script.js).

## Customisation

### Slots

To allow language keys to be dynamically applied to slots special class names must be applied to your slots when customising. Headings must have `.iproov-lang-heading` and terms (message, reason etc) must have `.iproov-lang-term`.

> `h3` tags and the `div` element are allowed without classes, this functionality has been deprecated and will be removed in `2.1.0`

```html
<div slot="passed">
  <h3 class="iproov-lang-heading">Passed</h3>
  <div class="iproov-lang-term">You have iProoved successfully</div>
</div>
```

> When customising any slots with button elements the type must be set to button

Visual customisation can be achieved with [templates and slots](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots) using the [Shadow DOM API](https://webkit.org/blog/4096/introducing-shadow-dom-api/). The following [slots](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot) can be used with the `<iproov-me>` web component and have associated [events](#events):

| Slot                  | Description                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| **button**            | Element that a user taps or clicks on to launch into fullscreen and start iProov               |
| **ready**             | Screen displayed to the user when the component is ready to start the main iProov user journey |
| **aborted**           | Screen displayed to the user when they exit fullscreen before iProoving                        |
| **progress**          | Screen displayed to the user when streaming has completed and iProov is processing the result  |
| **passed**            | Screen displayed to the user when the user passed iProov                                       |
| **failed**            | Screen displayed to the user when the user failed iProov                                       |
| **error**             | Screen displayed to the user in the event of an error                                          |
| **unsupported**       | Screen displayed to the user when their browser is not supported                               |
| **permission_denied** | Screen displayed to the user when camera permission has been blocked                           |
| **grant_permission**  | Screen displayed to the user when camera permission is unknown and not blocked                 |
| **grant_button**      | Element that user taps or clicks to grant camera permission                                    |
| **no_camera**         | Screen displayed to the user when there is no camera                                           |

Slots can be embedded as HTML or injected with JavaScript.

### Options

#### Assets URL

Currently, external resources are loaded from our CDN at iproov.app. This may cause issues with firewall rules and custom reverse proxy configurations and therefore this option is available to use.

These resources include WASM files, UI assets, animations, and web workers relevant to encoding and face finding on the client side.

Some of these files cannot be packaged as part of a JS bundle, and in some cases it makes sense for us to be able to ship updates without requiring code changes the integrator's end.

```html
<iproov-me
  token="***YOUR_TOKEN_HERE***"
  assets_url="https://assets.rp.yourdomain.com"
  base_url="https://your.rp.yourdomain.com"
>
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Base URL

You can change the backend server you are attempting to iProov against by passing the `base_url` property. This needs to point to the same platform used for generating tokens (defaults to the EU platform if not defined). Reverse proxies are supported and a custom path to the WebSocket endpoint can be used, for example: `https://custom.domain.com/custom-path/socket.io/v2/`

```html
<iproov-me token="***YOUR_TOKEN_HERE***" base_url="https://eu.rp.secure.iproov.me">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Prefer App

Use this collection of settings to handle launching a native app containing the iProov SDK on mobile devices.

- `prefer_app`
- `prefer_app_options`

The `prefer_app` setting converts the scan button into an app launch URL which will launch the iProov app or iProov SDK when within a WebView. The following values are allowed and multiple can be used when separated by a comma:

- always
- ios
- android
- ios-webview
- android-webview

```html
<iproov-me token="***YOUR_TOKEN_HERE***" prefer_app="ios,ios-webview">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

Base64 encoding is required to work around HTML attributes.

The `prefer_app_options` setting accepts a base64 encoded JSON object of iProov native SDK options:

```js
iProovMe.setAttribute(
  "prefer_app_options",
  btoa(JSON.stringify({ ui: { scan_line_disabled: true, filter: "classic" } }))
)
```

#### Allow Landscape

Mobile devices are by default prevented from iProoving while in landscape. This feature can be disabled by passing `allow_landscape` `true` with your component as shown below.

```html
<iproov-me token="***YOUR_TOKEN_HERE***" allow_landscape="true">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Show Countdown

By setting `show_countdown` to `true`, a countdown will be shown to the user before scanning actually starts. If this is set to `false`, when the users face becomes properly aligned the scanner will start in 2 seconds as long as the users face is still properly aligned. By default, `show_countdown` is `false`. The example below shows how to enable the countdown.

```html
<iproov-me token="***YOUR_TOKEN_HERE***" show_countdown="true">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Colours

You can customise the look and feel of the main layout by changing the following options. You can pass a literal value i.e. `red`, RGB i.e. `rgb(230, 245, 66)` or a hex value i.e. `#e6f542`.

```javascript
loading_tint_color = "#5c5c5c" // The app is connecting to the server or no face found. Default: grey (#5c5c5c)
not_ready_tint_color = "#f5a623" // Cannot start iProoving until the user takes action (e.g. move closer, etc). Default: orange (#f5a623)
ready_tint_color = "#01bf46" // Ready to start iProoving. Default: green (#01bf46)
```

The example below changes the default grey no face to `#4293f5` (blue), giving feedback like "Move Closer" to red `rgb(245, 66, 66)` and starting to `purple`.

```html
<iproov-me
  token="***YOUR_TOKEN_HERE***"
  loading_tint_color="#4293f5"
  not_ready_tint_color="rgb(245, 66, 66)"
  ready_tint_color="purple"
>
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Logo

You can use a custom logo by simply passing a relative link or absolute path to your logo. Ideally, the logo would be in an SVG format for scaling but you can use any web safe image format. If you don't pass a logo, the iProov logo will be shown by default. If you do not want a logo to show pass the `logo` attribute as `null`.

```html
<iproov-me token="***YOUR_TOKEN_HERE***" logo="https://www.waterloobank.co.uk/assets/img/logo.svg">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Kiosk Mode

Note this setting enables a feature which is in alpha and still under active development.

For deploying iProov on tablets or fixed hardware such as laptops and desktop devices. Enables snap to face and increases matchable range.

Set to true to enable; omit the setting to keep disabled.

A known issue is that kiosk mode currently has display issues in portrait mode, this will be resolved in a later release.

```html
<iproov-me token="***YOUR_TOKEN_HERE***" kiosk_mode="true">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
</iproov-me>
```

#### Custom Title

Specify a custom title to be shown. Defaults to show an iProov-generated message. Set to empty string "" to hide the message entirely. You can also pass in `%@` characters which will be mapped in this order to `type` (Enrol or Verify), `user_name` (the user_name you passed when generating the token), `sp_name` (the service provider you used to generate the token).

> Note: iProov-generated messages are passed through our translators. Passing a custom title will prevent this and you will need to provide the translated version.

##### Custom Title Examples:

```html
<!-- Set the title to a plain string -->
<iproov-me token="***YOUR_TOKEN_HERE***" custom_title="iProov Ltd" />

<!-- Hide the title, empty string, "null" or "false" can be used. Must be a string! -->
<iproov-me token="***YOUR_TOKEN_HERE***" custom_title="" />

<!-- Build dynamic string with type, user_name and sp_name-->
<!-- i.e. Below would generate "Enrol AS andrew@iproov.com TO iProov" -->
<iproov-me token="***YOUR_TOKEN_HERE***" custom_title="%@ AS %@ TO %@" />
```

### HTML

The simplest way to add a slot is to include it within the `<iproov-me>` HTML tag:

```html
<iproov-me token="***YOUR_TOKEN_HERE***">
  <div slot="ready">
    <h1 class="iproov-lang-heading">Ready to iProov</h1>
  </div>
  <div slot="button">
    <button type="button">Scan Face</button>
  </div>
</iproov-me>
```

> In order to style and manipulate your slots, you can add custom classes and IDs which can be accessed from your CSS and JavaScript.

### JavaScript

You can also build up the slots with JavaScript before injecting the Web Component:

```javascript
window.addEventListener("WebComponentsReady", function(event) {
  const iProovMe = document.createElement("iproov-me")
  iProovMe.setAttribute("token", "***YOUR_TOKEN_HERE***")
  const ready = document.createElement("div")
  ready.setAttribute("slot", "ready")
  ready.innerHTML = '<h1 class="iproov-lang-heading">Register your face</h1>'
  iProovMe.appendChild(ready)

  const button = document.createElement("button")
  button.setAttribute("slot", "button")
  button.innerText = "Start face scan..."
  iProovMe.appendChild(button)

  document.getElementById("your-container-id").appendChild(iProovMe)
})
```

With [jQuery](https://jquery.com), the entire Web Component can be injected with slots using HTML syntax, for example:

```javascript
window.addEventListener("WebComponentsReady", function(event) {
  const iProovMe = $('<iproov-me token="***YOUR_TOKEN_HERE***"></iproov-me>')

  iProovMe.append('<div slot="ready"><h1 class="iproov-lang-heading">Register your face</h1></div>')
  iProovMe.append('<button type="button" slot="button">Start face scan...</button>')

  $("#your-container-id").append(iProovMe)
})
```

### Language Support

The iProov Web SDK supports the customisation of languages through JSON configurations. All language files have the same keys and only the values of those keys are what will be shown.

#### Default Language

The default language is set to `en`.

> [You can view the default language file here which has all keys and translations.](https://github.com/iProov/web/blob/master/iproov-en.json)

#### Custom Language

You can customise the language by supplying the `language` key with your iProov component. The keys value must be valid JSON and passed as a string. This is then converted and merged with the default language overriding any given keys. See below for [language code examples](#language-code-examples).

### Language Code Examples

#### ES6 static JSON object

```javascript
window.addEventListener("WebComponentsReady", event => {
  const iProovMe = document.createElement("iproov-me")
  iProovMe.setAttribute("token", "***YOUR_TOKEN_HERE***")

  const customLanguage = `{
    "iproov_success": "You passed!",
    "prompt_loading": "Its loading"
  }`
  element.setAttribute("language", customLanguage)

  // inject iproov element into page
  document.getElementById("your-container-id").appendChild(iProovMe)
})
```

#### ES6 async/await fetch file from local or external source

```javascript
window.addEventListener("WebComponentsReady", async event => {
  async function getLanguage(path) {
    const response = await fetch(path)
    const language = await response.text()

    return language
  }

  const iProovMe = document.createElement("iproov-me")
  iProovMe.setAttribute("token", "***YOUR_TOKEN_HERE***")

  const languageFile = "" // local or external path to language file
  const customLanguage = await getLanguage(languageFile)
  element.setAttribute("language", customLanguage)

  // inject iproov element into page
  document.getElementById("your-container-id").appendChild(iProovMe)
})
```

#### React 16, Axios, ES6 async/await fetch file from local or external source

```jsx harmony
import React, { Component } from "react"
import axios from "axios" // assumes you have already installed axios as a dependency
import "@iproov/web" // includes the @iproov/web client into your app

export default class App extends Component {
  state = {
    language: "", // keep empty to stop render firing until we have a language object
    token: "***YOUR_TOKEN_HERE***",
  }

  componentDidMount() {
    this.setLanguage()
  }

  async setLanguage() {
    const path = "" // local or external path to language JSON file
    const response = await axios.get(path).catch(error => console.error(error))

    const language = await JSON.stringify(response.data)
    this.setState({ language })
  }

  render() {
    const { token, language } = this.state
    // only render in the iproov component when we have a language object
    if (!language) {
      return null
    }

    return (
      <div className="App">
        <iproov-me token={token} language={language} />
      </div>
    )
  }
}
```

#### Angular 7, HttpClient, ES6 fetch file from local filesystem

Add below to your `tsconfig.json` file to import json files locally in Angular. The example below is based from the [Angular integration examples](#Angular-v7).

```json
{
  "resolveJsonModule": true,
  "esModuleInterop": true
}
```

```typescript
import { Component, OnInit, Injectable } from "@angular/core"
import "@iproov/web"

import Language from "./lang.json" // enter your relative path to desired language file

@Component({
  selector: "app-iproov",
  templateUrl: "./iproov-component.html",
})
export class IproovComponent implements OnInit {
  constructor() {
    // this.getData() // external file example
    this.language = <any>Language // local file example
    this.token = "***YOUR_TOKEN_HERE***" // replace with your generated token
  }

  ngOnInit() {
    // inject iproov-me component into page with token
    window.addEventListener("WebComponentsReady", () => {
      let iProovMe = document.createElement("iproov-me")
      // set your generated token
      iProovMe.setAttribute("token", this.token)
      iProovMe.setAttribute("language", JSON.stringify(this.language))

      // inject iproov element into your page
      document.getElementById("iproov-container").insertAdjacentElement("beforebegin", iProovMe)
    })
  }
}
```

#### Angular 7, HttpClient, ES6 fetch file from external source

Add below to your `tsconfig.json` file to import json files locally in Angular. The example below is based from the [Angular integration examples](#Angular-v7).

```json
{
  "resolveJsonModule": true,
  "esModuleInterop": true
}
```

You then need to import `HttpClientModule` within your `app.module.ts`

```typescript
import { BrowserModule } from "@angular/platform-browser"
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from "@angular/core"
import { HttpClientModule } from "@angular/common/http" // import HttpClientModule from Angular

import { AppRoutingModule } from "./app-routing.module"
import { AppComponent } from "./app.component"
import { IproovComponent } from "./iproov.component"
@NgModule({
  declarations: [AppComponent, IproovComponent],
  imports: [
    BrowserModule,
    HttpClientModule, // add new imported module here
    AppRoutingModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class AppModule {}
```

Then within your component, you can access the HttpClient and inject into the `<iproov />` component as shown below.

```typescript
import { Component, OnInit, Injectable } from "@angular/core"
import { HttpClient } from "@angular/common/http" // import HttpClient from angular
import "@iproov/web"

@Component({
  selector: "app-iproov",
  templateUrl: "./iproov-component.html",
})
export class IproovComponent implements OnInit {
  // add HttpClient to component
  constructor(private http: HttpClient) {
    this.getData()
  }
  getData() {
    // get the json file and return as a string
    this.http.get("https://path-to-your-lang-file.json", { responseType: "text" }).subscribe(res => {
      this.language = <any>res
      this.injectElement()
    })
  }
  ready = false
  language = ""
  token = "***YOUR_TOKEN_HERE***" // replace with your generated token
  ngOnInit() {
    // inject iproov-me component into page with token
    window.addEventListener("WebComponentsReady", () => {
      this.ready = true
      this.injectElement()
    })
  }

  injectElement() {
    // only inject <iproov> component if lanugage has loaded and WebComponentsReady event has fired
    if (this.ready && this.language) {
      let iProovMe = document.createElement("iproov-me")
      // set your generated token
      iProovMe.setAttribute("token", this.token)
      iProovMe.setAttribute("language", this.language)

      // inject iproov element into your page
      document.getElementById("iproov-container").insertAdjacentElement("beforebegin", iProovMe)
    }
  }
}
```
