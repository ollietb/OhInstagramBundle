OhInstagramBundle
=================

Uses [Instaphp](https://github.com/sesser/Instaphp) by [sesser](https://github.com/sesser) as a Symfony2 service

Installation
------------

Install this bundle as usual by adding to composer.json:

    "oh/instagram-bundle": "dev-master"

Register the bundle in `app/AppKernel.php`:

    // app/AppKernel.php
    public function registerBundles()
    {
        return array(
            // ...
            new Oh\InstagramBundle\OhInstagramBundle(),
        );
    }

Add the OhInstagramBundle configuration to the imports section of `app/config/config.yml`:

	imports:
		- { resource: parameters.yml }
		- { resource: @OhInstagramBundle/Resources/config/services.yml }

Add your Instagram API client id/secret parameters to `app/config/parameters.yml`:
	
	parameters:
		instagram_api_client_id: [Client ID]
    	instagram_api_client_secret: [Client 

And if you're OK with the provided routes, add these to `app/config/routing.yml`:   	

    OhInstagramBundle:
        resource: "@OhInstagramBundle/Resources/config/routing.yml"
        prefix:   /

Usage (Controller)
------------

    //finding a user
    $query = "snoopdogg";

    $api = $this->get('instaphp');
		
    $response = $api->Users->Find($query);

    $userInfo = $response->data[0];

    // getting all the pages of results
    do {
        $pages[] = $response->data;
    while($response = $response->getNextPage());

You can also test if a user is logged in.

    //is a user logged in?
    $loggedIn = $this->get('instaphp_token_handler')->isLoggedIn();

Usage (Twig)
------------

Theres a login button included in the views. Just implement this line in your
Twig template

    {% render "OhInstagramBundle:Instagram:instagramLoginStatus" %}

You should set up your Instagram API account to callback to the
"OhInstagramBundle_callback" route, which you can set yourself, or use the ones
provided - "/instaphp/callback".

Instagram Auth Token
-----------

There are 2 TokenHandlers included.

1.  CookieToken - The Instagram auth code is stored in a cookie

        services:
            instaphp_token_handler:
                class:            Oh\InstagramBundle\TokenHandler\CookieToken
 
2.  UserToken - The Instagram auth code is stored in the User Entity. The methods 
setInstagramAuthCode() and getInstagramAuthCode() must be implemented on your 
User. When the login call is returned from Instagram, the code is set and the 
user is persisted and flushed in the Handler.

        services:
            instaphp_token_handler:
                class:            Oh\InstagramBundle\TokenHandler\UserToken
                arguments:        [@security.context, @doctrine.orm.default_entity_manager]

3.  Both - This will look to see if the user can be retrieved from the context
and if it can't it will store the auth code in a cookie.

        services:
            instaphp_user_token_handler:
                class:            Oh\InstagramBundle\TokenHandler\UserToken
                arguments:        [@security.context, @doctrine.orm.default_entity_manager]
            instaphp_cookie_token_handler:
                class:            Oh\InstagramBundle\TokenHandler\CookieToken
            instaphp_token_handler:
                class:            Oh\InstagramBundle\TokenHandler\TokenManager
                arguments:        [@instaphp_user_token_handler, @instaphp_cookie_token_handler]

You can also implement your own TokenHandlerInterface to store the auth code
somewhere else, like the session etc.

Tests
-------

@todo

Credits
-------

* Ollie Harridge (ollietb) as the author.
* Randy (sesser) for writing the Instaphp script [https://github.com/sesser/Instaphp]
