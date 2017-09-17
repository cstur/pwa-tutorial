## Pre-requesties
--------------------------

1. Install/Update chrome to 60 or above.
2. Install chrome web server [Chrome Web Server](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb?hl=en)
3. Install Chrome in mobile phone
4. Install firesbase tool via npm
```
npm install -g firebase-tools
```

## Steps for our demo (Weather App)

1. Go to workspace folder in your laptop/desktop and clone the repository below
```
C:\workspace> git clone https://github.com/mohanramphp/pwa-tutorial.git
C:\workspace> cd pwa-tutorial
C:\workspace\pwa-tutorial> cd demo
C:\workspace\pwa-tutorial\demo> cd work
C:\workspace\pwa-tutorial\demo\work>code .
```

2. Go to chrome apps in chrome and launch 200 ok!
```
chrome://apps/
```

3. Choose your work folder and stop and start the server and click on the url shown below web browser URL(s) 

4. To run the app with initial data
    1. In index.html, uncomment below line
    ```html
    <!--<script src="scripts/app.js" async></script>-->
    ```
    2. uncomment the following line at the bottom of your app.js file
    ```javascript
    // app.updateForecastCard(initialWeatherForecast);
    ```
    3. comment the line in #2 again its just to check app works with fake data.
5. Find the below TODO line in app.js and add code below.
```javascript
// TODO add saveSelectedCities function here
// Save list of cities to localStorage.
  app.saveSelectedCities = function() {
    var selectedCities = JSON.stringify(app.selectedCities);
    localStorage.selectedCities = selectedCities;
  };
  ```
6. Find the below TODO line in app.js and add code below.
```javascript
// TODO add startup code here
  app.selectedCities = localStorage.selectedCities;
  if (app.selectedCities) {
    app.selectedCities = JSON.parse(app.selectedCities);
    app.selectedCities.forEach(function(city) {
      app.getForecast(city.key, city.label);
    });
  } else {
    app.updateForecastCard(initialWeatherForecast);
    app.selectedCities = [
      {key: initialWeatherForecast.key, label: initialWeatherForecast.label}
    ];
    app.saveSelectedCities();
  }
  ```

  7. We have to save the selected city in local storage
  ```javascript
  document.getElementById('butAddCity').addEventListener('click', function() {
    // Add the newly selected city
    var select = document.getElementById('selectCityToAdd');
    var selected = select.options[select.selectedIndex];
    var key = selected.value;
    var label = selected.textContent;
    if (!app.selectedCities) {
      app.selectedCities = [];
    }
    app.getForecast(key, label);
    app.selectedCities.push({key: key, label: label});
    app.saveSelectedCities();
    app.toggleAddDialog(false);
  });
  ```
> Test it out
>    1. When first run, your app should immediately show the user the forecast from initialWeatherForecast.
>    2. Add a new city (by clicking the + icon on the upper right) and verify that two cards are shown.
>    3. Refresh the browser and verify that the app loads both forecasts and shows the latest information.
9. Use service workers to pre-cache the App Shell
    1. Find the below TODO line in app.js and add code below.
```javascript
// TODO add service worker code here
if ('serviceWorker' in navigator) {
    navigator.serviceWorker
             .register('./service-worker.js')
             .then(function() { console.log('Service Worker Registered'); });
}
```
10. Add this code to your new service-worker.js file

```javascript
var cacheName = 'weatherPWA-step-6-1';
var filesToCache = [];

self.addEventListener('install', function(e) {
  console.log('[ServiceWorker] Install');
  e.waitUntil(
    caches.open(cacheName).then(function(cache) {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll(filesToCache);
    })
  );
});
```
11. Now, reload your page. Open the service worker pane in chrome developer
12. Add the following script, below the install event in service-worker.js
```javascript
self.addEventListener('activate', function(e) {
  console.log('[ServiceWorker] Activate');
});
```
>- The activate event is fired when the service worker starts up
> - Basically, the old service worker continues to control the page as long as there is a tab open to the page
13. Enable update on reload option in Chrome dev tool
14. Add the following code to update the cache
```javascript
self.addEventListener('activate', function(e) {
  console.log('[ServiceWorker] Activate');
  e.waitUntil(
    caches.keys().then(function(keyList) {
      return Promise.all(keyList.map(function(key) {
        if (key !== cacheName) {
          console.log('[ServiceWorker] Removing old cache', key);
          return caches.delete(key);
        }
      }));
    })
  );
  return self.clients.claim();
});
```
> Case in which the app wasn't returning the latest data so to avoid that ``` self.clients.claim() ``` is used
15. Replace filesToCache variable declaration 
```javascript 
var filesToCache = [
  '/',
  '/index.html',
  '/scripts/app.js',
  '/styles/inline.css',
  '/images/clear.png',
  '/images/cloudy-scattered-showers.png',
  '/images/cloudy.png',
  '/images/fog.png',
  '/images/ic_add_white_24px.svg',
  '/images/ic_refresh_white_24px.svg',
  '/images/partly-cloudy.png',
  '/images/rain.png',
  '/images/scattered-showers.png',
  '/images/sleet.png',
  '/images/snow.png',
  '/images/thunderstorm.png',
  '/images/wind.png'
];
```
16. Let's now serve the app shell from the cache. Add the following code to the bottom of your service-worker.js file
```javascript 
self.addEventListener('fetch', function(e) {
  console.log('[ServiceWorker] Fetch', e.request.url);
  e.respondWith(
    caches.match(e.request).then(function(response) {
      return response || fetch(e.request);
    })
  );
});
```

>Your app is now offline-capable! Let's try it out.
> Test it out.
>   1. Reload your page and then go to the Cache Storage pane on the Application panel of DevTools. 
>   2. Right click Cache Storage, pick Refresh Caches, expand the section and you should see the name of your app shell cache listed on the left-hand side. 
>   3. When you click on your app shell cache you can see all of the resources that it has currently cached.
>   4. In Service Worker pane of DevTools and enable the Offline checkbox. After enabling it, you should see a little yellow warning icon next to the Network panel tab. This indicates that you're offline.
>   5. Reload your page to see the initial rendering with fake data

17. Use service workers to cache the forecast data
18. Add the following line to the top of your service-worker.js file
```javascript
var dataCacheName = 'weatherData-v1';
```
19. Update the code in activate event handler
```javascript
if (key !== cacheName && key !== dataCacheName) {
```
20. Update the fetch event
```javascript
self.addEventListener('fetch', function(e) {
  console.log('[Service Worker] Fetch', e.request.url);
  var dataUrl = 'https://query.yahooapis.com/v1/public/yql';
  if (e.request.url.indexOf(dataUrl) > -1) {
    e.respondWith(
      caches.open(dataCacheName).then(function(cache) {
        return fetch(e.request).then(function(response){
          cache.put(e.request.url, response.clone());
          return response;
        });
      })
    );
  } else {
    e.respondWith(
      caches.match(e.request).then(function(response) {
        return response || fetch(e.request);
      })
    );
  }
});
```
21. Get data from the cache - Find the TODO add cache logic here comment in app.getForecast() in app.js
```javascript
//TODO add cache logic here
if ('caches' in window) {
  /*
    * Check if the service worker has already cached this city's weather
    * data. If the service worker has the data, then display the cached
    * data while the app fetches the latest data.
    */
  caches.match(url).then(function(response) {
    if (response) {
      response.json().then(function updateFromCache(json) {
        var results = json.query.results;
        results.key = key;
        results.label = label;
        results.created = json.query.created;
        app.updateForecastCard(results);
      });
    }
  });
}
```
22. Notice how the cache request and the XHR request both end with a call to update the forecast card. How does the app know whether it's displaying the latest data? This is handled in the following code from app.updateForecastCard
```javascript
var cardLastUpdatedElem = card.querySelector('.card-last-updated');
var cardLastUpdated = cardLastUpdatedElem.textContent;
if (cardLastUpdated) {
  cardLastUpdated = new Date(cardLastUpdated);
  // Bail if the card has more recent data then the data
  if (dataLastUpdated.getTime() < cardLastUpdated.getTime()) {
    return;
  }
}
```
> The app should be completely offline-functional now.
> Test it now.
>   1. Save a couple of cities and press the refresh button on the app to get fresh weather data
>   2. then go offline and reload the page.
>   3. go to the Cache Storage pane on the Application panel of DevTools. Expand the section and you should see the name of your app shell and data cache listed on the left-hand side. Opening the data cache should should the data stored for each city.
23. Web App Install Banners and Add to Homescreen for Chrome on Android
24. Create a file named manifest.json in your work folder and copy/paste the following contents
```json
{
  "name": "Weather",
  "short_name": "Weather",
  "icons": [{
    "src": "images/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-256x256.png",
      "sizes": "256x256",
      "type": "image/png"
    }],
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#3E4EB8",
  "theme_color": "#2F3BA2"
}
```
25. Tell the browser about your manifest file. Now add the following line to the bottom of the ``` <head> ``` element in your index.html file
```html
<link rel="manifest" href="/manifest.json">
```
26. Add to Homescreen elements for Safari on iOS. In your index.html, add the following to the bottom of the ``` <head> ``` element

```html
<!-- Add to home screen for Safari on iOS -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-title" content="Weather PWA">
<link rel="apple-touch-icon" href="images/icons/icon-152x152.png">
```
27. Tile Icon for Windows. In your index.html, add the following to the bottom of the ``` <head> ``` element
```html
<meta name="msapplication-TileImage" content="images/icons/icon-144x144.png">
<meta name="msapplication-TileColor" content="#2F3BA2">
```
> Open up the Manifest pane on the Application panel. you'll be able to see it parsed and displayed in a human-friendly format on this pane.

28. Deploying our application in firebase
    1. Create a new app at https://firebase.google.com/console/
    2. open cmd prompt from work folder and run 
    ```cmd 
    firebase login 
    ```
    3. Initialize your app and provide the directory (likely "./") 
    ```cmd 
    firebase init 
    ```
    4. Finally, deploy the app to Firebase
    ```cmd
    firebase deploy
    ```
    5. done! Your app will be deployed to the domain: https://YOUR-FIREBASE-APP.firebaseapp.com
    


















    


