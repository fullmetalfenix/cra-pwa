# cra-pwa

## Using and testing [Create React Apps](https://reactjs.org/docs/create-a-new-react-app.html) default service worker setup to build a progressive web app.

### Step One: set up the project:
`npx create-react-app my-app --template cra-template-pwa`

Using this template (cra-template-pwa) instead of the default adds two important files: `src/service-worker.js` and `src/serviceWorkerRegistration.js`

What do they do? The presence of the `src/service-worker.js` file signals to the `InjectManifest` (from googles pwa library [Work box](https://developers.google.com/web/tools/workbox/)) plugin included in the template above to compile the service worker and inserts into it a dynamically created list of URLs to precache. Is the list perfect? No. But its a great start to a project!

### Step Two: Register the service worker
 Now for that other file, `src/serviceWorkerRegistration.js` - in index.js change the `servicewomen.unregister()` line to `servicewomen.register()` (Just like the comments say). Now, in the console, npm start and check out the Application tab in the developer tools (Ctrl + Shift + i) > Service worker aaaannd nothings there. Why? The service worker only compiles in the production build of Create React App so back at the console, npm build, then follow the prompts to `npm install -g serve` then `serve -s build` now check out the application and you can see the service worker is up and running! 
 
 ### Step Three: Checking out the benefits

Now to check out what it does. In the same service worker tab in the application pane of developer tools check "offline" near the top. Now reload - the page is still there! That's part of the magic of a PWA, users still get content even with a spotty connection (Lie-Fi) or even no connection at all due to the content being precached by the service worker.

Next lets take a look at the speed compared to using the development build. Create react app now ships with a library called [Web Vitals](https://github.com/GoogleChrome/web-vitals) that does a lot of the things that Google Lighthouse does. For our simple test we will add `console.log` in the reportWebVitals function call in index.js, reload the app and you can see that it is logging some performance info to the console - info about the cumulative layout shift, first contentful paint and other user/performance metrics but the one I wanted look at is the time to first byte. Fire up both the development server (npm start) and the build server (serve -s build) then take a look at the networking tab, select the first thing to load in the timeline - then in the timing tab look at the information - the connection info is missing replaced with a service worker section. Why? Because the resource is being served by the service worker from a local browser cache not the server! No DNS lookup, or connection info required. What does that mean? TTFB should be waaaay faster. To be fair, the production build is also optimized (concat. + minified) so it would be faster anyway but if you want to compare what the difference is between having a service worker serve up your content vs not having one you can just delete your service worker and disable the cache, reload the page and look at those statistics then re-enable the cache and give it a few reloads for good measure and look at those stats. In my tests TTFB is still less than half using the service worker / browser cache than without it and thats with the server actually running on my computer - imagine the difference out in the wild! The First Contentful Paint is about 1/4. Awesome! 


### Step 4: lets add another image

OK, an experiment: add another image below the react logo. Any image - I've used Grogu. Rebuild the app then open the page in an incogneto window (PWA's can be difficult to debug beceause of the caching). See the image? Good! Now open the developer pane, then reload the application with "offline" checked. What happens? Broken image. Why? As it says in the template documentation "All static site assets _that are a part of your webpack build_ are cached" and that it will "automatically precache all of your _webpack-generated assets_" - obviously, this image is not part of the process. How do we add it?

At the bottom of your service worker file add:
` precacheAndRoute([
    {url: '<url-to-your-image>', revision: 'null' },
  ]);`

Rebuild, reload and check out the cache now in the Applicaiton > Cache storage tab. Your image should be listed! Try it offline, and it should still be seen. Why? Check out the [documentation for precacheAndRoute()](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-precaching#.precacheAndRoute) "This method will add entries to the precache list and add a route to respond to fetch events." Whats that mean? it adds the image to the precache list so its cached and adds a fetch listener that matches request to that route (the one to your image) and serves it from the cache instead. Neat.  