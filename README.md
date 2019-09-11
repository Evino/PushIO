## PushIOManager for iOS

* [Integration Guide](https://docs.oracle.com/cloud/latest/marketingcs_gs/OMCFB/ios/)


## Release Notes

### 6.42.1 - Critical Xcode 11 Bug Fix

With the release of 6.42.1, we fixed a critical bug due to which App built with Xcode 11 and earlier version of PushIO SDKs(6.42.0 or below) were stopped getting push notifications on iOS 13 devices. The reason for this is that Xcode 11, breaks a common technique that apps and libraries like PushIO were using to generate push device token for the device. 

If you are using Xcode 11 to build the apps then you **must** update the Responsys PushIO SDK version to 6.42.1 or later, so your apps continue to work with APNS and Responsys Platform. 
 
 
### Upgrading SDK to 6.39

#### New API for In-App Messaging
With the release of 6.39, we have introduced a new API to fetch In-App Messages at app launch. Once you turn-on this feature in Responsys Interact, you should use the below API to enable this feature in the SDK. 

```objective-c
[[PushIOManager sharedInstance] setInAppMessageFetchEnabled:YES];
```
##### Recommendation
* **Redesigned In-app Backend**: For any new In-app implementation or upgrade to 6.39, we strongly recommend enabling the "Redesigned In-app Backend” feature both in the SDK and for your Account at Responsys.

#### Add WebKit.Framework
UIWebView is replaced with WKWebView in SDK 6.39.0. Please make sure to add WebKit.framework in your App target.


### 6.33.1 Using location API

#### Start location tracking

SDK starts tracking location when `startUpdatingLocationForPush:` is called.
**NOTE:** Application need to provide the correct values of `desiredLocationAccuracy` and `distanceFilter` before calling `startUpdatingLocationForPush:`. Otherwise it will result into more frequent registrations.

```objective-c
    [[PushIOManager sharedInstance] setDistanceFilter:500.0];
    [[PushIOManager sharedInstance] setDesiredLocationAccuracy:100.0];
    [[PushIOManager sharedInstance] startUpdatingLocationForPush];
```

#### Stop location tracking

SDK stops tracking location when `stopUpdatingLocationForPush:` is called.

```objective-c
   [[PushIOManager sharedInstance] stopUpdatingLocationForPush];
```

#### Application provides location

If application wants to track the location, it can pass the new location to SDK by calling `setLastLocation:`

```objective-c
    CLLocation *newLocation = nil;//Populate the corresponding values of new CLLocation instance
    [[PushIOManager sharedInstance] setLastLocation:newLocation];
```


### Upgrading PushIO SDK 6.29.2 to 6.32.0

* **Supported iOS versions**: PushIO SDK 6.32.0 and later will support iOS 8.0 and later.
*  **Supported XCode versions**: PushIO SDK 6.32.0 and later is supported by XCode 8.3.2 onwards.

**[1.]** Add the **SQLite** framework to your application target:
   
6.32.0 uses `sqlite` to store the events (that is, register, engagement, unregister) and sync later. Your mobile application needs to add the **`sqlite3.0.tbd`** framework in your application target.

**[2.]** Add the **UserNotifications** framework to your application target:
   
6.32.0 works with the `UserNotifications` framework APIs.  Your mobile application needs to add the **`UserNotifications`** framework.

**[3.]** Enable the **Push Notifications** capability:

Your mobile application needs to add the **Push Notifications** capability, which you can do from the **Capabilities** tab of your application settings.  ![alt text](https://raw.githubusercontent.com/pushio/PushIOManager_iOS/master/NotificationCapabilities.png "Application Capabilities")
   
Ensure that when you enable Push Notifications:

* Add the Push Notifications feature to your app id.
* Add the Push Notifications entitlement to your entitlements file

#### Code Changes
For PushIO SDK 6.32.0, make the following changes to your mobile application code:

**[1.]** Add **Debug** vs **Release** configuration in **`AppDelegate.m`**:

Mobile apps now control whether configuration is loaded from either **`pushio_config_debug.json`** or **`pushio_config.json`**.  
Applications must set the `configType` to let the SDK read the suggested configuration file:
    
```objective-c
    #ifdef DEBUG
        [PushIOManager sharedInstance].configType = PIOConfigTypeDebug; //load pushio_config_debug.json
    #else
        [PushIOManager sharedInstance].configType = PIOConfigTypeRelease; //load pushio_config.json
    #endif
```

**IMPORTANT:** If your mobile app does not set the `configType` value, the app will _not_ select a configuration file (`pushio_config_debug.json` or `pushio_config.json`). Without the configuration file, your app will be unable to communicate with the server, and critical functions will fail (register, fetch messages, track engagements, and the like).

**[2.]** Enable logging in **`AppDelegate.m`**:

With 6.32.0, **`setDebugLevel`** is discontinued.  To enable logging, use the **`enableLogging`** and **`setLogLevel`** methods.  Use **`disableLogging`** to disable logging.

```objective-c
    #ifdef DEBUG
        [[PushIOManager sharedInstance] enableLogging:YES];
        [[PushIOManager sharedInstance] setLogLevel:PIOLogLevelInfo]; //PIOLogLevelWarn or PIOLogLevelError
    #else
        [[PushIOManager sharedInstance] disableLogging];
    #endif
```

**[3.]** Implement **`UserNotifications`** framework support:
Implement the **`UNUserNotificationCenterDelegate`** method in your **`AppDelegate.h`** and **`AppDelegate.m`** files, as shown below:
  
```objective-c
    //In AppDelegate.h
    #import <UserNotifications/UserNotifications.h>
    @interface AppDelegate : UIResponder <UIApplicationDelegate,UNUserNotificationCenterDelegate>
    @end
```

In AppDelegate.m, set the userNotificationCenter delegate, preferably in didFinishLaunchingWithOptions:

```objective-c
    [UNUserNotificationCenter currentNotificationCenter].delegate= self;
```

Implement userNotificationCenter delegate methods in Appdelegate.m:    
    
```objective-c
    -(void) userNotificationCenter:(UNUserNotificationCenter *)center
        didReceiveNotificationResponse:(UNNotificationResponse *)response
                 withCompletionHandler:(void(^)())completionHandler{
        [[PushIOManager sharedInstance] userNotificationCenter:center didReceiveNotificationResponse:response withCompletionHandler:completionHandler];
    }

    -(void) userNotificationCenter:(UNUserNotificationCenter *)center
               willPresentNotification:(UNNotification *)notification
                 withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler{
        [[PushIOManager sharedInstance] userNotificationCenter:center willPresentNotification:notification withCompletionHandler:completionHandler];
    }
```

### New Registration methods

**[1.]** **Configure**:

First application need to configure with SDK as following:

```objective-c
    NSError *error = nil;
    [[PushIOManager sharedInstance] configureWithAPIKey:API_KEY accountToken:ACCOUNT_TOKEN error:&error];
    if(nil == error){
        NSLog(@"SDK Configured Successfully");
    }else{
        NSLog(@"Unable to configure SDK, reason: %@", error.description);
    }
```

**[2.]** **Register**:

Once configured successfully, application can register with SDK as following:

```objective-c
[[PushIOManager sharedInstance] registerForAllRemoteNotificationTypes:^(NSError *error, NSString *deviceToken) {
        if (nil == error) {
            NSError *regTrackError = nil;
            [[PushIOManager sharedInstance] registerApp:&regTrackError completionHandler:^(NSError *regAppError, NSString *response) {
                if (nil == regAppError) {
                    NSLog(@"Application registered successfully!");
                }else{
                    NSLog(@"Unable to register application, reason: %@", regAppError.description);
                }
            }];
            if (nil == regTrackError) {
                NSLog(@"Registration locally stored successfully.");
            }else{
                NSLog(@"Unable to store registration, reason: %@", regTrackError.description);
            }
        }
    }];
```

## Media Attachment

To integrate media notification follow the guide [Stand alone](http://docs.oracle.com/cloud/latest/marketingcs_gs/OMCFA/ios/media-attachments/) or [Integrated](http://docs.oracle.com/cloud/latest/marketingcs_gs/OMCFB/ios/media-attachments/) as applicable. In addition to that there are few extra steps needed to avoid specific errors:

* **Don't** add PIOMediaAttachmentExtension.framework into **app target**.
    - PIOMediaAttachmentExtension.framework should be added only in notification service extension target and not the application's target. Otherwise there will be installation issue in iOS version lower than iOS 10.
* **Don't** add PIOMediaAttachmentExtension.framework into **embedded binaries**.
    - PIOMediaAttachmentExtension.framework shouldn't be added as embedded binary. Otherwise there will be compile time error as: 'bundle format unrecognized, invalid, or unsuitable'


## Contact
* Support: [My Oracle Support] (http://support.oracle.com)

Copyright © 2019, Oracle Corporation and/or its affiliates. All rights reserved. Oracle and Java are registered trademarks of Oracle and/or its affiliates. Other names may be trademarks of their respective owners.
