---
layout: post
title:  "Embedding An EmberJS App in Drupal 7"
date:   2015-09-06 00:00:00 +1100
tags: drupal, emberjs, javascript
permalink: /building-emberjs-application-headless-drupal-7
---

Before you can build an Ember app to embedded in Drupal 7, you’ll need to get Drupal running “headlessly” i.e. outputting data via an API instead of rendering HTML through a theme. I’ve written a tutorial to demonstrate how to set this up: [Exposing Drupal 7 data in a custom RESTful API with the RESTful module](http://anthonygore.com/exposing-drupal-data-custom-restful-api-restful-module/).

There are a number of ways you can run Ember and other JS frameworks alongside headless Drupal, for example as standalone apps connected to Drupal only by the API. In this tutorial we’ll be embedding an Ember app in a Drupal page. This has a number of advantages e.g. allowing your app to utilise Drupal’s Permission API and menu system.

**1. Create a custom Drupal module**

We’ll be making an application in this tutorial called Animal World. to display different animals. Our custom module will be called animal_world

**2. Create a new Ember project within your custom module**

For this tutorial you’ll need to install the [Ember-CLI](http://www.ember-cli.com/). Once installed, navigate to the animal_world folder and create a new Ember project:

    animal_world$ ember new ember-app

You can check your Ember app has installed:

    animal_world$ cd ember-app
    animal_world/ember-app$ ember serve

Navigate to _http://<i></i>localhost:4200_ and you’ll see it running on a Node server.

**3. Get Drupal to load your Ember app**

We need to get Drupal to load the Ember app when the user navigates to a certain path. (If you want your app to run standalone, you can skip this step). Let’s launch the app when users visits _http://<i></i>animals.dev/animal-world_. Add this code to your custom module:

_animal_world/animal_world.module_

    define('ANIMAL_WORLD_URL', 'animal-world');
    define('ANIMAL_WORLD_APP', 'ember-app');
     
    /**
     * Implements hook_menu().
     */
    function animal_world_menu() {
     
      $items[ANIMAL_WORLD_URL] = array(
        'title'             => 'Animal World',
        'access callback'   => TRUE,
        'page callback'     => 'animal_world_page',
        'delivery callback' => 'animal_world_delivery',
        'type'              => MENU_CALLBACK
      );
     
      return $items;
    }
     
    /**
     * The page callback function. Loads the Ember app
     */
    function animal_world_page() {
     
     $app = ANIMAL_WORLD_APP
     $path = drupal_get_path('module', 'animal_world') . '/' . $app;
     
     drupal_add_js("{$path}/dist/assets/vendor.js");
     drupal_add_js("{$path}/dist/assets/{$app}.js");
     drupal_add_css("{$path}/assets/vendor.css");
     drupal_add_css("{$path}/assets/{$app}.css");
     
     return theme('animal_world_ember');
    }
     
    /**
     * Delivery callback used to override html.tpl.php from the module
     *
     * @param $page_callback_result
     */
    function animal_world_delivery($page_callback_result) {
     
      global $language;
     
      // Pass variables to the template.
      $vars = array(
        'language'    => $language,
        'head_title'  => drupal_get_title(),
        'favicon'     => '',
        'styles'      => drupal_get_css(),
        'scripts'     => drupal_get_js(),
        'messages'    => drupal_get_messages(),
        'content'     => $page_callback_result,
        'ember_base'  => url(ANIMAL_WORLD_URL, array(
          'absolute' => TRUE,
        )) . '/',
      );
     
      echo theme('animal_world_html', $vars);
     
    }

We need to add a bit more code, but let’s first look at what the above does:

* The hook_menu implementation tells Drupal what to do when a user visits path /animal-world.
* The page callback function loads the files needed for your Ember app. Keep in mind that as you create your app, Ember will build it into a single .js and .css file. The page callback loads those, as well as the vendor .js and .css files.
* The delivery callback simply wraps your ember app in a Drupal theme file. By default it would just be wrapped inside your theme html.tpl.php file. If you’re okay with that, you don’t need a delivery callback. I want a custom template though so I don’t get all the navigation, footer etc from my theme. We pass the theme the obvious parameters, but I’m also passing ’ember_base’ which will be explained below.

Both callbacks are calling the theme function, so let’s now add a hook implementation for that:

_animal_world/animal_world.module_

    /**
     * Implements hook_theme()
     *
     * @return mixed
     */
    function animal_world_theme() {
     
      $theme['animal_world_embed'] = array(
        'template'    => 'animal-world-embed',
        'path'        => drupal_get_path('module', 'animal_world') . '/templates'
      );
     
      $theme['animal_world_html'] = array(
        'template'    => 'animal-world-html',
        'path'        => drupal_get_path('module', 'animal_world') . '/templates',
        'variables'   => array(
          'language'    => NULL,
          'head_title'  => NULL,
          'favicon'     => NULL,
          'styles'      => NULL,
          'scripts'     => NULL,
          'messages'    => NULL,
          'content'     => NULL,
          'ember_base'  => NULL
        ),
      );
     
      return $theme;
    }

Then create a templates directory in your module and add these files:

_animal_world/templates/animal-world-embed.tpl.php_

    <div id="ember-app"></div>

_animal_world/templates/animal-world-html.tpl.php_

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.0//EN"
      "http://www.w3.org/MarkUp/DTD/xhtml-rdfa-1.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="<?php print $language->language; ?>" version="XHTML+RDFa 1.0">
     
    <head>
      <title><?php print $head_title; ?></title>
      <base href="<?php print $ember_base; ?>" />
      <?php print $styles; ?>
      <?php print $scripts; ?>
    </head>
    <body>
    <?php
    if (!empty($page['message'])):
      foreach($page['message'] as $type => $message):
        ?>
        <div class="messages <?php print $type; ?>">
          <ul>
            <?php foreach($message as $msg): ?>
              <li><?php print $msg; ?></li>
            <?php endforeach; ?>
          </ul>
        </div>
        <?php
      endforeach;
    endif; ?>
     
    <?php print $content; ?>
     
    </body>
    </html>

What you need to know about the above code is:

* The animal_world_embed template is used to embed the Ember app into the page. We’ll set this up from within the Ember app as well.
* The animal_world_html template is simply a replacement of the default html template for the theme. It’s important that you add a <base> tag in the head and set the base URL of the Ember app, otherwise Ember may have trouble dealing with routes.

**4. Configure Ember**

Let’s switch over to EmberJS now. By default, Ember will attach the application template to the <body> tag of an HTML page. We’re instead going to specify a different root element property:

_animal_world/ember_app/app.js_

    App = Ember.Application.create({
      rootElement: '#ember-app'
    });

Now when Ember fires up, it will know to embed output in the template file _animal-world-embed.tpl.php_

Go to _http://<i></i>animals.dev/animal-world_ and you’ll see the Ember app as loaded by Drupal!

**5. Load and display Drupal content in Ember**

Getting Drupal to load the Ember app is the first bit, now we want Ember to grab data from Drupal’s API and display it.

Let’s set up a very simple app that will do nothing more than display some Drupal content type to the screen.

Firstly create a route that we’ll call main:

    animal_world/ember-app$ ember generate route main

Now set it’s path. We want the main route to be used immediately when the app is launched.

_animal_world/ember-app/app/router.js_

    Router.map(function() {
      this.route('main', {path: '/'});
    });

In the tutorial I wrote on setting up a Drupal web service, I demonstrated with a content type called dogs. Let’s use that content type again. The API method and it’s output are below:

_http://<i></i>animals.dev/api/v1.0/dogs/1_

    {
      "dog": [
        {
          "id": 1,
          "label": "Rusty",
          "breed": "Kelpie",
          "age": 6
        }
      ]
    }

We’ll now load this content type into an Ember model:

    animal_world/ember-app$ ember generate model dogs

animal_world/ember-app/app/models/dogs.js

    export default DS.Model.extend({
        label:          DS.attr(),
        breed:          DS.attr(),
        age:            DS.attr(),
    });

As long as the JSON object output by the API is the same as what I’ve specified above, Ember will automatically load the JSON data into the model we created. But there is one additional configuration we have to make: we need to set the API endpoint namespace in a custom adapter.

    animal_world/ember-app$ ember generate adapter application

_animal_world/ember-app/app/adapters/application.js_

    export default DS.RESTAdapter.extend({
        namespace:      'api/v1.0',
    });

With all the above done, Ember will be correctly loading the models from Drupal’s API. The last thing to do is add to our main template so that Ember will display the dogs content to screen

_animal_world/ember-app/app/templates/main.hbs_

    <h2>Dogs</h2>
    {{#each model as |dog|}}
        <li>
            <h3>{{dog.label}}</h3>
            <ul>
                <li>{{{dog.age}}}</li>
                <li>{{dog.breed}}</li>
            </ul>
        </li>
    {{/each}}

Now when we visit our animal world app we’ll see a simple representation of our dogs content type.
