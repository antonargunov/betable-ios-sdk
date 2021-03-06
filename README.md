# Changelog

If the SDK you downloaded does not have a versioning number, assume it is pre 0.8.0.

## Current

* Now requires `AdSupport.framework`
* Support for iOS7
* More stable pre-caching of authorize page
* Some support for promotions
* Now using a track enpoint for better tracking of users, and tracks users through ad installs.

## 0.9.0

* Added hooks for Betable Testing Profiles. Which allows you to test your games against new features and flows, without changing the code at all (Coming Soon)

## 0.8.0

* Now supports batched requests
* Uses in app web view for authorization instead of bouncing to Safari
* Now supports unbacked bets and credit bets.
* added logout method, which properly clears cookies and access tokens when a user logs out.

# Betable iOS SDK

## Adding the Framework

In the `framework` directory of this repository is a folder called `Betable.framework` and one called `Betable.bundle`.  Download these directories and then drag and drop them into your project (Usually into the frameworks group).  Then just `#import <Betable/Betable.h>` in whichever files you reference the Betable object form.  (This is the method that the [`betable-ios-sample`](https://github.com/betable/betable-ios-sample) app uses.)

To use this framework you are required to include the following iOS frameworks: `Foundation.framework`, `UIKit.framework`, and `AdSupport.framework`.

If you want to modify the code and build a new framework, simply build the Framework target in this project and the folders will be built to the proper build locations (usually `~/Library/Developer/Xcode/DerivedData/Betable.framework-<hash>/Build/Products/Debug-iphoneos/`).  Simply drag those files into your project from there or you can link to them so you can continue development on both the framework and your project at the same time.

## `Betable` Object

This is the object that serves as a wrapper for all calls to the Betable API.

### Initializing

    - (Betable*)initWithClientID:(NSString*)clientID
                    clientSecret:(NSString*)clientSecret
                     redirectURI:(NSString*)redirectURI;

To create a `Betable` object simply initilize it with your client ID, client secret and redirect URI.  All of these can be set at <https://developers.betable.com> when you create your game.  Your redirect URI needs to have a custom unique scheme and the domain needs to be authorize. An example is betable+<company_name>+<game_name>://authorize .  It is important that it is unique so the oauth flow can be completed.  See **Authorization** below for more details.

### Adding the Token

<pre><code>self.accessToken = <em>accessToken</em></code></pre>

If you have previously acquired an access token for the user you can simply set it after the initialization, skipping the authorization and access token acquisition steps, and start making requests to the Betable API.

### Authorization

    - (void)authorizeInViewController:(UIViewController*)viewController
                          onAuthorize:(BetableAccessTokenHandler)onAuthorize
                            onFailure:(BetableFailureHandler)onFailure
                             onCancel:(BetableCancelHandler)onCancel;

This method should be called when no access token exists for the current user.  It will initiate the OAuth protocol.  It will open a UIWebView in portrait and direct it to the Betable signup/login page.  After the person authorizes your app at <https://betable.com>, Betable will redirect them to your redirect URI which can be registered at <https://developers.betable.com> after configuring your game. This will be handled by the `Betable` object's `handleAuthroizeURL:` method inside of your applicaiton delegate's `application:handleURLOpen:`.

The redirect URI should have a protocol that opens your app.  See [Apple's documentation](http://developer.apple.com/library/ios/#documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/AdvancedAppTricks/AdvancedAppTricks.html#//apple_ref/doc/uid/TP40007072-CH7-SW50) for details.  It is suggested that your URL scheme be <code>betable+<em>game_id</em></code> and that your redirect URI be <code>betable+<em>game_id</em>://authorize</code>.  After the user has authorized your app, the authroize view will invoke your app'a `application:handleOpenURL:` in your `UIApplicationDelegate`.  Inside that method you need to call the `Betable` objects `handleAuthorizeURL:`.

There are 3 handlers to pass in to this call: `onAuthroize`, `onFailure`, and `onCancel`. onAuthorize and onFailure can be set at anytime on the betable object between when this call is made and when the response is handled inside of `application:handleURLOpen:`

#### onAuthorize(NSString *accessToken)

This is called when the person successfully completes the authorize flow. It gets passed the accessToken. You should store this accessToken with your user so that subsequent launches do not require reauthorization.

#### onFailure(NSURLResponse *response, NSString *responseBody, NSError *error)

This is called when the server rejects the authorization attempt by a user. `error` will have more information on why it was rejected.

#### onCancel()

This is called when the person cancels out of the authorization at some point during the authroization flow.

### Getting the Access Token

    - (void)handleAuthorizeURL:(NSURL*)url

Once your app receives the redirect uri in `application:handleOpenURL:` of your `UIApplicationDelegate` you can pass the uri to the `handleAuthorizeURL:` method of your `Betable` object.

This is the final step in the OAuth protocol.  In the `onComplete` handler that you passed into the `authorizeInViewController:onAuthorizationComplete:onFailure:onCancel:` you will recieve your access token for the user associated with this `Betable` object.  You will want to store this with the user so you can make future requests on their behalf.

### Loggging out

    - (void)logout

If you need to disassociate the current player with the betable object simply call the logout method.  This handles destroying the cookies, resetting the authorize web browser, and removing the betable token.

### Betting

    - (void)betForGame:(NSString*)gameID
              withData:(NSDictionary*)data
            onComplete:(BetableCompletionHandler)onComplete
             onFailure:(BetableFailureHandler)onFailure;

This method is used to place a bet for the user associated with this Betable object.

* `gameID`: this is your gameID which is registered and can be checked at <https://developers.betable.com>
* `data`: this is a dictionary that will converted to JSON and sent as the request body.  It contains all the important information about the bet being made.  For documentation on the format of this dictionary see <https://developers.betable.com/docs#api-documentation>.
* `onComplete`: this is a block that will be called in case of success.  Its only argument is a dictionary that contains Betable's JSON response.
* `onFailure`: This is a block that will be called in case of error.  Its arguments are the `NSURLResponse` object, the string reresentation of the body, and the `NSError` that was raised.

####Unbacked Betting
If you want to make a bet that is not backed by the accounting software but just uses our betting math, you can make an unbacked bet. 
    - (void)unbackedBetForGame:(NSString*)gameID
                      withData:(NSDictionary*)data
                    onComplete:(BetableCompletionHandler)onComplete
                     onFailure:(BetableFailureHandler)onFailure;

If you would like to do this with an unauthorized user you can an access token that only has unbacked betting permission from the following method:

    - (void)unbackedToken:(NSString*)clientUserID
               onComplete:(BetableAccessTokenHandler)onComplete
                onFailure:(BetableFailureHandler)onFailure;

#### Credit Betting

If you want to make a credit bet, backed and unbacked, use these two methods respectively:

    - (void)creditBetForGame:(NSString*)gameID
                  creditGame:(NSString*)creditGameID
                    withData:(NSDictionary*)data
                  onComplete:(BetableCompletionHandler)onComplete
                   onFailure:(BetableFailureHandler)onFailure;

    - (void)unbackedCreditBetForGame:(NSString*)gameID
                          creditGame:(NSString*)creditGameID
                            withData:(NSDictionary*)data
                          onComplete:(BetableCompletionHandler)onComplete
                           onFailure:(BetableFailureHandler)onFailure;

In both of these methods `creditGameID` is the ID of the game for which you would like to make a bet.  `gameID` is the game your user authed with and the game in which they won the credits.

### Batching Bet Requests

You can batch requests to the api server by using the [`Betable request batching endpoint`](https://developers.betable.com/docs/#batch-requests).

The SDK supports this with an object called `BetableBatchRequest`. You simply initialize it with a Betable object and you can add requests to it. Once all the requests you wish to batch have been added you can fire the requests and wait for the batch response. There are two ways of creating these requests, you can create them manually and add them, or you can use the prebuilt convenience methods.

####Manually Creating Requests

You can create your own requests using the following.

    - (NSMutableDictionary* )createRequestWithPath:(NSString*)path
                                            method:(NSString*)method
                                              name:(NSString*)name
                                      dependencies:(NSArray*)dependnecies
                                              data:(NSDictionary*)data;

And then add it to the requests for that batch with the following.

	- (void)addRequest:(NSDictionary*)request;

####Using the convenience request methods

You can use the betting and unbacked betting methods which automatically create and add the proper requests.

    - (NSMutableDictionary* )betForGame:(NSString*)gameID
                               withData:(NSDictionary*)data
                              withName: (NSString*)name;

    - (NSMutableDictionary* )unbackedBetForGame:(NSString*)gameID
                                       withData:(NSDictionary*)data
                                       withName: (NSString*)name;

    - (NSMutableDictionary* )creditBetForGame:(NSString*)gameID
                                   creditGame:(NSString*)creditGameID
                                     withData:(NSDictionary*)data
                                     withName:(NSString*)name;

    - (NSMutableDictionary* )unbackedCreditBetForGame:(NSString*)gameID
                                           creditGame:(NSString*)creditGameID
                                             withData:(NSDictionary*)data
                                             withName: (NSString*)name;

####Issuing the batched requests

Once you have added all the requests you want to the batch, simply fire the batch request.

    - (void)runBatchOnComplete:(BetableCompletionHandler)onComplete 
                     onFailure:(BetableFailureHandler)onFailure;

The `BetableCompletionHandler` will receive a `NSDictionary` that will represent the documented JSON response found in the [Betable batch request api](https://developers.betable.com/docs/#response-protocol).

### Getting User's Account

    - (void)userAccountOnComplete:(BetableCompletionHandler)onComplete
                        onFailure:(BetableFailureHandler)onFailure;

This method is used to retrieve information about the account of the user associated with this `Betable` object.

* `onComplete`: this is a block that will be called in case of success.  Its only argument is a dictionary that contains Betable's JSON response.
* `onFailure`: this is a block that will be called in case of error.  Its arguments are the `NSURLResponse` object, the string reresentation of the body, and the `NSError` that was raised.

### Getting User's Wallet

    - (void)userWalletOnComplete:(BetableCompletionHandler)onComplete
                       onFailure:(BetableFailureHandler)onFailure;

This method is used to retrieve information about the wallet of the user associated with this betable object.


* `onComplete`: this is a block that will be called in case of success.  Its only argument is a dictionary that contains Betable's JSON response.
* `onFailure`: this is a block that will be called in case of error.  Its arguments are the `NSURLResponse` object, the string reresentation of the body, and the `NSError` that was raised.

### Completion and Failure Handlers

#### BetableAccessTokenHandler:

    typedef void (^BetableAccessTokenHandler)(NSString *accessToken);

This is called when `token:onCompletion:onFailure` successfully retrieves the access token.

#### BetableCompletionHandler:

    typedef void (^BetableCompletionHandler)(NSDictionary *data);

This is called when any of the APIs successfully return from the server.  `data` is a nested NSDictionary object that represents the JSON response.

#### BetableFailureHandler:

    typedef void (^BetableFailureHandler)(NSURLResponse *response, NSString *responseBody, NSError *error);

This is called when something goes wrong during the request.  `error` will have details about the nature of the error and `responseBody` will be a string representation of the body of the response.

### Accessing the API URLs

* `(NSString*)getTokenURL;`
* `(NSString*)getBetURL:(NSString*)gameID;`
* `(NSString*)getWalletURL;`
* `(NSString*)getAccountURL;`
