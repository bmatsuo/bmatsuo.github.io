---
layout: post
title:  "On Protected Resources: Injection"
date:   2013-10-06 23:49:37 -0700
categories: webapps security xss html5 draft
---
In this post I am going to focus on cross-site scripting (XSS) and content
injection. It is all about protecting your users and your content from 
malicious third party payloads injections. In the classic case this injection
takes the form of javascripts. But, potentially more now with HTML5, injections
of other content can open attack vectors.

Almost all popular modern HTML authoring and templating frameworks provide
safe-by-default escaping behavior. This movement has greatly reduced XSS
vunerabilities in websites. But, a common pactice today is to load assets
(javascripts, css, etc.) from many vendors on a page. There are measures one
can and should take to vet these assets and ensure their trustworthy nature.
But, there is always the possibility of exploits and other unintended behavior
compromising your site's security.

##Enter Content Security Policy

Content Security Policy (CSP) is a W3C Candidate Recommendation allows enforcing
granular restrictions on page content. It defines inclusive lists of directives.
Most directives allow loading a particular kind of resource from specified
locations. Although there is a directive of applies global content
restrictions.

###Source declaration

Source lists are a space separated list of schemes/hosts. They specify the set
of locations associated resources are permitted to be loaded from. There are
several special values in source lists. A lone `'none'` means that this type
of resource cannot be loaded. The value `'self'` allows resources orginating
from the same domain can be loaded.

###Global content restriction

The `sandbox` directive is meant to place global content restrictions on the
protected resource. This is similar to iframe sandboxing (citation?). Among
other things it controls `window.top.location` hijacking.

##Important directives

The most important directive is `default-src`. It is used to place a general
policy on external connections. Appropriate usage of CSP seems to be to place
a fairly restrictive `defailt-src` directive on resources and override
specific directives as necessary.

###XSS directives

There are several specific directives important for XSS. The `script-src`
directive is the most important. It controls where javascript assets can be
loaded from (including inline and eval).

The `connect-src` directive is also important for XSS. It controls hosts
javascript can connect to for XHR, web sockets, etc.

Several `sandbox` directive values also target XSS vulnerabilities. A sandboxed
resource must `allow-scripts` must be provided in order to execute javascript
assets. A sandboxed resource can only access cookies if `allow-same-origin` is
provided (unconfirmed).

##Policy violation reporting

The W3C standard defines a simple protocal for browsers to (more or less)
anonymously report attempted CSP violations to web services for investigation.
This is pretty cool. The `report-uri` directive defines a list of uris to which
browsers POST JSON violation report documents.

I do wonder if a naive policy and server implementation could result in a
browser-based DDOS attack from CSP violation reports :p

##Compliance with the W3C standard

[Firefox compliance](https://bugzilla.mozilla.org/show_bug.cgi?id=663566) is
still in progress. Chrome compliance from
[the docs](http://developer.chrome.com/extensions/contentSecurityPolicy.html)
and from my own testing appears to be pretty good. Safari apparently doesn't
support it at all. IE 10 does not support it according to the
[W3C document](http://www.w3.org/TR/CSP/).

All browsers support various deprecated versions of CSP. Safari and Chrome
supports `X-WebKit-CSP` (unconfirmed). Firefox and IE (10) support
`X-Content-Security-Policy` (unconfirmed). Hopefully with adoption of the CSP
standard cross browser support will improve by the next iteration of IE. That's
a shitty metric. But I trust Mozilla to have their act together and no one gives
a fuck about Safari.

##Conclusion

As the standard gets adopted, CSP should become a web standard tool in all web
developer's toolchests. Proper HTML entity escaping is necessary, but CSP
completely blocks anything that manages to sneak through.
