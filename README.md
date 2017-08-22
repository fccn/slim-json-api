# slim-jsonAPI

[SLIM framework](https://github.com/codeguy/Slim) extension to implement json API's.

# About

This project is a fork of  [slim-jsonAPI](https://github.com/entomb/slim-json-api/).
The major differences are as follow:
- Ability to supress rewrite of *error* field on response

## Installation
Using composer you need to first add this repository to the **composer.json** file *repositories* parameter:

```json
    {
      "repositories": [
          {
              "type": "vcs",
              "url":  "https://github.com/fccn/slim-json-api.git"
          }
      ]
    }
```

Then you can include this component using the following command:

```sh
   composer require fccn/slim-json-api:dev-master
```

Or you can edit the **composer.json** file directly and add the following:

```json
    {
        "require": {
            "slim/slim": "2.3.*",
            "fccn/slim-json-api": "dev-master"
        }
    }

```

## Usage
To include the middleware and view you just have to load them using the default _Slim_ way.
Read more about Slim Here (https://github.com/codeguy/Slim#getting-started)

```php
    require 'vendor/autoload.php';

    $app = new \Slim\Slim();

    $app->view(new \JsonApiView());
    $app->add(new \JsonApiMiddleware());
```

### .htaccess sample
Here's an .htaccess sample for simple RESTful API's
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

### example method
all your requests will be returning a JSON output.
the usage will be `$app->render( (int)$HTTP_CODE, (array)$DATA);`

#### example Code
```php

    $app->get('/', function() use ($app) {
        $app->render(200,array(
                'msg' => 'welcome to my API!',
            ));
    });

```


#### example output
```json
{
    "msg":"welcome to my API!",
    "error":false,
    "status":200
}

```

## Errors
To display an error just set the `error => true` in your data array.
All requests will have an `error` param that defaults to false.

```php

    $app->get('/user/:id', function($id) use ($app) {

        //your code here

        $app->render(404,array(
                'error' => TRUE,
                'msg'   => 'user not found',
            ));
    });

```
```json
{
    "msg":"user not found",
    "error":true,
    "status":404
}

```

You can optionally throw exceptions, the middleware will catch all exceptions and display error messages.

```php

    $app->get('/user/:id', function($id) use ($app) {

        //your code here

        if(...){
            throw new Exception("Something wrong with your request!");
        }
    });

```
```json
{
    "error": true,
    "msg": "ERROR: Something wrong with your request!",
    "status": 500
}

```

## Supressing rewrite of the error field

By default all content set in the *error* field on the response is overwritten with TRUE or FALSE depending on the HTTP_CODE provided in the render method call. If you need to send additional information in the error field you can disable this override by calling the *overrideError()* method with false before calling *render()*.

```php
  $app->get('/user/:id', function($id) use ($app) {

      //your code here
      $app->view()->overrideError(false);
      $app->render(404,array(
              'error' => array('code' => 404, 'message' => 'not found')
          ));
  });
```

```json
{
    "error": {
      "code": 404,
      "message": "user not found"
    },
    "status":404
}
```

The override also works on data only responses.

```php
  $app->get('/user/:id', function($id) use ($app) {
    //set data only response (no error, status or flash fields)
    $app->view()->dataOnly(true);

    //your code here

    $app->view()->overrideError(false);
    $app->render(404,array(
        'error' => array('code' => 404, 'message' => 'not found')
    ));
  });
```

```json
{
  "error": {
    "code": 404,
    "message": "user not found"
  }
}
```


## Embedding response data and metadata in separate containers
It is possible to separate response metadata and business information in separate containers.

#### To make it possible just init JsonApiView with containers names
```php
   require 'vendor/autoload.php';

    $app = new \Slim\Slim();

    $app->view(new \JsonApiView("data", "meta"));
    $app->add(new \JsonApiMiddleware());
```

#### Response
```json
{
    "data":{
        "msg":"welcome to my API!"
    },
    "meta":{
        "error":false,
        "status":200
    }
}
```


## Routing specific requests to the API
If your site is using regular HTML responses and you just want to expose an API point on a specific route,
you can use Slim router middlewares to define this.

```php
    function APIrequest(){
        $app = \Slim\Slim::getInstance();
        $app->view(new \JsonApiView());
        $app->add(new \JsonApiMiddleware());
    }


    $app->get('/home',function() use($app){
        //regular html response
        $app->render("template.tpl");
    });

    $app->get('/api','APIrequest',function() use($app){
        //this request will have full json responses

        $app->render(200,array(
                'msg' => 'welcome to my API!',
            ));
    });
```


## middleware
The middleware will set some static routes for default requests.
**if you dont want to use it**, you can copy its content code into your bootstrap file.

***IMPORTANT: remember to use `$app->config('debug', false);` or errors will still be printed in HTML***
