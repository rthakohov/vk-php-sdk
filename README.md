# vk-php-sdk

PHP library for VK API interaction, includes OAuth 2.0 authorization and API methods. Full VK API features documentation can be found [here](http://vk.com/dev).

This library has been created using the VK API JSON Schema. It can be found [here](https://github.com/VKCOM/vk-api-schema). It uses VK API [version](https://vk.com/dev/versions) 5.69.

## 1. Prerequisites

* PHP 7.1 or later

## 2. Installation

The VK PHP SDK can be installed using Composer by running the following command:

```sh
composer require vkcom/vk-php-sdk
```

## 3. Initialization

Create VKApiClient object using the following code:

```php
$vk = new VKApiClient();
```

Also you can initialize `VKApiClient` with different API version and different language like this:

```php
$vk = new VKApiClient('5.69');
```

```php
$vk = new VKApiClient('5.69', VKLanguage::ENGLISH);
```

## 4. Authorization

The library provides the authorization flows for user based on OAuth 2.0 protocol implementation in vk.com API. Please read the full [documentation](https://vk.com/dev/access_token) before you start.

### 4.1. Authorization Code Flow

OAuth 2.0 Authorization Code Flow allows calling methods from the server side.

This flow includes two steps — obtaining an authorization code and exchanging the code for an access token. Primarily you should obtain the "code" ([manual user access](https://vk.com/dev/authcode_flow_user) and [manual community access](https://vk.com/dev/authcode_flow_group)) by redirecting the user to the authorization page using the following method:

Create `VKOAuth` object first:
```php
$oauth = new VKOAuth();
```

#### 4.1.1. For getting **user access key** use following command:
```php
$oauth = new VKOAuth();
$client_id = 1234567;
$redurect_uri = 'https://example.com/vk';
$display = VKOAuthDisplay::PAGE;
$scope = array(VKOAuthUserScope::WALL, VKOAuthUserScope::GROUPS);
$state = 'secret_state_code';

$browser_url = $oauth->authorize(OAuthResponseType::CODE, $client_id, $redurect_uri, $display, $scope, $state);
```
#### 4.1.2. Or if you want to get **community access key** use:
```php
$oauth = new VKOAuth();
$client_id = 1234567;
$redurect_uri = 'https://example.com/vk';
$display = VKOAuthDisplay::PAGE;
$scope = array(VKOAuthGroupScope::MESSAGES);
$state = 'secret_state_code';
$groups_ids = array(1, 2);

$browser_url = $oauth->authorize(VKOAuthResponseType::CODE, $client_id, $redurect_uri, $display, $scope, $state, $groups_ids);
```

[User access key](https://vk.com/dev/permissions?f=1.%20%D0%9F%D1%80%D0%B0%D0%B2%D0%B0%20%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%B0%20%D0%B4%D0%BB%D1%8F%20%D1%82%D0%BE%D0%BA%D0%B5%D0%BD%D0%B0%20%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8F) and [community access key](https://vk.com/dev/permissions?f=2.%20%D0%9F%D1%80%D0%B0%D0%B2%D0%B0%20%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%B0%20%D0%B4%D0%BB%D1%8F%20%D1%82%D0%BE%D0%BA%D0%B5%D0%BD%D0%B0%20%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D1%81%D1%82%D0%B2%D0%B0) uses different values inside scope array

After successful authorization user's browser will be redirected to the specified **redirect_uri**. Meanwhile the code will be sent as a GET parameter to the specified address:

```sh
https://example.com?code=CODE
```

Then use this method to get the access token:

```php
$oauth = new VKOAuth();
$client_id = 1234567;
$client_secret = 'SDAScasd'
$redurect_uri = 'https://example.com/vk';
$code = 'CODE';

$access_token = $oauth->getAccessToken($client_id, $client_secret, $redurect_uri, $code);
```

The **redirect_uri** should be the URL that was used to get a code at the first step.

### 4.2. Implicit flow

In difference with authorization code flow this flow gives you temporary access key.

Read more about [user access key](https://vk.com/dev/implicit_flow_user) and [community access key](https://vk.com/dev/implicit_flow_group).

First step to get access using Implicit flow is creating `VKOauth` object:
```php
$oauth = new VKOAuth();
```

#### 4.2.1. For getting **user access key** use following command:
```php
$oauth = new VKOAuth();
$client_id = 1234567;
$redurect_uri = 'https://example.com/vk';
$display = VKOAuthDisplay::PAGE;
$scope = array(VKOAuthUserScope::WALL, VKOAuthUserScope::GROUPS);
$state = 'secret_state_code';
$revoke_auth = true;

$browser_url = $oauth->authorize(VKOAuthResponseType::TOKEN, $client_id, $redurect_uri, $display, $scope, $state, $revoke_auth);
```

If you want to make user getting access anyway, set **revoke_auth** as true.

#### 4.2.2. Or if you want to get **community access key** use:
```php
$oauth = new VKOAuth();
$client_id = 1234567;
$redurect_uri = 'https://example.com/vk';
$display = VKOAuthDisplay::PAGE;
$scope = array(VKOAuthGroupScope::MESSAGES);
$state = 'secret_state_code';
$groups_ids = array(1, 2);

$browser_url = $oauth->authorize(VKOAuthResponseType::TOKEN, $client_id, $redurect_uri, $display, $scope, $state, $groups_ids);
```

Arguments are similar with authorization code flow

After successful authorization user's browser will be redirected to the specified **redirect_uri**. Meanwhile the access token will be sent as a GET parameter to the specified address:

For **user access key** will be:
```sh
https://example.com#access_token=533bacf01e11f55b536a565b57531ad114461ae8736d6506a3&expires_in=86400&user_id=8492&state=123456
```
And for **community access key**:
```sh
https://example.com#access_token_XXXXXX=533bacf01e11f55b536a565b57531ad114461ae8736d6506a3&expires_in=86400
```

**access_token** is your new access token.  
**expires_in** is lifetime of access token in seconds.  
**user_id** is user identifier.  
**state** is string from `authorize` method.  
access_token_**XXXXXX** is community access token where XXXXXX is community identifier.

## 5. API Requests
 
You can find the full list of VK API methods [here](https://vk.com/dev/methods).
 
### 5.1. Request sample
 
Example of calling method **users.get**:
 
```php
$vk = new VKApiClient();
$response = $vk->users()->get($access_token, array(
    'user_ids'  => array(1, 210700286),
    'fields'    => $array('city', 'photo'),
));
```
 
### 5.2. Uploading Photos into a Private Message
 
Please read [the full manual](https://vk.com/dev/upload_files?f=4.%20Uploading%20Photos%20into%20a%20Private%20Message) before the start.
 
Call **photos.getMessagesUploadServer** to receive an upload address:
 
```php
$vk = new VKApiClient();
$address = $vk->photos()->getMessagesUploadServer('{access_token}');
```

Then use **upload()** method to send files to the **upload_url** address received in the previous step:

```php
$vk = new VKApiClient();
$photo = $vk->request()->upload($address['upload_url'], 'photo', 'photo.jpg');
```

You will get a JSON object with **server**, **photo**, **hash** fields. To save a photo call **photos.saveMessagesPhoto** with these three parameters:

```php
$vk = new VKApiClient();
$response_save_photo = $vk->photos()->saveMessagesPhoto($access_token, array(
    'server' => $photo['server'],
    'photo'  => $photo['photo'],
    'hash'   => $photo['hash'],
));
```

Then you can use **owner_id** and **id** parameters from the last response to create an attachment of the uploaded photo. 

### 5.3. Uploading Video Files
 
Please read [the full manual](https://vk.com/dev/upload_files_2?f=9.%20Uploading%20Video%20Files) before the start.
 
Call **video.save** to get a video upload server address:

```php
$vk = new VKApiClient();
$address = $vk->video()->save($access_token, array(
    'name' => 'My video',
));
```

Send a file to **upload_url** received previously calling **upload()** method:

```php
$vk = new VKApiClient();
$video = $vk->request()->upload($address['upload_url'], 'video_file', 'video.mp4');
```

Videos are processed for some time after uploading.

## 6. Groups updates
 
### 6.1. Long Poll

Enable Callback API Long Poll for your group and specify which events should be tracked by calling the following API method:

```php
$vk = new VKApiClient();
$vk->groups()->setLongPollSettings($access_token, array(
  'group_id'      => 159895463,
  'enabled'       => 1,
  'message_new'   => 1,
  'wall_post_new' => 1,
));
```

Override methods from VKCallbackApiHandler class for handling events:

```php
class CallbackApiMyHandler extends VKCallbackApiHandler {
    public function messageNew($object) {
        echo 'New message: ' . $object['body'];
    }
    
    public function wallPostNew($object) {
        echo 'New wall post: ' . $object['text'];
    }
}
```

To start listening to LongPoll events, create an instance of your CallbackApiMyHandler class, instance of VKCallbackApiLongPollExecutor class and call method listen():

```php
$vk = new VKApiClient();
$access_token = 'asdj4iht2i4ntokqngoiqn3ripogqr';
$group_id = 159895463;
$wait = 25;

$handler = new CallbackApiMyHandler();
$executor = new VKCallbackApiLongPollExecutor($vk, $access_token, $group_id, $handler, $wait);
$executor->listen();
```

Parameter **wait** is the waiting period.

While calling function **listen()** you can also specify the number of the event from which you want to receive data. The default value is the number of the last event.

Example:

```php
$vk = new VKApiClient();
$access_token = 'asdj4iht2i4ntokqngoiqn3ripogqr';
$group_id = 159895463;
$timestamp = 12;
$wait = 25;

$executor = new VKCallbackApiLongPollExecutor($vk, $access_token, $group_id, $handler, $wait);
$executor->listen($timestamp);
```

### 6.2. Callback API

CallbackApi handler will wait until VK send notification about event when it happened you may handle this event. More information [here](https://vk.com/dev/callback_api).

You should to configure Callback API inside your community settings. 

First step will be approve your domain. VK sends you request to your server with event type **confirmation** and you should to send back confirmation string. In other types of event you should to send back `ok` string.

Look at this example:
```php
use VK\CallbackApi\Server\VKCallbackApiServerHandler;

class ServerHandler extends VKCallbackApiServerHandler {
    const SECRET = 'ab12aba';
    const GROUP_ID = 123999;
    const CONFIRMATION_TOKEN = 'e67anm1';

    function confirmation(int $group_id, ?string $secret) {
        if ($secret === static::MY_SECRET && $group_id === static::GROUP_ID) {
            echo static::CONFIRMATION_TOKEN;
        }
    }
    
    public function messageNew(int $group_id, ?string $secret, array $object) {
        echo 'ok';
    }
}

$handler = new ServerHandler();
$data = json_decode(file_get_contents('php://input'));
$handler->parse($data);
```

To handle events you should to override methods from VKCallbackApiServerHandler class like this. 

`confirmation` event handler contains 2 arguments: group id, and secret key. You must to override confirmation method.

