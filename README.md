<p align="center">
<a href="https://packagist.org/packages/waxframework/routing"><img src="https://img.shields.io/packagist/dt/waxframework/routing" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/waxframework/routing"><img src="https://img.shields.io/packagist/v/waxframework/routing" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/waxframework/routing"><img src="https://img.shields.io/packagist/l/waxframework/routing" alt="License"></a>
</p>

# About WaxFrameWork Routing

WaxFramework Routing is a powerful routing system for WordPress plugins that is similar to the popular PHP framework Laravel. This package makes use of the WordPress REST route system and includes its own custom route system, known as the `Ajax Route`.

One of the key features of WaxFramework Routing is its support for middleware. Middleware allows you to perform additional actions before each request.

By using WaxFramework Routing in your WordPress plugin, you can easily create custom routes and middleware to handle a wide variety of requests, including AJAX requests, with ease. This makes it an excellent tool for developing modern and dynamic WordPress plugins that require advanced routing capabilities and additional security measures.

- [About WaxFrameWork Routing](#about-waxframework-routing)
	- [Requirement](#requirement)
		- [Methods structure](#methods-structure)
	- [Installation](#installation)
	- [Configuration](#configuration)
	- [Register Routes In Routes File](#register-routes-in-routes-file)
		- [Rest Route](#rest-route)
			- [Write your first route](#write-your-first-route)
			- [With Controller](#with-controller)
			- [Dynamic Routing](#dynamic-routing)
			- [Route Grouping](#route-grouping)
			- [Resource Controller](#resource-controller)
				- [Actions Handled By Resource Controller](#actions-handled-by-resource-controller)
	- [Middleware](#middleware)
	- [License](#license)


## Requirement

WaxFramework routing requires a dependency injection (DI) container. We do not use any hard-coded library, so you can choose to use any DI library you prefer. However, it is important to follow our DI structure, which includes having the `set`, `get`, and `call` methods in your DI container.

We recommend using [PHP-DI](https://php-di.org/) as it already has these 3 methods implemented in the package.

### Methods structure
Here is the structure of the methods that your DI container should have in order to work with WaxFramework routing:

1. `set` method

	```php
	/**
     * Define an object or a value in the container.
     *
     * @param string $name Entry name
     * @param mixed $value Value, define objects
     */
    public function set( string $name, $value ) {}
	```
2. `get` method

	```php
	
    /**
     * Returns an entry of the container by its name.
     *
     * @template T
     * @param string|class-string<T> $name Entry name or a class name.
     *
     * @return mixed|T
     */
    public function get( $name ) {}
	```
3. `callback` method
	```php
	 /**
     * Call the given function using the given parameters.
     *
     * Missing parameters will be resolved from the container.
     *
     * @param callable $callable   Function to call.
	 * 
     * @return mixed Result of the function.
     */
    public function call( $callable ) {}
	```
## Installation

```
composer require waxframework/routing
```
## Configuration
1. Your plugin must include a `routes` folder. This folder will contain all of your plugin's route files.

2. Within the `routes` folder, create two subfolders: `ajax` and `rest`. These folders will contain your plugin's route files for AJAX and REST requests, respectively.

3. If you need to support different versions of your routes, you can create additional files within the `ajax` and `rest` subfolders. For example, you might create `v1.php` and `v2.php` files within the `ajax` folder to support different versions of your AJAX routes.

4. Folder structure example:
	```
	routes:
	    ajax:
	        api.php
            v1.php
            v2.php
	    rest:
	       api.php
		   v1.php
	```
5. In your `RouteServiceProvider` class, set the necessary properties for your route system. This includes setting the `rest and ajax namespaces`, the versions of your routes, any middleware you want to use, and the directory where your route files are located. Here's an example:
	```php

    <?php

    namespace MyPlugin\Providers;

    use WaxFramework\Contracts\Provider;
    use MyPlugin\Container;
    use WaxFramework\Routing\Providers\RouteServiceProvider as WaxRouteServiceProvider;

    class RouteServiceProvider extends WaxRouteServiceProvider implements Provider {

        public function boot() {

            /**
             * Set Di Container
             */
            parent::$container = new Container;

            /**
             * OR you use PHP-Container 
             * Uses https://php-di.org/doc/getting-started.html
             */
            // parent::$container = new DI\Container();


            /**
             * Set required properties
             */
            parent::$properties = [
                'rest'       => [
                    'namespace' => 'myplugin',
                    'versions'  => ['v1', 'v2']
                ],
                'ajax'       => [
                    'namespace' => 'myplugin',
                    'versions'  => []
                ],
                'middleware' => [
                    'admin' => \MyPlugin\Middleware\EnsureIsUserAdmin::class
                ],
                'routes-dir' => ABSPATH . 'wp-content/plugins/my-plugin/routes'
            ];

            parent::boot();
        }
    }

	```
6. Finally, execute the `boot` method of your `RouteServiceProvider` class using the `init` action hook, like so:

	```php
	add_action('init', function() {
		$route_service_provider = new \MyPlugin\Providers\RouteServiceProvider;
		$route_service_provider->boot();
	});
	```
That's it! Your plugin is now configured with WaxFrameWork Routing system, and you can start creating your own routes and handling requests with ease.

## Register Routes In Routes File

### Rest Route 
`routes/rest/api.php`

#### Write your first route
To create your first RESTful route in WordPress, you can use the `Route` and `Response` classes from the `WaxFramework\Routing` namespace, as shown below:
```php
<?php

use WaxFramework\Routing\Route;
use WaxFramework\Routing\Response;

defined('ABSPATH') || exit;

Route::get('user', function() {
	return Response::send(['ID' => 1, 'name' => 'john']);
});
```

In this example, we're using the `get()` method of the Route class to define a `GET request` to the /user endpoint. The closure passed as the second argument returns a response using the `Response::send()` method, which takes an array of data to be returned in `JSON format`.
#### With Controller
If you prefer to use a controller for your route logic, you can specify the controller and method as an array, as shown below:

```php
Route::get('user', [UserController::class, 'index']);
```
Here, we're using the `get()` method of the Route class to define a `GET request` to the /user endpoint. We're specifying the controller class and method as an array, where `UserController::class` refers to the class name and `index` is the method name.

#### Dynamic Routing
You can use dynamic routing to handle requests to endpoints with dynamic parameters. To define a route with a required parameter, use curly braces around the parameter name, as shown below:

```php
// Required id
Route::get('users/{id}', [UserController::class, 'index']);
```

To define a route with an optional parameter, you can add a question mark after the parameter name, as shown below:

```php
// Optional id
Route::get('users/{id?}', [UserController::class, 'index']);
```
#### Route Grouping
You can group related routes together using the `group()` method. This allows you to apply middleware or other attributes to multiple routes at once. You can create `nested groups` as well, as shown below:

```php
Route::group( 'admin', function() {

    Route::get( '/',  [UserController::class, 'index'] );

    Route::group( 'user', function() {
        Route::get( '/', [UserController::class, 'index'] );
        Route::post( '/', [UserController::class, 'store'] );
        Route::get( '/{id}', [UserController::class, 'show'] );
        Route::patch( '/{id}', [UserController::class, 'update'] );
        Route::delete( '/{id}', [UserController::class, 'delete'] );
    } );
} );
```

#### Resource Controller
Resource routing is a powerful feature that allows you to quickly assign CRUD `(create, read, update, delete)` routes to a controller with a single line of code. To create resource routes, you can use the `resource()` method. Here is an example:

```php
Route::resource( 'user', UserController::class );
```

Resource routing automatically generates the typical CRUD routes for your controller, as shown in the table below:

##### Actions Handled By Resource Controller

| Verb   | URI           | Action |
|--------|---------------|--------|
| GET    | /users        | index  |
| POST   | /users        | store  |
| GET    | /users/{user} | show   |
| PATCH  | /users/{user} | update |
| DELETE | /users/{user} | delete |

With resource routing, you don't have to define each route separately. Instead, you can handle all of the CRUD operations in a single controller class, making it easier to organize your code and keep your routes consistent.

## Middleware

To create a middleware, you need to implement the `Middleware` interface. The `handle` method of the middleware must return a boolean value. If the handle method returns `false`, the request will be stopped immediately.

Here is an example of creating a middleware class named `EnsureIsUserAdmin`:

```php
<?php

namespace MyPlugin\App\Http\Middleware;

use WaxFramework\Routing\Contracts\Middleware;
use WP_REST_Request;

class EnsureIsUserAdmin implements Middleware
{
    /**
    * Handle an incoming request.
    *
    * @param  WP_REST_Request  $wp_rest_request
    * @return bool
    */
    public function handle( WP_REST_Request $wp_rest_request ): bool
    {
        return current_user_can( 'manage_options' );
    }
}
```

Once you have created the middleware, you need to register it in the RouteServiceProvider [Configuration](#configuration).


## License

WaxFramework Routing is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).