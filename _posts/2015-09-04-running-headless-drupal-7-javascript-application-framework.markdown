---
layout: post
title:  "Running Headless Drupal 7 With A Javascript Application Framework"
date:   2015-09-04 00:00:00 +1100
tags: drupal, javascript
permalink: /running-headless-drupal-7-javascript-application-framework/
---

When Drupal is running headlessly it’s theme is missing and instead the user is interacting with another application that is likely sending and receiving data to Drupal via an API.

The advantages of doing this can be presented from two perspectives:

1.  **I want to use Drupal for my web app. Why go “headless” and use a front-end framework?** Standard Drupal 7 is fine for simple, form-based apps. You get jQuery and jQuery UI out of the box, so form validation, conditional fields etc aren’t too much hassle. But Drupal (or any CMS for that matter) is really not up to the task of building a highly interactive web app.
2.  **I want use a front-end application framework. Why build it on top of Drupal?** CMS’s are great for managing complex data types including storage, retrieval, permission-based access etc. A CMS will also have an UI which will be handy for administrating your app. Drupal just happens to be particularly good at all these things. It’s APIs, hook system and thousands of modules make it the superior choice.

To build a web app with a JS framework on top of Drupal 7, you need to do the following:

1.  Get Drupal to function headlessly. This can be done by providing an API for your Drupal instance that allows another application to access the content. I’ve written a tutorial on how to do that with the RESTful module: [Exposing Drupal 7 data in a custom RESTful API with the RESTful module](/exposing-drupal-data-custom-restful-api-restful-module/)
2.  Get your front-end app to talk to Drupal. If Drupal is exposing data via an API this isn’t too hard to achieve. I’ve also written a tutorial on getting an EmberJS app to work from inside Drupal: [Embedding An EmberJS App In Drupal 7](http://anthonygore.com/building-an-emberjs-application-on-headless-drupal-7)
