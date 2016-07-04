# withings connector
## About Withings
[Withings](http://www2.withings.com/) is a company that designs and sells fitness and health trackers, monitors and analyzers.
The device's data and services are exposed to developers as REST APIs.
## Purpose of the scriptr.io connector for withings
The purpose of this connector is to simplify and streamline the way you access withings' APIs from scriptr.io, by providing you with a few native objects that you can directly integrate into your own scripts. 
This will hopefully allow you to create sophisticated health and fitness oriented applications. 
## Components
- withings/user.js: this is the main object to interact with. It provides access to the data that is generated by the devices of a given user (the one for who you are passing an access token)
- withings/notifications.js: manages subscriptions to any type of withings notifications. 
- withings/common.js: configuration file that contains parameters used by the notifications script.
- withings/mappings.js: configuration file used for internal purposes (constants and descriptions).
- withings/withingsClient.js: generic http client that handles the communication between scriptr.io and withings' APIs. Used by user and notifications.
- withings/api/handleEvent.js: default callback invoked by withings when sending notifications.
- withings/notificationHandlers/DefaultHandler.js: this is the default handler that is triggered upon the occurrence of any
withings event. You can define your own handlers and map them to event using the "withings/common" script. 
- withings/authorization/getRequestTokenUrl.js: This script implements steps 1 and 2 of the Withings OAuth authorization process.
- withings/authorization/getAccessToken.js: This script implements step 3 of the Withings OAuth authorization process.
- withings/test/tests.js: a list of examples on how to use the connectors' objects and corresponding methods.
- withings/lib: this folder contains utility scripts that are used by the OAuth authorization process.

## How to use
- Use the Import Modules feature to deploy the aforementioned scripts in your scriptr account, in a folder named "modules/withings".
- Create a developer account and an application at [withings](http://www2.withings.com/us/en/developers).
- Once done, make sure to copy/paste the values of your API key and API secret in the corresponding
variables of the "config" file (respectively client_id and client_secret).
- Create an end user account (https://account.withings.com/)  
- Create a test script in scriptr, or use the script provided in modules/withings/test/. 

### Obtain access tokens from withings

#### Step 1
From a front-end application, send a request to the ```modules/withings/authorization/getRequestTokenUrl``` script.
The result returned by the aforementioned script should resemble the following:

```
>> curl -X POST  -H 'Authorization: bearer <YOUR_AUTH_TOKEN>' 'https://api.scriptrapps.io/modules/withings/authorization/getRequestTokenUrl.js'

{
	"metadata": {
		"requestId": "f88395ff-c150-4e6b-ac7a-d771b698ceb1",
		"status": "success",
		"statusCode": "200"
	},
	"result": "https://oauth.withings.com/account/authorize?oauth_consumer_key=231cef543c960eld70bck08p8b4e3081dc407e8984eb914e46e93f649c77ce&oauth_nonce=164c1554f4nc6oe7533691c4510a939b&oauth_signature=QyXJoKQaOJwXaSZ9lvWyhTa1LA0%3D&oauth_signature_method=HMAC-SHA1&oauth_timestamp=1440595329&oauth_token=64094794f2c6cd2k8e475b25f647010503f9d9d598789e62223355c2b28&oauth_version=1.0"
}
```
#### Step 2

From the front-end, issue a request to the obtained URL. This redirects your end user to the withings login and authorization page, 
where he has to enter his credentials then authorize the application on the requested scope. 
Once this is done, withings automatically calls back the ```modules/withings/authorization/getAccessToken.js``` script, providing it with access and secret tokens.
 These ared stored in your scriptr.io's global storage.

### Use the connector

In order to use the connector, you need to import the main module: ```modules/withings/user.js```, as described below:
```
var userModule = require("/modules/withings/user.js");
```
Then create a new instance of the User class, defined in this module (we assume that we already otbained an access token for the given user):
```
var user = new userModule.User({userid:"123456"});
```
The User class provides many methods to obtain data related to devices pertaining to the end user, such as:
```

// list all user's activities starting from the given date
user.listActivities({from: "2015-07-21"}); 

// list the user's sleep measures for the given time framne
user.listSleepMeasures({from: new Date("2015-07-21"), to:new Date("2015-08-24")});

// subscribe to notifications 
var common = require("/modules/withings/common.js");
var sub = {
  "notificationType": common.notificationTypes.WEIGHT
};

var notificationManager = user.getNotificationManager();
notificationManager.subscribeToNotification(sub); 
```

*You can check the list of all available methods using the ```modules/withings/test/tests.js``` script.*

### About notifications
As mentioned, you can subscribe to notifications that are emitted by a given device for the concerned user.
By default, withings is instructed by the connector to send notifications to the https://api.scriptrapps.io/modules/withings/api/handleNotifications.js API 
that resides in your account. This API's job is to trigger the handler that is configured to process the corresponding notification type.
By default, the "modules/withings/notifications/DefaultHandler.js" is invoked for all the notification types. You can implement your own handlers 
and configure what handler to use for what notification type in the "modules/withings/common.js" script.
