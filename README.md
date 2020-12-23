# cra-pwa

# Using and testing [Create React Apps](https://reactjs.org/docs/create-a-new-react-app.html) default service worker setup to build a progressive web app.

## Step One: set up the project:
`npx create-react-app my-app --template cra-template-pwa`

Using this template (cra-template-pwa) instead of the default adds two important files: `src/service-worker.js` and `src/serviceWorkerRegistration.js`

What do they do? The presence of the `src/service-worker.js` file signals to the `InjectManifest` (from googles pwa library [Workbox](https://developers.google.com/web/tools/workbox/) ) plugin included in the template above to compile the service worker and inserts into it a dynamically created list of URLs to precache. Is the list perfect? No. But its a great start to a project!

## Step Two: Register the service worker
 Now for that other file, `src/serviceWorkerRegistration.js` - in index.js change the `serviceWorker.unregister()` line to `serviceWorker.register()` (Just like the commments say). Now, in the console, npm start and check out the Applicaiton tab in the developer tools (Ctrl + Shift + i) > Service worker aaaannd nothings their. Why? The service worker only compiles in the production build of Create React App so back at the console, npm build, then follow the prompts to  `npm install -g serve` then `serve -s build` now check out the applicaiton and you can see the service worker is up and running! 
 
 ### Step Three: Checking out the benifits

Now to check out what it does. In the same service worker tab in the application pane of developer tools check "offline" near the top. Now reload - the page is still there! Thats part of the maagic of a PWA. You'll notice that not everything is cached though, the favicon isn't appearing, but we will fix that later - for now this is great! Users are still getting content even with a spotty connection (Lie-Fi) or even no connection at all due to the content being precached by the service worker.

Next lets take a look at the speed compaired to using the development build. Create react app now ships with a library called [Web Vitals](https://github.com/GoogleChrome/web-vitals) that does alot of the things that Google Lighthouse does
