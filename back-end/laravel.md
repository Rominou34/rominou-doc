# Laravel Documentation

## Table of contents

* [Installation](#installation)
* Basics
  * [Routes](#routes)
  * [Controllers](#controllers)
  * [Requests](#requests)
  * [Responses](#responses)
  * [Cookies](#cookies)
  * [Middlewares](#middlewares)
  * [Views](#views)
* Databases
  * [Configuration](#configuration)
  * [Query system](#query-system)
  * [Complex queries](#complex-queries)
* Services
  * [Cache](#cache)
  * [Localization](#localization)
  * [Mail](#mail)
* [Blade Templating](#blade-templating)
  * [Basics](#blade-basics)
  * [Control structures](#control-structures)
* Use cases
  * [Combining routes, controllers and views to display and register users](#combining-routes-controllers-and-views-to-display-and-register-users)

# Installation

1. Install Composer
2. From any folder: `composer global require "laravel/installer"` ( also works with Lumen by using `"laravel/lumen-installer"` )
3. Create your project by exectuing this command from the folder where you want it: `composer create-project --prefer-dist laravel/lumen projet`. It also works with this one: `laravel new projet`
4. Go to the `.env` file and change these settings ( localhost on WAMP ):

  ```
  DB_HOST=127.0.0.1:3306
  DB_PORT=3306
  DB_DATABASE=DB_NAME
  DB_USERNAME=root
  DB_PASSWORD=''
  ```
5. Screw that shit, I'll make my own framework

# Basics

## Routes

**Everything regarding routes happens in `app/Http/routes.php`**

*To create a new basic route:*

```php
Route::get('/', function () {
    return 'Hello world!';
});
```

This will return 'Hello World!' if you make a GET request on `/`

*To redirect depending on "folders" in your url*

```php
Route::get('admin', function () {
    return 'Hi admin!';
});
```

This will return 'Hi admin' if you make a GET request on `admin/`.

*To create a route using a controller:*

```php
Route::get('/', 'Controller@function');
```

This will call the function `function()` from the controller `Controller` if you make a GET request `/`

*To redirect if there's a parameter in your url*

```php
Route::get('msg/{id}', 'Controller@msg');
```

This will call the function `msg()` in the controller `Controller` if you make a GET request on `msg/` with a parameter

**Note:** The parameters automatically get passed to the controller so **don't call the controller like this:**

```php
Route::get('msg/{id}', 'Controller@msg(id)');
```

It's possible to filter the parameters using RegEx ( regulars expressions ):

```php
Route::get('{lang}/welcome', function() {
  return view('welcome');
})
->where('lang', 'fr|en|es|it|de|po');
```

## Controllers

The default controller can be found at `app/Http/Controllers/Controller.php`

If you want to create your own controller, you'll have to make it a child class of the base controller:

1. Import the base controller by putting `use Illuminate\Routing\Controller as BaseController;` at the beginning of your file
2. Declare your controller like this:
```php
class myController extends BaseController {
  // Your controller's stuff
}
```

## Requests

You can retrieve the parameters of a request from your controller by setting a `Request` parameter in your function, which will allow you to get all the parameters:
```php
public function post(Request $req) {
  $id = $req->input('id'); // We get the 'id' parameter form the request
  $name = $req->input('name'); // We get the 'name' parameter
}
```

You can also set a fallback value in case the parameter wasn't set:

```php
$name = $req->input('name', 'Bob'); // If the 'name' parameter is not set, it will default to 'Bob'
```

## Responses

First, you'll need to import the Response class `use Illuminate\Http\Response`

With this, you can customize your response's header and status, here is the basic syntax:
```php
Route::get('/', function () {
    return (new Response($content, $status))
                  ->header($var, $value);
});
```

And here is an example:
```php
Route::get('/', function () {
    return (new Response('{value: 1}', 200))
                  ->header('Content-Type', 'application/json');
});
```

In this example, I returned JSON, as I set `Content-Type: application/json` in the header, and the page was returned with a status 200.

You can of course change the status of the response, and you can also return custom header values ( I still hasn't find any use for that tho ):

```php
Route::get('/', function () {
    return (new Response('Hi there', 404))
                  ->header('Content-Type','text/html')
                  ->header('And-his-name-is','JOHN CENA');
});
```

Here we set the status of the response to 404 and we set a custom header element to JOHN CENA

**NOTE: The convention for custom HTTP headers is to start their name with 'X-' so this header should actually be rename 'X-And-his-name-is'**

*Alternative syntax*

You can pass an array of headers as a parameter instead of passing all headers one by one:
```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

#### Return options

If you wish to return a view as your response's content, you can use the `view` method:
```php
return response()
            ->view('view', $data)
```

And you can also easily return JSON with the json() method:
```php
return response()->json(['name' => 'Rominou', 'age' => 19, 'sex' => 'big']);
```

*More to come once I'll get better at Laravel*

## Cookies

You can set cookies in your responses the same way you set headers:

```php
return response($content)
                ->cookie('name', 'value');
```

You can pass more parameters in your cookie constructor for more customization:

```php
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

## Middlewares

HTTP Middlewares are used to filter and redirect HTTP requests before processing them with a controller

#### Creating a Middleware

* Navigate to the root folder of the project and execute this command:
  ```
  php artisan make:middleware MiddlewareName
  ```
* A file `MiddlewareName.php` wille be created in `app/Http/Middleware`
* Inside the file `app/Http/Kernel.php`, add this in the array `protected $middlewareGroups`:
```php
'MiddlewareName' => [
      \App\Http\Middleware\MiddlewareName::class,
],
```
* Finally, to add your middleware to a set of routes, go into `routes.php` and assign your routes to the middleware:
  * Either add the routes inside the middleware declaration:
  ```php
  Route::group(['middleware' => ['MiddlewareName']], function () {
    Route::get('/', function () { /* Do something */ });
      Route::post('/','Controller@function');
  });
  ```
  * Or assign the middleware to each route:
  ```php
  Route::get('/', ['middleware' => 'MiddlewareName', function() {}]);
  // Assign multiple middlewares to a route
  Route::post('/', ['middleware' => ['Mid1', 'Mid2'], function() {}]);
  ```
  * If you want to assign your middleware to each request, put it in the `protected $middleware` inside `app/Http/Kernel.php`:
  ```php
  protected $middleware = [
        \App\Http\Middleware\MiddlewareName::class,
    ];
  ```
  * You can assign a group of middleware by grouping them inside `app/Http/Kernel.php` and assigning the group:
  ```php
  protected $middlewareGroups = [
        'midGroup' => [
            \App\Http\Middleware\Middleware1::class,
            \App\Http\Middleware\Middleware2::class,
        ],
  ```
  and then in your file `routes.php`:
  ```php
  Route::group(['middleware' => ['midGroup']], function () {
    Route::get('/', function () {
      /* Do something */
    });
  });
  ```
#### How it works


#### Before / After Middlewares

Middlewares can do their job *before* or *after* a request is being processed by the application. You choose this by putting your code *before* or *after* `$next($request)`:

```php
public function handle($request, Closure $next)
{
  // Your action will be performed before the request is being process
  return $next($request);
}

public function handle($request, Closure $next)
{
  //We send the request deeper in the application
  $response = $next($request);

  // And we do our thing when it comes back
  return $response;
}
```

**Terminating middleware**

Sometimes you want the middleware to do some work after the HTTP response has been sent to the browser ( in the example of a cache system, you send the response to the browser and then cache it ). Terminating middlewares are here for that, just put a `terminate()` function inside your middleware and it will terminate the shit our of Sarah Connor:

```php
public function terminate($request, $response)
{
    // Cache your page
}
```

You can put both `handle()` and `terminate()` functions inside one middleware so he does work on both sides.

You can see a full code example [here](#using-a-middleware-to-cache-pages)

## Views

Laravel views contain the HTML code we will display on the website, they are located in the folder `resources/views`.

To create a new view, just create a new files in the folder: `resources/views/welcome.php`

Then, put some HTML code in it:

```html
<div class="msg">
  <h1>Welcome <?php echo $name; ?></h1>
</div>
```

Then, call your view ( from your route or controller ) like this:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'James']);
});
```

You can also pass an object as a parameter:

```html
<div class="userInfo">
  <p>ID: <?php echo $user->id; ?></p>
  <p>Name: <?php echo $user->name; ?></p>
</div>
```

# Databases

## Configuration

First of all, you'll need to modify some settings inside the file `config/database.php`:

```
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', 'localhost'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
],
```

Normally, if you correctly configured your environment variables inside the `.env` file like you had to do it [here](#installation), this code will automatically load them

This is the code if you are using MySQL, if you use PostGreSQL or SQLite there are other lines to change ( in the same file )

## Query system

**First of all, don't forget `use DB;` in order to use Databases**

Laravel works a different way than pure PHP when it comes to SQL queries. In fact, while you actually use SQL in queries with pure PHP, with Laravel you use functions from the `DB` object that will construct the query so you never really deal with true SQL ( it's pretty much the same as MeteorJS )

Here is an example query equal to `SELECT * FROM Users`:

```php
$users = DB::table('users')->get();
```

This is pretty much how all queries will be done using Laravel: **No SQL, just functions that build into SQL**

If you want to display the users:
```php
foreach ($users as $user) {
    echo $user->name;
}
```

## Complex queries

Now that we saw how queries were done using Laravel we can go a little deeper, and you'll see that it still follows SQL syntax while not using SQL at all.

For example, if you want to select a single row using a condition ( `SELECT * FROM Users WHERE name = John` ) it will look like this:

```php
$user = DB::table('users')->where('name', 'John')->first();
```

This is still a `SELECT *` tho, if you want to extract only a column from this, use `->value()`:

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

If you have a lot of data and want to extract it divided into little groups, use the `chunk()` function

**Note:** This function is a little bit weird at first. Let's say you have 250 products and run this command:
```php
DB::table('table')->orderBy('id')->chunk(100, function($products) {
  foreach ($products as $prod) {
    echo($prod->name);
  }
});
```

It will do this:
* Take 100 products from the DB
* Display all of them
* Take the next 100
* Display them
* Take the last 50

So you will actually end up with all your 250 products displayed, but they just won't be displayed in one run ( which will make the user wait if you have a lot ), they will display by groups of 100.

**Important note:** If you just want the 100 products ( you want it to stop after one run ), make the function retrun false:
```php
DB::table('table')->orderBy('id')->chunk(100, function($products) {
  foreach ($products as $prod) {
    echo($prod->name);
  }
  return false;
});
```


# Services

## Cache

Laravel makes Caching items pretty easy vie the `Cache` facade. Just put some caching functions inside a middleware on your static pages and you're good.

#### Retrieve items

You can retrieve the value of an item in the cache with the `Cache::get()` function:
```php
$value = Cache::get('key');
$value = Cache::get('key', 'default');
```

You can also retrieve items and delete them at the same time by using the `Cache::pull()` function:
```php
$value = Cache::pull('key');
```

Both functions will return `null` if no item with the specified key exists

#### Storing items

You can store items using the `Cache::put()` function:

```php
// You can specify the duration of the item by giving the number of minutes
Cache::put('key', 'value', $minutes);

// Or you can use a PHP DateTime ( if you use Carbon don't forget to use Carbon\Carbon; )
$expiresAt = Carbon::now()->addMinutes(10);
Cache::put('key', 'value', $expiresAt);
```

If you want to store an item forever, use the `Cache::forever()` function:
```php
Cache::forever('key', 'value');
```

The method `Cache::add()` will add an item to the cache only if it doesn't exist:
```php
// Returns true if it added the item or false if it already existed
Cache::add('key', 'value', $minutes);
```

#### Deleting items

If you want to delete an item from the cahce use the `Cache::forget()` function:
```php
Cache::forget('key');
```

You can either flush the whole cache with the `flush()` method, however **it doesn't espect the cache prefix** so if you run multiple Laravel websites it will delete the cache of all of them:
```php
Cache::flush();
```

#### Helper functions

**Checking if an item exists**

You can check if an item exists in the cache using the `Cache::has()` function:
```php
Cache::has('key')
```

**Increment / Decrement value of an item**

You can easily increment or decrement the value of an item using the corresponding functions:
```php
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

## Localization

Laravel comes with an easy-to-use Localization system, allowing you to easily make your website readable in different languages. It works best with Blade templating language and allows you to replace text in the HTML by variables, which will then be transformed into different texts depending on the language.

#### Initial configuration

First, set a fallback language inside `config/app.php`:

```
'fallback_locale' => 'fr',
```

#### Setting the language

To set the language, you can use the `App::setLocale()` method:
```php
Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);
});
```

The snippet of code will set the language to `en` if you make a request on `welcome/en`.

**TIP:** If you use this system ( URL parameters ) to change the language, use a RegEx constraint on said URL parameters so a route `welcome/test` won't redirect on `welcome.php` while setting the language to `test`:
```php
Route::get('welcome/{lang}', function($lang) {
  App::setLocale($lang);
})
->where('lang', 'fr|en|es|it');
```

#### Setting your language variables

Now, just go into the folder `resources/lang` and create a folder for each language ( `resources/lang/fr`, `resources/lang/en` etc. )

These are the folders in which you'll create your localization files.

**Each file must be present under the same name inside all folders**

that means that your structure will look like this:
```
resources
  |- lang
    |- fr
    |  |- base.php
    |- en
       |- base.php
```

These files will be used to declare your language variables, which will be declared using an array of keyed strings. Here is pretty much how it will look like:

```php
<?php

return [
    'welcome' => 'Welcome to our application'
];
```

And in the corresponding file in the `resources/lang/fr` folder:

```php
<?php

return [
    'welcome' => 'Bienvenue dans notre application'
];
```

#### Using your language variables

Now, to call your variables, just use the function `trans('file_path.variable')`

For example, we will display the welcoming message from the previous part ( supposing the file is name `base.php` ):
```php
// Pure PHP
echo(trans('base.welcome'));

// Blade templating
{{ trans('base.welcome') }}
```

And as a result, if our language was set to `en`, we'll see `"Welcome to our application"`, else if it was set to `fr` we'll see `"Bienvenue dans notre application"`

## Mail

#### Setting up your mail config

So first, go into the `.env` file located at the root of your folder and change the mail settings:
```
MAIL_DRIVER=smtp
MAIL_HOST= *Mail host*
MAIL_PORT= *Port*
MAIL_USERNAME= *Username*
MAIL_PASSWORD= *Password*
MAIL_ENCRYPTION=null
```

To test mail sending, I personally use [Mailtrap.io](https://mailtrap.io/), which allows you to send emails to fake inboxes ( the free plan is at 2 emails/seconds ), which is perfect to fool around or test your shit.

#### Bracing yourself

**First of all**, you'll need to use the Laravel Object `User` to send emails.

So, *what is this shit ?*

The User object is located at `App/User`, and it allows you to manage users ( in the context of e-mails, you create or retrieve a user and use his e-mail and name to send your mail )

Don't forget the usual imports ( we Python now ):
```
use Mail;
use App\User;
```

#### Sending e-mails

So here's a sample code I used to send a test e-mail to myself:

```php
public function testMail(Request $request)//, $id)
    {
        // I create a new user
        $user = new User;

        // I set it an e-mail and a name
        $user->email = 'ayxiit@gmail.com';
        $user->name = 'Rominou';

        // I was too lazy to create a view so I send a 503 error page
        Mail::send('errors.503', ['user' => $user], function ($m) use ($user) {
            $m->from('hello@app.com', 'Your Application');

            $m->to($user->email, $user->name)->subject('Here is a nice 503 page');
        });

        // Just some feedback
        return 'Mail sent';
    }
```

**Note:** If you're using Mailtrap.io, don't look in the inbox you put in the e-mail ( here "ayxiit@gmail.com" ), the mail won't be sent but it will be displayed in your fake inbox on Mailtrap.io ( that's what the site is used for, sending e-mails to fake inboxes without spamming the shit out of people )

So here's a recap of what this function did:
* Create a user named 'Rominou', with a e-mail address of 'ayxiit@gmail.com'
* Send an e-mail to this user, with these characteristics:
  * The sender is named 'Your Application' and its e-mail is 'hello@app.com'
  * The subject of the e-mail is 'Here is a nice 503 page'
  * The content of the e-mail is the view 'errors.503'

As you can see, the content of the e-mail is a view, so you'll have to create a folder `emails` containing all your views so you don't get lost.

Also, as with every views, you can pass parameters, so if you use a code like this `{{ $user->name }}` ( or `<?php echo($user->name); ?>` in pure PHP ), you are able to display the name of the user in your mail, that's neat.

#### Advanced functions

If you wish to send e-mails without having to create a new `App/User` each time, just use the e-mail address directly with the `->to(adress, name)` method:
```php
Mail::send('emails.welcome', $data, function ($message) {
    $message->from('us@example.com', 'Laravel');
    // Without the name
    $message->to('foo@example.com')->cc('bar@example.com');
    // With the name
    $message->to('foo@example.com', 'Rominou')->cc('bar@example.com');
});
```

Note I also used the `->cc(address)` method, meaning I sent a carbon copy to "bar@example.com"

There are a lot of methods like that speaking for themselves, so I'll just post them here without explaining much:
```php
$message->from($address, $name = null);
$message->sender($address, $name = null);
$message->to($address, $name = null);
$message->cc($address, $name = null);
$message->bcc($address, $name = null);
$message->replyTo($address, $name = null);
$message->subject($subject);
$message->priority($level);
$message->attach($pathToFile, array $options = []);

// Attach a file from a raw $data string...
$message->attachData($data, $name, array $options = []);

// Get the underlying SwiftMailer message instance...
$message->getSwiftMessage();
```
# Blade Templating

Blade is the templating system of Laravel, used to insert content inside a HTML template ( because `<?php echo();?>` is 2 hazbeen )

To use Blade on a PHP file, use the .blade.php file extension, and just return the file like any view using the `view()` function:

```php
Route::get('blade', function () {
    return view('template');
});
```

## Blade Basics

**Echoing a variable**

Display some variable using `{{ $var }}`

You can choose to echo a variable with a default fallback value like this:

```php
{{ $name or 'Default' }}
```

**Include files**

Include another file using `@include('header')`

**NOTE:** It is possible to include while using values, for example you can put `<header>{{ $title }}</header>` in your file `header.blade.php` and include it like this:

```php
@include('header', ['title'=>'The header's title goes here'])
```

**Blade comments**

You can put comments inside your Blade files that won't appear in your rendered HTML file ( like PHP comments and contrary to HTML comments )

```php
{{-- This comment will not be present in the rendered HTML --}}
```

## Control structures

You can ( and should ) use control structures like if, else, loops, and all these cool things

**If - else**

```php
@if (id == 1)
  // Do something
@elseif (id > 1)
  // Do some other things
@else
  // Do nothing
@endif
```

**Unless**

```php
@unless (id == 1)
  You are not admin you pleb
@endunless
```

**Loops**

Blade actually offers a lot of loops so you have a lot of choice to do things

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

**Breaking loops**

You can break loops using `@break` or skip the current iteration using `@continue`

```php
@foreach ($users as $user)
    @if($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if($user->number == 5)
        @break
    @endif
@endforeach
```

# Use cases

#### Combining routes controllers and views to display and register users

Here are example files `routes.php`, `ControllerUsers.php` and `viewUser.php` that act like this:
* GET request on `user/` shows all users
* GET request on `user/{id}` shows the user with the corresponding id
* POST request on `user/` inserts a new user in the BDD with the required parameters
* PUT request on `user/{id}` inserts or replace the user with the corresponding id ( used to update infos )
* DELETE request on `user/{id}` deletes the user with the corresponding id

*routes.php*
```php
Route::get('user/', 'ControllerUsers@getAllUsers');
Route::get('user/{id}', 'ControllerUsers@getUserById');
Route::post('user/', 'ControllerUsers@saveUser');
Route::put('user/{id}', 'ControllerUsers@updateUser');
Route::delete('user/{id}', 'ControllerUsers@deleteUser');
```

*ControllerUsers.php*
```php
namespace App\Http\Controllers;
use DB; // You can't use databases without this

// Blabla import everything
use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Routing\Controller as BaseController;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Http\Request;

class ControllerUsers extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;

    public function getAllUsers()
    {
        $users = DB::select('select * from users');
        foreach($users as $user) {
          echo(view('viewUser', ['user' => $user]));
        }
    }

    public function getUserById(Request $request) {
      $id = $request->input('id');
      $user = DB::select('select * from users where id =  ?', [$id]);
      echo(view('viewUser', ['user' => $user]));
    }

    public function saveUser(Request $request)
    {
      $i = null;
      $n = $request->input('name');
      $s = $request->input('surname');
      DB::insert('insert into users (id, name, surname) values (?, ?, ?)', [$i, $n, $s]);
    }

    public function updateUser(Request $request) {
      // Don't have time to test it so I'm gonna put a placeholder kappa
      $id = $request->input('id');
      updateTheBitch($id);
    }

    public function deleteUser(Request $request) {
      // Same as updateUser
      $id = $request->input('id');
      updateThisNigga($id);
    }
}
```

*viewUser.php*
```html
<div class="userInfo">
  <p>User id: <?php echo $user->id; ?></p>
  <p>User name: <?php echo $user->name; ?></p>
  <p>User surname: <?php echo $user->surname; ?></p>
</div>
```


#### Using a middleware to cache pages

This is a basic middleware I coded during my internship in order to cache static pages of a website

*CachePage.php*
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Cache;
use Carbon\Carbon;
use Illuminate\Http\Response;

class CachePage
{
    /*
    * Handles the incoming request and checks the cache
    * If the page already exists in the cache, return the one from the cache,
    * else return the actual webpage from the site
    */
     public function handle($request, Closure $next)
     {
       if(Cache::has($request->fullUrl())) {
         return response(Cache::get($request->fullUrl()));
       } else {
         return $next($request);
       }

     }

     /*
     * After the response ( the webpage ) has been sent to the browser,
     * it will save it in the cache if it isn't already there
     * NOTE: I know I could use Cache:add()
     */
     public function terminate($request,Response $response)
     {
       // The pages will expire after one day ( 24 * 60 minutes = 1440 )
       $expiresAt = Carbon::now()->addMinutes(1440);
       if(!Cache::has($request->fullUrl())) {
         Cache::put($request->fullUrl(),$response->getContent(), $expiresAt);
       }
    }
}
```
