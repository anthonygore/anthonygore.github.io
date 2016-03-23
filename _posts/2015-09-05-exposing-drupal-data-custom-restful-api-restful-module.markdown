---
layout: post
title:  "Exposing Drupal 7 Data In A Custom RESTful API With The Restful Module"
date:   2015-09-05 00:00:00 +1100
tags: drupal, emberjs, javascript
permalink: /exposing-drupal-data-custom-restful-api-restful-module/
---

You can use the [RESTful module](https://github.com/RESTful-Drupal/restful) to create a custom web service that exposes data from your Drupal 7 instance. For example, if you have a content type called “dogs” with fields “label”, “age”, “breed”, you could have instances of the dogs content type available as a JSON object at an endpoint like:

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

The RESTful module can expose data from many data types in Drupal including taxonomies, nodes etc.

Let’s assume we have the dogs content type setup in Drupal. How do we set up a web service?

**1. Install the Restful module**

[https://github.com/RESTful-Drupal/restful](https://github.com/RESTful-Drupal/restful)

**2. Create a custom module**

e.g. animals_api, with these files:

_animals_api/animals_api.info_

    name = Animals API
    description = RESTful API for animals based on restful module
    core = 7.x
    dependencies[] = restful

_animals_api/animals_api.module_

    <?php
    /**
    * Implements hook_ctools_plugin_directory(). This hook is used to inform the
    * CTools plugin system about the location of a directory that should be
    * searched for files containing plugins of a particular type.
    */
    function restful_custom_ctools_plugin_directory($module, $plugin) {
      if ($module == 'restful') {
        return 'plugins/' . $plugin;
      }
    }

**3. Declare a rest method**

Create a plugins directory and a restful directory inside that:

_animals_api/plugins_
_animals_api/plugins/restful_

To declare the dogs method, we create a file called dogs.inc

_animals_api/plugins/restful/dogs.inc_

    <?php
    $plugin = array(
     'label' => t('Dogs'),
     'resource' => 'dogs',
     'name' => 'dogs',
     'entity_type' => 'node',
     'bundle' => 'dogs',
     'description' => t('Help Me Choose answers (aka features)'),
     'class' => 'RestfulEntityBaseNode',
    );

Note:

* The resource ‘dogs’ determines the root URL of the method
* The name ‘dogs’ must match the name of the file
* The bundle ‘dogs’ is the machine name of the content type in Drupal (if it were a taxonomy you wanted to expose etc, you’d use the machine name of the vocabulary etc)
* ‘class’ (and ‘formatter’, not yet shown) will be explained below…

With that, we should now be able to access:

_http://<i></i>animals.com/api/v1.0/dogs/_

And we’d if we’ve added one instance of dogs we’ll see:

    {
       "count": 1,
       "self": {
         "title": "Self",
         "href": "http://animals.dev/api/v1.0/dogs"
       },
       "data": [
         {
           "id": 1,
           "label": "Rusty"
         }
       ]
     }

And to view entity with id 1:

_http://<i></i>animals.com/api/v1.0/dogs/1_

    {
      "self": {
        "title": "Self",
        "href": "http://animals.dev/api/v1.0/dogs/1"
      },
      "data": [
        {
          "id": 1,
          "label": "Rusty"
        }
      ]
    }

**4. Customise the fields**

By default, you’ll only see the label field of each entity in the output. To do add additional fields, you need to extend one of the bases classes supplied by RESTful.

In our dogs.inc file we used the RestfulEntityBaseNode class to define output. We’ll change that to our own custom class RestfulDogs:

_animals_api/plugins/restful/RestfulDogs.class.php_

    class RestfulDogs extends RestfulEntityBaseNode {
     
      /**
       * Overrides RestfulEntityBaseNode::publicFieldsInfo().
       */
      public function publicFieldsInfo() {
        $public_fields = parent::publicFieldsInfo();
     
        $public_fields['age'] = array(
         'property' => 'field_age',
         'sub_property' => 'value',
        );
        $public_fields['breed'] = array(
         'property'          => 'field_breed',
         'process_callbacks' => array(
           array($this, 'process_breed')
         )
        );
        return $public_fields;
      }
     
       /**
        * Callback to return the breed taxonomy name
        */
       public function process_breed($tax) {
         if (!empty($tax->tid)) {
           $tax_obj = taxonomy_term_load($tax->tid)
           if ($tax_obj) {
             return $tax_obj->name;
           }
           else {
        return false;
           }
        }
        return null;
      }
    }
    
So what’s going on here? We need to override `publicFieldsInfo()` to be able to specify what fields we want to expose in the web service. This function returns an array of fields which we can simply add to.

Notes:

* The ‘property’ property is where you put the machine name of the field in your entity that you want to expose
* The ‘sub_property’ property is the child element that contains the actual value of the field you want to expose
* The ‘process_callbacks’ property is where you can list one or more functions to transform your field before it’s rendered. In the example above, I’ve processed the field ‘breed’. Let’s assume that field is a term reference and will only render a tid in the API output. We want the taxonomy name to display instead. The callback will actually wrap the tid in an entity wrapper, so the callback function ‘process_breed’ receives a taxonomy object as an argument. Pretty cool.

Now the output will include the additional fields:

_http://<i></i>animals.dev/api/v1.0/dogs/1_

    {
     "self": {
        "title": "Self",
        "href": "http://animals.dev/api/v1.0/dogs/1"
      },
      "data": [
        {
          "id": 1,
          "label": "Rusty",
          "breed": "Kelpie",
          "age": 6
        }
      ]
    }

**5. Format the output**

Finally, let’s transform the output a bit. I want to use this API as store for an EmberJS application, so I need my JSON object to look a little different.

We can extend the formatter class to achieve this. Firstly create another directory:

_animals_api/plugins/formatter/_

Declare a new formatter:

_animals_api/plugins/formatter/custom_json.inc_

    <?php 
     
    $plugin = array(
     'label' => t('Custom JSON Formatter'),
     'description' => t('Defining a custom formatter to allow ntegration with Ember'),
     'name' => 'custom_json',
     'class' => 'RestfulCustomFormatterJson',
    );
    
Extend one of the formatter classes:

_animals_api/plugins/formatter/RestfulCustomFormatterJson.class.php_

    class RestfulCustomFormatterJson extends \RestfulFormatterJson implements \RestfulFormatterInterface {
     
     /**
      * {@inheritdoc}
      */
     public function prepare(array $data) {
     
       $output = parent::prepare($data);
     
       $key = 'data';
     
       switch ($this->handler->getBundle()) {
         case 'dogs':
           $key = dog';
           break;
       }
     
       $output[$key] = $output['data'];
       unset($output['data']);
     
       if (isset($output['count'])) {
         unset($output['count']);
       }
       if (isset($output['self'])) {
         unset($output['self']);
       }
     
       return $output;
     }
    }


What’s this code do?

* Firstly, I want the root of the JSON object to be ‘dog’ not ‘data’. To do this, I’ve created a switch based on the bundle name. We only have dogs in our api, but if we have cats, ferrets etc, we want a root that reflects those content types too.
* Secondly, I don’t want the ‘count’ and ‘self’ data in my JSON object, so I’ve simply unset those.

The final step is to apply this formatter to the dogs method. In our dogs.inc file we can include another property in the plugin array:

    'formatter' => 'custom_json'
   
Now we get this as our output. Notice the JSON root has changed and the default ‘count’ and ‘self’ properties have disappeared:

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

I’m planning a follow-up post for integrating EmberJS with Drupal via the RESTful module.
