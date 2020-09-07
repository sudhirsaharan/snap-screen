# snap-screen

Building a screenshot taking extension
Here i talk about the basics of extensions and we will learn about creating a screenshot taking extension which takes a screenshot of the currently visible page of your browser.
Let’s start making our extension now.

Create the Manifest
The core of a chrome extension is the manifest file. We need to create a manifest.json file in our folder. Inside your manifest file add this
{
    "name": "Screenshot extension",
    "version": "1.0",
    "description": "Building a screenshot taking extension",
    "manifest_version": 2
}
After adding these keys, we can add the extension in our chrome extensions tab. But first, let’s understand the meaning of each line.

 * name : Name is the name of our extension which is visible to the users, just like Grammarly, honey etc.
 * version : Version describes the version number of your extension, we will start that from 1.0
 * description : Description is the description for your extension.
 * manifest_version : Developers should specify which version of the manifest specification their package targets by setting a manifest_version key in their manifests. Manifest version 1 was deprecated in Chrome 18. So, we will be using manifest version 2.
  
Now, the directory holding the manifest file can be added as an extension in developer mode in its current state.
Open the Extension Management page by navigating to chrome://extensions.
The Extension Management page can also be opened by clicking on the Chrome menu, hovering over More Tools then selecting Extensions.
Enable Developer Mode by clicking the toggle switch next to Developer mode.
Click the LOAD UNPACKED button and select the extension directory.




Now let’s add some functionality to our extension. We’ll start by adding a background script in our folder and some other essentials like icons and permissions to our manifest file.
```
{
    "name": "Screenshot extension",
    "version": "1.0",
    "description": "Building a screenshot taking extension",
    "background": {
        "scripts": ["background.js"],
        "persistent": false
    },
    "browser_action": {
        "default_title": "Take a screen shot!" 
    },
    "permissions": [
        "activeTab"
    ],
    "manifest_version": 2,
}
```
* background : Registering a background script in the manifest tells the extension which file to reference, and how that file should behave. The extension is now aware that it includes a non-persistent background script and will scan the registered file for important events it needs to listen for.
We will set persistent to false. The only occasion to keep a background script persistently active is if the extension uses chrome.webRequest API to block or modify network requests. The webRequest API is incompatible with non-persistent background pages.
* browser_action : We use browser actions to put icons in the main Google Chrome toolbar, to the right of the address bar. In addition to its icon, a browser action can have a tooltip, a badge, and a popup. Popup contains the HTML pages. For now, we will be using icon and title.
* icons : It specifies the icons of our extension with their dimensions.
* permissions : If an API requires you to declare permission in the manifest, then its documentation tells you how to do so. Each permission can be either one of a list of known strings (such as “geolocation”) or a match pattern that gives access to one or more hosts. We are using activeTab for our extension, if we read about activeTab’s description, it would mean that “the activeTab permission gives an extension temporary access to the currently active tab when the user invokes the extension - for example by clicking its browser action. Access to the tab lasts while the user is on that page, and is revoked when the user navigates away or closes the tab.”

This completes our manifest file. Now, let’s move on to background.js file.

```
let id = 100;
chrome.browserAction.onClicked.addListener(() => {
chrome.tabs.captureVisibleTab((screenshotUrl) => {
    const viewTabUrl = chrome.extension.getURL('screenshot.html?id=' + id++)
    let targetId = null;
chrome.tabs.onUpdated.addListener(function listener(tabId,     changedProps) {
if (tabId != targetId || changedProps.status != "complete")
        return;
chrome.tabs.onUpdated.removeListener(listener);
const views = chrome.extension.getViews();
      for (let i = 0; i < views.length; i++) {
        let view = views[i];
        if (view.location.href == viewTabUrl) {
          view.setScreenshotUrl(screenshotUrl);
          break;
        }
      }
    });
chrome.tabs.create({url: viewTabUrl}, (tab) => {
      targetId = tab.id;
    });
  });
});
```
Let’s understand this file step by step. First, we add a listener on our icon placed on the right of the address bar and capture the visible tab using the chrome tabs API and call captureVisibleTab method. This method returns us the data URL which we will use to show the image in the next tab.

Then we will create a tab URL to open the URL in the next tab and we append an id to its end so that every screenshot has a different page and should not conflict with the previous ones. For this, we initialized a id variable on the top from 100 which will keep on incrementing with each click.
We open the tab URL by using the chrome tabs create method and passing it the URL that we just created and we save the tab id that we get from this method after the tab is created in the targetId variable.

We also attach a listener on the tab created which is fired when it is loaded. So we attach a listener on onUpdated event of the tabs API. We did not attach a listener on onCreated event because the tab’s URL may not be set at the time this event is fired, but you can listen to onUpdated events so as to be notified when a URL is set. Inside the listener, we check if the opened tab’s id is the same as the target id that we just stored and the status of the page loading is complete . The changedProps object will return either loading or complete .

As soon as we pass the checks, we will remove the listener as we don’t need it now, so we remove it via removeEventListener.
We fetch all the views opened by our extension using getViews method and it returns an array of the JavaScript ‘window’ objects for each of the pages running inside the current extension. Inside the loop, we match each and every entry’s URL to the unique URL we created at the top and if we get a match, we call a function on that view which will be called on the page that has been opened by our extension and we pass our image URL to the page so that it can display it to the user.
Now, let’s see what we will add in our screenshot.html file that we just opened above.
```
<html>
  <script src="screenshot.js"></script>
  <body>
    <img id="target" src="white.png" height="480">
  </body>
</html>
```

We just add an image tag which will be used for displaying the image url sent by our extension, for the time being we are using a white background placeholder until replaced.
And inside our screenshot.js file that we just imported above.
```
function setScreenshotUrl(url) {
  document.getElementById('target').src = url;
}
```

This is the same setScreenshotUrl that we called in background.js file inside the loop on our view and it takes a URL as its parameter and sets that URL as image source URL and it displays our image.
