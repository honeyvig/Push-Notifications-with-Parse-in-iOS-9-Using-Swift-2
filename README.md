# Push-Notifications-with-Parse-in-iOS-9-Using-Swift-2
To implement push notifications using Parse in an iOS app (with iOS 9 and Swift 2), follow these steps. Since Parse has deprecated its push notification services after January 2017, you would need to use a self-hosted Parse Server or consider using Back4App, a platform that offers hosted Parse services. However, I will proceed with the steps assuming that you still want to use the Parse SDK for push notifications.
Steps to Set Up Push Notifications with Parse in iOS 9 Using Swift 2

    Set Up Parse Account and Parse SDK
        If you haven't already, create an account on Parse or set up a self-hosted Parse server or use Back4App.
        Install the Parse SDK using CocoaPods.
        Ensure your project has access to Push Notification Services by enabling them in your app's capabilities.

    Set Up Push Notifications in Apple Developer Portal
        Go to Apple Developer Portal and create an App ID for your app.
        Enable Push Notifications in your App ID configuration.
        Create an APNs Key for push notifications and download it.
        Upload this key to Parse or your Parse Server/Back4App.

    CocoaPods for Parse SDK
        Ensure that you have CocoaPods installed, and then add the Parse SDK to your Podfile:

    platform :ios, '9.0'

    pod 'Parse', '~> 1.17.4'
    pod 'ParsePush', '~> 1.17.4'

    Integrate Parse SDK and Push Notifications in Your App

Swift Code for Push Notifications in iOS 9 with Parse SDK (Swift 2)
1. Set Up Parse SDK in AppDelegate.swift

import UIKit
import Parse
import ParsePush

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    // Replace with your Parse App ID and Client Key
    let parseAppID = "YOUR_PARSE_APP_ID"
    let parseClientKey = "YOUR_PARSE_CLIENT_KEY"
    let parseServerURL = "https://your-parse-server-url.com/parse"

    // Called when the app has launched
    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        
        // Initialize Parse
        Parse.setApplicationId(parseAppID, clientKey: parseClientKey)
        let configuration = ParseClientConfiguration {
            $0.isLocalDatastoreEnabled = true
            $0.server = parseServerURL
        }
        Parse.initializeWithConfiguration(configuration)
        
        // Register for Push Notifications
        application.registerUserNotificationSettings(UIUserNotificationSettings(forTypes: [.Alert, .Badge, .Sound], categories: nil))
        application.registerForRemoteNotifications()

        return true
    }

    // Handle the reception of push notification
    func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        // This is where you handle the received push notification
        // Example: Use ParsePush to handle the notification
        PFPush.handlePush(userInfo)

        completionHandler(.newData)
    }

    // When the device successfully receives the push token
    func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
        // Parse requires converting the device token to a string and registering it
        let installation = PFInstallation.currentInstallation()
        installation.setDeviceTokenFromData(deviceToken)
        installation.saveInBackgroundWithBlock { (succeeded: Bool, error: NSError?) in
            if succeeded {
                print("Device token saved successfully.")
            } else {
                print("Failed to save device token: \(error?.localizedDescription ?? "Unknown error")")
            }
        }
    }

    // Handle failure to register for remote notifications
    func application(application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: NSError) {
        print("Failed to register for push notifications: \(error.localizedDescription)")
    }
}

2. Request Permissions for Notifications

Ensure that you are requesting permission from the user to show push notifications. This is already handled above in the didFinishLaunchingWithOptions method, where you call registerUserNotificationSettings.
3. Sending Push Notifications (Server-Side or Parse API)

Once the user has registered for notifications and has a device token, you can send notifications either from your server or using the Parse dashboard.

For instance, if you want to send a push notification to a user from your backend (Parse Server or Back4App), you can use the Parse REST API to trigger the push notification. Below is an example of sending a push notification from the server-side using the Parse REST API.
Example of Sending a Push Notification from Parse API (Node.js Server)

const axios = require('axios');

async function sendPushNotification() {
  const parseServerURL = 'https://your-parse-server-url.com/parse';
  const headers = {
    'X-Parse-Application-Id': 'YOUR_PARSE_APP_ID',
    'X-Parse-REST-API-Key': 'YOUR_PARSE_REST_API_KEY',
    'Content-Type': 'application/json'
  };

  const data = {
    "where": {
      "deviceType": "ios"
    },
    "data": {
      "alert": "Hello! You have a new notification."
    }
  };

  try {
    const response = await axios.post(`${parseServerURL}/push`, data, { headers });
    console.log("Push notification sent successfully:", response.data);
  } catch (error) {
    console.error("Error sending push notification:", error);
  }
}

// Call the function to send a push notification
sendPushNotification();

4. Test the Push Notification

Once the setup is complete, you can run your app on a real device and send a test push notification. The notification will be received by the app if it is properly registered.
Key Points:

    Registering for Push Notifications: You need to register for push notifications in the AppDelegate and handle the device token to save it in the Parse server.
    Handling Incoming Notifications: Implement the didReceiveRemoteNotification method to handle the notifications when the app is running in the background or foreground.
    Sending Notifications: You can send push notifications from your server using the Parse REST API, or manually from the Back4App dashboard.

Conclusion:

With this setup, you should be able to implement push notifications in your iOS 9 app using Parse. This code allows for receiving and handling push notifications in the app, and you can send push notifications from the backend via the Parse API.
