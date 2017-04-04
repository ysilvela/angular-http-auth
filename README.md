HTTP Auth Interceptor Module
============================
para AngularJS
-------------

Es una implementacion del concepto descrito en
[Authentication in AngularJS (or similar) based application](http://espeo.eu/blog/authentication-in-angularjs-or-similar-based-application/).

Funciona para todas las releases de AngularJS **1.0.x** and **1.2.x**,
see [releases](https://github.com/witoldsz/angular-http-auth/releases).

Ver demo [aqui](http://witoldsz.github.com/angular-http-auth/)
or switch to [gh-pages](https://github.com/witoldsz/angular-http-auth/tree/gh-pages)
branch for source code of the demo.

Usage
------

- Install via bower: `bower install --save angular-http-auth`
- ...or via npm: `npm install --save angular-http-auth`
- Include as a dependency in your app: `angular.module('myApp', ['http-auth-interceptor'])`

Manual
------

Este modulo instala un interceptor $http y provee del servicio `authService`.

El interceptor $http hace lo siguiente:
La configuracion del cada llamada (la url solicitada, payload y parametros)
de todas las respuestas HTTP 401 es almacenada en un buffer y siempre que se invoque, el mensaje
`event:auth-loginRequired` sera emitido por broadcast a $rootScope.

El servicio `authService` tiene 2 metodos: `loginConfirmed()` y `loginCancelled()`.

El programador es el responsable de invocar a `loginConfirmed()` despues del login. Opcionalmente se puede pasar por argumentos a este metodo lo que se vaya a pasar a loginConfirmed
$broadcast. Esto es muy util, por ejemplo si necesitas pasar deltalles del usuario logado. El`authService` tratara de reintentar las peticiones fallidas con respuesta HTTP 401.

El programador es el responsable de invocar a  `loginCancelled()` cuando la autenticacion ha sido invalidada. Opcionalmente puedes pasar argumentos que serán enviados al metodo loginCancelled via
$broadcast. El `authService` cacnelara todas las peticiones pendientes antes de que falle la peticion 401.

In the event that a requested resource returns an HTTP 403 response (i.e. the user is
authenticated but not authorized to access the resource), the user's request is discarded and
the `event:auth-forbidden` message is broadcast from $rootScope.

#### Ignoring the 401 interceptor

Sometimes you might not want the interceptor to intercept a request even if one returns 401 or 403. In a case like this you can add `ignoreAuthModule: true` to the request config. A common use case for this would be, for example, a login request which returns 401 if the login credentials are invalid.

###Typical use case:

* somewhere (some service or controller) the: `$http(...).then(function(response) { do-something-with-response })` is invoked,
* the response of that requests is a **HTTP 401**,
* `http-auth-interceptor` captures the initial request and broadcasts `event:auth-loginRequired`,
* your application intercepts this to e.g. show a login dialog:
 * DO NOT REDIRECT anywhere (you can hide your forms), just show login dialog
* once your application figures out the authentication is OK, call: `authService.loginConfirmed()`,
* your initial failed request will now be retried and when proper response is finally received,
the `function(response) {do-something-with-response}` will fire,
* your application will continue as nothing had happened.

###Advanced use case:

####Sending data to listeners:
You can supply additional data to observers across your application who are listening for `event:auth-loginConfirmed` and `event:auth-loginCancelled`:

      $scope.$on('event:auth-loginConfirmed', function(event, data){
      	$rootScope.isLoggedin = true;
      	$log.log(data)
      });

      $scope.$on('event:auth-loginCancelled', function(event, data){
        $rootScope.isLoggedin = false;
        $log.log(data)
      });

Use the `authService.loginConfirmed([data])` and `authService.loginCancelled([data])` methods to emit data with your login and logout events.

####Updating [$http(config)](https://docs.angularjs.org/api/ng/service/$http):
Successful login means that the previous request are ready to be fired again, however now that login has occurred certain aspects of the previous requests might need to be modified on the fly. This is particularly important in a token based authentication scheme where an authorization token should be added to the header.

The `loginConfirmed` method supports the injection of an Updater function that will apply changes to the http config object.

    authService.loginConfirmed([data], [Updater-Function])

    //application of tokens to previously fired requests:
    var token = response.token;

    authService.loginConfirmed('success', function(config){
      config.headers["Authorization"] = token;
      return config;
    })

The initial failed request will now be retried, all queued http requests will be recalculated using the Updater-Function.

It is also possible to stop specific request from being retried, by returning ``false`` from the Updater-Function:

    authService.loginConfirmed('success', function(config){
      if (shouldSkipRetryOnSuccess(config))
        return false;
      return config;
    })
