---
layout:     post
title:      OAuth 2.0 Libraries
subtitle:   OAuth 2.0 is the next evolution of the OAuth protocol which was originally created in late 2006. OAuth 2.0 focuses on client developer simplicity while providing specific authorization flows for web applications, desktop applications, mobile phones, and living room devices. This specification is being developed within the IETF OAuth WG and is based on the OAuth WRAP proposal.
date:       2014-11-18
author:     Longniao
header-img: img/post-oauth.jpg
catalog: true
tags:
    - OAuth
---
            
## OAuth 2.0

![OAuth 2.0 logo](http://oauth.net/images/oauth-2-sm.png)OAuth 2.0 is the next evolution of the OAuth protocol which was originally created in late 2006. OAuth 2.0 focuses on client developer simplicity while providing specific authorization flows for web applications, desktop applications, mobile phones, and living room devices. This specification is being developed within the [IETF OAuth WG](https://www.ietf.org/mailman/listinfo/oauth) and is based on the [OAuth WRAP](http://wiki.oauth.net/OAuth-WRAP) proposal.

Questions, suggestions and protocol changes should be discussed on the [mailing list](https://www.ietf.org/mailman/listinfo/oauth).

### Reading the spec

The final version of the spec can be found at [http://tools.ietf.org/html/rfc6749](http://tools.ietf.org/html/rfc6749).

### Implementations

#### Server Libraries

*   Java

    *   [Apache Oltu](http://oltu.apache.org/)
    *   [Spring Security for OAuth](http://static.springsource.org/spring-security/oauth/)
    *   [Apis Authorization Server (v2-31)](https://github.com/OpenConextApps/apis)
    *   [Restlet Framework (draft 30)](http://www.restlet.org/)
    *   [Apache CXF](http://cxf.apache.org/)

*   PHP

    *   [PHP OAuth2 Server](https://github.com/bshaffer/oauth2-server-php) and [Demo](https://github.com/bshaffer/oauth2-demo-php)
    *   [PHP OAuth 2.0 Auth and Resource Server](https://github.com/thephpleague/oauth2-server) and [Demo](https://github.com/lncd/oauth2-example-auth-server)
    *   [PHP OAuth 2.0](https://github.com/fkooman/php-oauth) (AS with SAML/BrowserID AuthN, with management REST API, see [DEMO](https://frko.surfnetlabs.nl/workshop/))
    *   [PHP OAuth2.0](https://github.com/authbucket/oauth2) for [Silex](http://silex.sensiolabs.org/) and [Demo](http://oauth2.authbucket.com/demo)
    *   [PHP OAuth2.0](https://github.com/authbucket/oauth2-bundle) for [Symfony](http://symfony.com/) and [Demo](http://oauth2-bundle.authbucket.com/demo)

*   Python

    *   [Python OAuth 2.0 Provider](https://github.com/StartTheShift/pyoauth2) (see [Tutorial](http://tech.shift.com/post/39516330935/implementing-a-python-oauth-2-0-provider-part-1))
    *   [OAuthLib](https://github.com/idan/oauthlib) (a generic implementation of the OAuth request-signing logic) is avaliable for [Django](https://github.com/evonove/django-oauth-toolkit) and [Flask](https://github.com/lepture/flask-oauthlib) web frameworks

*   [NodeJS OAuth 2.0 Provider](https://github.com/t1msh/node-oauth20-provider)
*   [Ruby OAuth2 Server (draft 18)](https://github.com/nov/rack-oauth2)
*   [.NET DotNetOpenAuth](http://www.dotnetopenauth.net/)
*   [Erlang Oauth2 Server framework](https://github.com/kivra/oauth2)
*   [Thinktecture IdentityServer](https://github.com/thinktecture/Thinktecture.IdentityServer.v3)

#### Client Libraries

*   [PHP](http://www.phpclasses.org/package/7700-PHP-Authorize-and-access-APIs-using-OAuth.html)
*   [PHP OAuth 2.0 client](https://github.com/fkooman/php-oauth-client)
*   [OAuth2/OpenID Connect Client Library for PHP/Zend Framework 2](https://github.com/ivan-novakov/php-openid-connect-client)
*   [Cocoa](http://github.com/leebyron/cocoa-oauth2)
*   [iPhone and iPad](http://github.com/lukeredpath/LROAuth2Client)
*   [iOS and Mac OS X (draft 10)](http://github.com/nxtbgthng/OAuth2Client)

*   Java

    *   [Apache Oltu](http://oltu.apache.org/)
    *   [Spring Social](http://www.springsource.org/spring-social)
    *   [Spring Security for OAuth](http://static.springsource.org/spring-security/oauth/)
    *   [Restlet Framework (draft 30)](http://www.restlet.org/)
    *   [scribe-java](https://github.com/fernandezpablo85/scribe-java)

*   Python

    *   [sanction](http://github.com/demianbrecht/sanction)
    *   [rauth](http://github.com/litl/rauth)

*   [Ruby Gem](http://github.com/intridea/oauth2)
*   [Ruby](http://github.com/aflatter/oauth2-ruby)
*   [Javascript](http://github.com/andreassolberg/jso)

*   .NET

    *   [OWIN Middleware](http://www.nuget.org/packages/Microsoft.Owin.Security.OAuth)
    *   [DotNetOpenAuth](http://www.dotnetopenauth.net/)
    *   [Spring Social for .NET](http://www.springframework.net/social/)

*   Qt/C++

    *   [O2 (supports OAuth 1.0a and 2.0](https://github.com/pipacs/o2)

*   Lua/Corona SDK

    *   [Corona/Lua OAuth 2.0 API](http://selz.co/1kxjJVl)

#### Services that support OAuth 2

*   [37signals (draft 5)](http://groups.google.com/group/37signals-api/browse_thread/thread/86b0da52134c1b7e)
*   [Box](http://developers.box.com/oauth/)
*   [Beeminder](http://beeminder.com/api)
*   [Campaign Monitor](http://www.campaignmonitor.com/api/getting-started/#authenticating_with_oauth)
*   [Clever](https://clever.com/)
*   [Dropbox](https://www.dropbox.com/developers/core/docs#oa2-authorize)
*   [Facebook's Graph API](http://developers.facebook.com/docs/authentication/) (see [sociallipstick.com/?p=239](http://www.sociallipstick.com/?p=239))
*   [Foursquare](https://developer.foursquare.com/overview/auth)
*   [Geoloqi](https://developers.geoloqi.com)
*   [GitHub](http://developer.github.com/v3/oauth/)
*   [Google](https://developers.google.com/accounts/docs/OAuth2)
*   [Meetup](http://www.meetup.com/meetup_api/auth/#oauth2)
*   [NationBuilder](http://nationbuilder.com/api_quickstart)
*   [Salesforce](http://www.salesforce.com/us/developer/docs/api_rest/Content/quickstart_oauth.htm)
*   [Citrix ShareFile](http://www.sharefile.com/)
*   [SoundCloud](http://developers.soundcloud.com/docs/api/reference)
*   [Do.com (draft 22)](https://do.com)
*   [Windows Live](http://msdn.microsoft.com/en-us/library/live/hh243647.aspx)

### Legacy

For more information on OAuth 1.0 and 1.0a, see the [old About page](/about).

**Edit This Site**

The source code to this site is [available on Github](https://github.com/aaronpk/oauth.net). 
Feel free to submit pull requests with changes!

