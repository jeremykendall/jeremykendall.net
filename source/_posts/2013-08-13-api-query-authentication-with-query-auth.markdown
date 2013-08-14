---
layout: post
title: "API Query Authentication with Query Auth"
date: 2013-08-13 12:57
comments: true
categories: [PHP, Composer, API, Authentication, Development]
---

Most APIs require some sort of query authentication: a method of signing API
requests with an API key and signature.  The signature is usually generated
using a shared secret.  When you're consuming an API, there are (hopefully) easy
to follow steps to create signatures. When you're writing your own API, you
have to whip up both server-side signature validation and a client-side
signature creation strategy. [Query Auth](https://github.com/jeremykendall/query-auth)
endeavors to handle both of those tasks; signature creation and signature validation.

## Philosophy

Query Auth is intended to be -- and is written as -- a bare bones library.  Many of
niceties and abstractions you'd find in a fully featured API library or SDK are
absent.  The point of the library is to provide you with the ability to
focus on writing the meat of your API while offloading the authentication bits.

## What's Included?

There are three components to Query Auth: request signing for API consumers
and creators, request signature validation for API creators, and API key and
API secret generation.

### Request Signing

``` php
$collection = new QueryAuth\NormalizedParameterCollection();
$signer = new QueryAuth\Signer($collection);
$client = new QueryAuth\Client($signer);

$key = 'API_KEY';
$secret = 'API_SECRET';
$method = 'GET';
$host = 'api.example.com';
$path = '/resources';
$params = array('type' => 'vehicles');

$signedParameters = $client->getSignedRequestParams($key, $secret, $method, $host, $path, $params);
```

`Client::getSignedRequestParams()` returns an array of parameters to send via
the querystring (for `GET` requests) or the request body. The parameters are
those provided to the method (if any), plus `timestamp`, `key`, and `signature`.

### Signature Validation

``` php
$collection = new QueryAuth\NormalizedParameterCollection();
$signer = new QueryAuth\Signer($collection);
$server = new QueryAuth\Server($signer);

$secret = 'API_SECRET_FROM_PERSISTENCE_LAYER';
$method = 'GET';
$host = 'api.example.com';
$path = '/resources';
// querystring params or request body as an array,
// which includes timestamp, key, and signature params from the client's
// getSignedRequestParams method
$params = 'PARAMS_FROM_REQUEST'; 

$isValid = $server->validateSignature($secret, $method, $host, $path, $params);
```

`Server::validateSignature()` will return either true or false.  It might also
throw one of three exceptions:

* `MaximumDriftExceededException`: If timestamp is too far in the future
* `MinimumDriftExceededException`: It timestamp is too far in the past
* `SignatureMissingException`: If signature is missing from request params

Drift defaults to 15 seconds, meaning there is a 30 second window during which the
request is valid. The default value can be modified using `Server::setDrift()`.

### Key Generation

You can generate API keys and secrets in the following manner.

``` php
$randomFactory = new \RandomLib\Factory();
$keyGenerator = new QueryAuth\KeyGenerator($randomFactory);

// 40 character random alphanumeric string
$key = $keyGenerator->generateKey();

// 60 character random string containing the characters
// 0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ./
$secret = $keyGenerator->generateSecret();
```

Both key and secret are generated using Anthony Ferrara's [RandomLib](https://github.com/ircmaxell/RandomLib)
random string generator.

## That's Kinda Ugly, Dude

As I pointed out, the Query Auth library is pretty bare bones.  There are a lot
of opportunities for abstraction that would make the library much easier to use
and much nicer to look at.  If I added them to Query Auth, however, that
would lock library users into whichever HTTP client I chose to use. The same
concern would go for whatever other abstractions I decided on. The point here is
to offload query authentication, and only query authentication, to the Query Auth
library.

## Sample Implementation

In order to demonstrate how one might implement the Query Auth library, I've whipped
up a [sample implementation](https://github.com/jeremykendall/query-auth-impl) for you.

The sample uses [Vagrant](http://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/)
to allow you to see the whole thing in action.  [Slim Framework](http://slimframework.com/)
runs the API, [Guzzle](http://guzzlephp.org/) is used to make requests to the API,
and both a `GET` and `POST` request are implemented.  [JSend](https://github.com/shkm/JSend), 
[Jamie Schembri's](https://twitter.com/shkm) PHP implementation of the 
OmniTI [JSend specifiction](http://labs.omniti.com/labs/jsend), is used to send 
messages back from the API, and [Parsedown PHP](https://github.com/erusev/parsedown),
Emanuil Rusev's Markdown parser for PHP, is used to render the sample implementation's
documentation.

### Request Signing

In the sample implementation, request signing has been abstracted in the
`Example\ApiRequestSigner` class. Signing requests is now as simple as passing
the request object and credentials object to the `signRequest` method:

``` php
/**
 * Signs API request
 *
 * @param RequestInterface $request     HTTP Request
 * @param ApiCredentials   $credentials API Credentials
 */
public function signRequest(RequestInterface $request, ApiCredentials $credentials)
{
    $signedParams = $this->client->getSignedRequestParams(
            $credentials->getKey(),
            $credentials->getSecret(),
            $request->getMethod(),
            $request->getHost(),
            $request->getPath(),
            $this->getParams($request)
            );

    $this->replaceParams($request, $signedParams);
}
```

### Signature Validation

In the sample implementation, signature validation has been abstracted in the
`Example\ApiRequestValidator` class. Validating request signatures is now as
simple as passing the request object and credentials object to the `isValid`
method:

``` php
/**
 * Validates an API request
 *
 * @param  Request        $request     HTTP Request
 * @param  ApiCredentials $credentials API Credentials
 * @return bool           True if valid, false if invalid
 */
public function isValid(Request $request, ApiCredentials $credentials)
{
    return $this->server->validateSignature(
        $credentials->getSecret(),
        $request->getMethod(),
        $request->getHost(),
        $request->getPath(),
        $this->getParams($request)
    );
}
```

### Signing a GET Request

Signing a request is now extremely clean and simple.  Here's the `GET` example from
the sample implementation.

```php
/**
 * Sends a signed GET request which returns a famous mangled phrase
 */
$app->get('/get-example', function() use ($app, $credentials, $requestSigner) {

    // Create request
    $guzzle = new GuzzleClient('http://query-auth.dev');
    $request = $guzzle->get('/api/get-example');

    // Sign request
    $requestSigner->signRequest($request, $credentials);

    $response = $request->send();

    $app->render('get.html', array('request' => (string) $request, 'response' => (string) $response));
});
```

### Validating a GET Request

Validating a `GET` request is equally clean and simple.  Note the `try/catch` that
handles possible exceptions from the validation class.

``` php
/**
 * Validates a signed GET request and, if the request is valid, returns a
 * famous mangled phrase
 */
$app->get('/api/get-example', function () use ($app, $credentials, $requestValidator) {

    try {
        // Validate the request signature
        $isValid = $requestValidator->isValid($app->request(), $credentials);

        if ($isValid) {
            $mistakes = array('necktie', 'neckturn', 'nickle', 'noodle');
            $format = 'Klaatu... barada... n... %s!';
            $data = array('message' => sprintf($format, $mistakes[array_rand($mistakes)]));
            $jsend = new JSendResponse('success', $data);
        } else {
            $jsend = new JSendResponse('fail', array('message' => 'Invalid signature'));
        }
    } catch (\Exception $e) {
        $jsend = new JSendResponse('error', array(), $e->getMessage());
    }

    $response = $app->response();
    $response['Content-Type'] = 'application/json';
    echo $jsend->encode();
});
```

### Sample Request and Response

The code above produces the below request and response:

#### Request

```
GET /api/get-example?key=ah5yEgQzjuFsC9nWsRI4Nar3ikOqWVPcD3OntHpg&timestamp=1376416267&signature=3DqimkvigYBorGi8wHfil9lB8oCWhB%2BHYt6rVfE4zx4%3D HTTP/1.1
Host: query-auth.dev
User-Agent: Guzzle/3.7.2 curl/7.22.0 PHP/5.5.1-2+debphp.org~precise+2
```

#### Response

```
HTTP/1.1 200 OK
Date: Tue, 13 Aug 2013 17:51:07 GMT
Server: Apache/2.4.6 (Ubuntu)
X-Powered-By: PHP/5.5.1-2+debphp.org~precise+2
Content-Length: 75
Content-Type: application/json

{"status":"success","data":{"message":"Klaatu... barada... n... necktie!"}}
```

## Wrapping Up

So there you have it: [QueryAuth](https://github.com/jeremykendall/query-auth)
to sign and validate API requests (and generate keys and secrets!) and a
[sample implementation](https://github.com/jeremykendall/query-auth-impl) to
get you going.  If you find this helpful, or have any questions or comments,
please let me know.  If you find any horrible mistakes, please feel free to
submit an [issue](https://github.com/jeremykendall/query-auth/issues) or a
[pull request](https://github.com/jeremykendall/query-auth/pulls), or you can
always submit the offending code to [CSI: PHP](http://csiphp.com) :-)
