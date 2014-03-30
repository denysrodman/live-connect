#siftware/live-connect

A PHP package that consumes Microsoft's Live Connect REST API allowing OneDrive (SkyDrive) interaction and authentication.

Uses OAuth 2.0 authorization code grant flow as [documented here](http://msdn.microsoft.com/en-us/library/live/hh243647.aspx).

Most of the MS Live Connect examples use Javascript, [this](http://msdn.microsoft.com/en-us/library/live/hh243649.aspx) is the best resource I found for a general explanation of the Live Connect auth process on the server side.

##Install

Use Composer.

    cd && mkdir project-root && cd project-root

Create a file called composer.json, put this in it:

    {
        "require": {
            "siftware/live-connect": "dev-master"
        },
        "autoload": {
            "psr-4": {
                "Siftware\\": "src/Siftware"
            }
        }
    }

Install composer

    curl -s http://getcomposer.org/installer | php
    mv composer.phar composer

Grab the library

    ./composer install


##Usage

Create this file called `test/example.php`

```php
<?php

// -- Bootstrapping. Put me in an include file

/**
* Get these from https://account.live.com/developers/applications
*/
define("LIVE_CLIENT_ID", "<put yours here>");
define("LIVE_CLIENT_SECRET", "<put yours here>");

define ("LC_LOG_CHANNEL", "live-connect");

// this should write to your project root (from the test directory)
define ("LC_LOG_PATH", __DIR__ . "/../");
define ("LC_LOG_NAME", "live-connect.log");

//Useful debugging output, by default to a file (uses Monolog)
//Exceptions also go to the same place
define ("LC_DEBUG", true);

define("LIVE_CALLBACK_URL", "http://live-connect.dev/example.php");

require_once __DIR__ . '/../vendor/autoload.php';

use Siftware\LiveConnect;
use Siftware\LiveRequest;

$live = new LiveConnect(LIVE_CLIENT_ID, LIVE_CLIENT_SECRET, LIVE_CALLBACK_URL);

//See here for a full list of scopes and what they are used for:
//http://msdn.microsoft.com/en-us/library/live/hh243646.aspx
$live->setScopes("wl.offline_access,wl.signin,wl.basic,office.onenote_create");

// -- /end Bootstrapping

// This conditional is only needed when the request is for a new user.
// In production, catering for step 1 of the OAuth process (getting an auth code)
// could be all handled on the callback page and the rest of the time just use
// $this->getAuthToken() which handles auth/refresh for you

$authCode = (isset($_GET["code"]) ? $_GET["code"] : "");
if (!$live->authenticate($authCode))
{
    print "Unable to authenticate against Live Connect";
}
else
{
    print "<pre>";
    print_r(json_decode($live->getProfile()));
    print_r(json_decode($live->getContacts()));
    print "</pre>";
}
```

As per the inline comments, note how the auth code we get from step one is being checked for in the example. We have to look out for this only for the very first time we authenticate. So, in production it might be a better idea to have the aunthenticate conditional on a dedicated callback page and rely on the authentication check within `liveRequest()`.

##To Do

I built this so I could start interacting with the OneNote API. So for now the only thing compelete is Authentication.

It will be relatively trivial to implement quite a few of the content retreival endpoints now, 2 have been supplied.

*Testing* needed.

If you end up using this as a base and flesh some more of the content methods out feel free to submit a pull request.

##Author

I'm [Darren Beale](http://beale.rs) ([@bealers](http://twitter.com/bealers))

##Credit

 There are a couple of other OneDrive/LiveConnect classes that I have borrowed ideas from:

  - [Anuradha Jayathilaka's live-api-php-class](https://github.com/astroanu/live-api-php-class) which I couldn't get working
  - [Jonathan Lovatt's php-skydrive] (https://github.com/lovattj/php-skydrive) which I didn't try to get working but I did nick the TokenStore idea from

##Licence

 This package is released under the [MIT Licence](http://opensource.org/licenses/MIT)