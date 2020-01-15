---
id: csrf
title: CSRF Protection
sidebar_label: CSRF Protection
---

## Introduction

Octopy makes it easy to protect your application from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) attacks. Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user.

Octopy automatically generates a CSRF "token" for each active user session managed by the application. This token is used to verify that the authenticated user is the one actually making the requests to the application.

Anytime you define an HTML form in your application, you should include a hidden CSRF token field in the form so that the CSRF protection middleware can validate the request. You may use the `@csrf` Blade directive to generate the token field:

```html
<form method="POST" action="/profile">
    @csrf
    ..
</form>
```

The `VerifyCSRFToken` [middleware](/docs/middleware), which is included in the `global` middleware group, will automatically verify that the token in the request input matches the token stored in the session.

## Excluding URIs

Sometimes you may wish to exclude a set of URIs from CSRF protection. For example, if you are using [Stripe](https://stripe.com) to process payments and are utilizing their webhook system, you will need to exclude your Stripe webhook handler route from CSRF protection since Stripe will not know what CSRF token to send to your routes.

Typically, you should place these kinds of routes outside of the `global` middleware group that the `RouteServiceProvider` applies to all routes in the `app/Route/Web.php` file. However, you may also exclude the routes by adding their URIs to the `$except` property of the `VerifyCSRFToken` middleware:

```php
<?php

namespace App\HTTP\Middleware;

use Octopy\HTTP\Middleware\VerifyCSRFToken as Middleware;

class VerifyCSRFToken extends Middleware
{
    /**
     * @var array
     */
    protected $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ];
}
```

## X-CSRF-TOKEN

In addition to checking for the CSRF token as a POST parameter, the `VerifyCSRFToken` middleware will also check for the `X-CSRF-TOKEN` request header. You could, for example, store the token in an HTML `meta` tag:

```html
<meta name="token" content="{{ csrf() }}">
```
Then, once you have created the `meta` tag, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications:

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="token"]').attr('content')
    }
});
```

## X-XSRF-TOKEN

Octopy stores the current CSRF token in a `XSRF-TOKEN` cookie that is included with each response generated by the framework. You can use the cookie value to set the `X-XSRF-TOKEN` request header.
This cookie is primarily sent as a convenience since some JavaScript frameworks and libraries, like Angular and Axios, automatically place its value in the `X-XSRF-TOKEN` header.