# Perfect-Notifications [简体中文](README.zh_CN.md)

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="http://stackoverflow.com/questions/tagged/perfect" target="_blank">
        <img src="http://www.perfect.org/github/perfect_gh_button_2_SO.jpg" alt="Stack Overflow" />
    </a>  
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>  
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat" alt="Swift 3.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>

APNs remote Notifications for Perfect. This package adds push notification support to your server. Send notifications to iOS/macOS devices.


## Issues

We are transitioning to using JIRA for all bugs and support related issues, therefore the GitHub issues has been disabled.

If you find a mistake, bug, or any other helpful suggestion you'd like to make on the docs please head over to [http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1) and raise it.

A comprehensive list of open issues can be found at [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)


Building
--------

This is a Swift Package manager based project. Add this repository as a dependency in your Package.swift file.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.Package(url:"https://github.com/PerfectlySoft/Perfect-Notifications.git", majorVersion: 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Overview
--------

This system runs on the server side. Typically at app launch, an Apple device will register with Apple's system for remote notifications. Doing so will return to the device an ID which can be used by external systems to address the device and send notifications through APNs.

When the device obtains its ID it will need to transmit it to **your** server. Your server will store this id and use it when sending notifications to one or more devices through APNs.

Obtain APNs Auth Key
--------

To connect your server to Apple's push notification system you will first need to obtain an "APNs Auth Key". This key is used on your server to configure its APNs access. You can generate this key through your Apple developer account portal. Log in to your developer account and choose "Certificates, IDs &amp; Profiles" from the menu. Then, under "Certificates", choose "APNs Auth Key".

If you haven't already created and downloaded the auth key, click "+" to create a new one. Make sure you select **Apple Push Notification Authentication Key (Sandbox & Production)**. This one key can be used for both development or production and can be used for any of your iOS/macOS apps.

Click "Continue" and you will be given a chance to download the **private key**. You must download this key now and **save the file**. Also copy the "Key ID" shown in the same view. This will be a 10 character string.

Finally you will need to locate your developer team id. Click "Account" near the window's top. Select "Membership" in the menu. You will then be shown much of your personal information, including "Team ID". This is another 10 character string. Copy this value.

Server Configuration
------

To send notifications from your server your must have three pieces of information:

1. The private key file which was downloaded
2. The 10 character key id
3. Your 10 character team id
4. An iOS/macOS app id

These four pieces of information are used to perform push notifications. This information must reside on your server. You can store this information in any manner provided it can be used by the server. For simplicity, the rest of this example assumes that the private key file is in the server's working directory and that the two keys and the app id are embedded in the Swift code.

In your server Swift code, you must `import PerfectNotifications`. Then, before you start any HTTP servers or send any notifications you must add a "configuration" for the notifications you will be sending. This very simply ties your APNs keys to a name which you can then use later when pushing notifications.

```swift
import PerfectNotifications

// your app id. we use this as the configuration name, but they do not have to match
let notificationsAppId = "my.app.id"

let apnsKeyIdentifier = "AB90CD56XY"
let apnsTeamIdentifier = "YX65DC09BA"
let apnsPrivateKeyFilePath = "./APNSAuthKey_AB90CD56XY.p8"

NotificationPusher.addConfigurationAPNS(
	name: notificationsTestId, 
	production: false, // should be false when running pre-release app in debugger
	keyId: apnsKeyIdentifier, 
	teamId: apnsTeamIdentifier, 
	privateKeyPath: apnsPrivateKeyFilePath)
```

After the configuration has been added, notifications can be sent at any point. To do so, create a `NotificationPusher` with your app id, or "topic", then trigger a notification to one or more devices by calling its `pushAPNS` function:

```swift
let deviceIds: [String] = [...]
let n = NotificationPusher(apnsTopic: notificationsTestId)
n.pushAPNS(
	configurationName: notificationsTestId, 
	deviceTokens: deviceIds, 
	notificationItems: [.alertBody("Hello!"), .sound("default")]) {
		responses in
		print("\(responses)")
		...
}
```

The topic is required when creating a NotificationPusher. Additional optional parameters can be provided to customize the notification's expiration, priority and collapse-id. Consult Apple's APNS documentation for the semantics of these options.

Public API
----

The full public version 2.1.x API for notification pusher follows:

```swift
public class NotificationPusher {
	
	/// Add an APNS configuration which can be later used to push notifications.
	public static func addConfigurationAPNS(
		name: String, 
		production: Bool, 
		keyId: String, 
		teamId: String, 
		privateKeyPath: String)

	/// Initialize given an apns-topic string.
	public init(
		apnsTopic: String,
		expiration: APNSExpiration = .immediate,
		priority: APNSPriority = .immediate,
		collapseId: String? = nil)
		
	/// Push one message to one device.
	/// Provide the previously set configuration name, device token.
	/// Provide a list of APNSNotificationItems.
	/// Provide a callback with which to receive the response.
	public func pushAPNS(
		configurationName: String, 
		deviceToken: String, 
		notificationItems: [APNSNotificationItem], 
		callback: @escaping (NotificationResponse) -> ())
	
	/// Push one message to multiple devices.
	/// Provide the previously set configuration name, and zero or more device tokens. The same message will be sent to each device.
	/// Provide a list of APNSNotificationItems.
	/// Provide a callback with which to receive the responses.
	public func pushAPNS(
		configurationName: String, deviceTokens: [String],
		notificationItems: [APNSNotificationItem],
		callback: @escaping ([NotificationResponse]) -> ())
}
```

The remaining structures, including APNSNotificationItem follow:

```swift
/// Items to configure an individual notification push.
public enum APNSNotificationItem {
    /// alert body child property
	case alertBody(String)
    /// alert title child property
	case alertTitle(String)
    /// alert title-loc-key
	case alertTitleLoc(String, [String]?)
    /// alert action-loc-key
	case alertActionLoc(String)
    /// alert loc-key
	case alertLoc(String, [String]?)
    /// alert launch-image
	case alertLaunchImage(String)
    /// aps badge key
	case badge(Int)
    /// aps sound key
	case sound(String)
    /// aps content-available key
	case contentAvailable
	/// aps category key
	case category(String)
	/// aps thread-id key
	case threadId(String)
    /// custom payload data
	case customPayload(String, Any)
    /// apn mutable-content key
	case mutableContent
}

public enum APNSPriority: Int {
	case immediate = 10
	case background = 5
}

/// Time in the future when the notification, if has not be able to be delivered, will expire.
public enum APNSExpiration {
	/// Discard the notification if it can't be immediately delivered.
	case immediate
	/// now + seconds
	case relative(Int)
	/// absolute UTC time since epoch
	case absolute(Int)
}

/// The response object given after a push attempt.
public struct NotificationResponse: CustomStringConvertible {
	/// The response code for the request.
	public let status: HTTPResponseStatus
	/// The response body data bytes.
	public let body: [UInt8]
	/// The body data bytes interpreted as JSON and decoded into a Dictionary.
	public var jsonObjectBody: [String:Any]
	/// The body data bytes converted to String.
	public var stringBody: String
	public var description: String
}
```

Additional Notes
----

APNs requests are made from your server to Apple's servers "api.development.push.apple.com" or "api.push.apple.com" on port 443. One request will be used when sending one notification to one or more devices. Each connection will remain open and will be reused when sending subsequent notifications. If a connection "goes away" or there are no idle connections that can be used then a new connection will be opened. This is in accordance with Apple's recommended usage of APNs and should provide the best throughput when dealing with many concurrent notification requests.

Consult [Perfect-NotificationsExample](https://github.com/PerfectExamples/Perfect-NotificationsExample) for a client/server combination which can be easily configured with your own information to quickly get APNS notifications for your apps.

## Further Information
For more information on the Perfect project, please visit [perfect.org](http://perfect.org).
