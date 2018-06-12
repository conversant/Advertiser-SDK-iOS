Conversant Advertiser SDK - iOS
==================

The Conversant Advertiser SDK provides a simple way to gather information on how customers use your application, and allows Conversant to subsequently tailor relevant and meaningful messages to your users based on their interaction and purchase history. The following documentation provides a high-level overview of integrating the Conversant Advertiser SDK - your Conversant Client Integration Engineer will provide a customized integration document outlining events and data for your app and unique purpose.

## Getting Started

Getting started with the Conversant SDK is easy, just follow these simple steps:
* Grab the latest release of the SDK here:  [https://github.com/conversant/Advertiser-SDK-iOS/releases](https://github.com/conversant/Advertiser-SDK-iOS/releases)
* Add the SDK to your App
* Create a ConversantTagManagerSDKInfo Property List
* Use the SDK in your App

### Adding the SDK to your Application

To include the SDK in your app:
  
  1. Drag the desired 'CNVRTagManager.framework' from the Finder into the Project Navigator of your Xcode Project
  2. Select your project in the Project Navigator
  3. Select your desired target under the targets list
  4. Select the General tab
  5. In the Embedded Binaries section, click + to search frameworks
  6. Select 'CNVRTagManager.framework' from the list of binaries that can be embedded

###### Notes
* The Conversant Advertiser SDK utilizes Apple's Core Data framework. If you do not use Core Data in your app, you will need to embed it.
* As a [dynamic library](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html) the Conversant Advertiser SDK allows for a universal binary to be used both on a device and in the simulator. For development, the 'Release-iphoneuniversal' will likely always be the correct version of the framework to embed. Unfortunately (at the time of writing) iTunes Connect does not automatically strip builds for unused architectures, and will not accept submissions with unused binaries. Before submitting to the your app to the app store you must remove the unused binaries either by:
  * Including the 'Release-iphoneos' framework in place of the 'Release-iphoneuniversal' framework before submitting
  * Adding a build script that strips unused architectures at build time as outlined [here](http://ikennd.ac/blog/2015/02/stripping-unwanted-architectures-from-dynamic-libraries-in-xcode/)

### Create a ConversantTagManagerSDKInfo Property List

To create a property list in Xcode:

  1. Select the Resources folder for your app in the Project Navigator
  2. Choose 'New File' from the File menu
  3. In the 'Resource' template category, select the Property List template
  4. Name the file 'ConversantTagManagerSDKInfo.plist'
  5. Add a Key of 'ConversantSiteID'
  6. Add a Type of String
  7. Add a Value with the CTM Site ID provided by your Client Integration Engineer

Alternatively, you can create a property list in a text editor such as TextEdit or BBEdit:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>ConversantSiteID</key>
    <string>CTM Site ID</string>
</dict>
</plist>
```

###### Notes
* Be sure to replace 'CTM Site ID' with your the Conversant Site ID provided by your Client Integration Engineer
* A sample ConversantTagManagerSDKInfo.plist is available in CNVRTagManager.framework. That file can be dragged into your project from the Finder, just be sure to check 'Copy items if needed' when doing so. 

### GDPR Support

The Conversant Advertiser SDK fully supports the [IAB's GDPR Transparency and Consent Framework In-App API](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/Mobile%20In-App%20Consent%20APIs%20v1.0%20Final.md) and will not collect data or sync events for EU users without explicit consent. If your app includes a [Consent Management Provider](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/v1.1%20Implementation%20Guidelines.md#publisher) ("CMP") that implemnents the IAB's GDPR Transparency and Consent Framework In-App API then no additional set-up steps are necessary.

If your app is not subject to GDPR requirements, you should explicitly declare that via [NSUserDefaults](https://developer.apple.com/documentation/foundation/nsuserdefaults) in order to prevent the Conversant Advertiser SDK from attempting to determine consent on each event. To do so, set the 'IABConsent_SubjectToGDPR' preference to "0" (as a String, not an Integer) when your app first initializes.

**Writing to NSUserDefaults Examples**

Swift
```
var isSubjectToGDPR = "0"
UserDefaults.standard.set(isSubjectToGDPR, forKey: "IABConsent_SubjectToGDPR")
```

Objective-C
```
NSString *isSubjectToGDPR = @"0";
[[NSUserDefaults standardUserDefaults] setObject:isSubjectToGDPR forKey:@"IABConsent_SubjectToGDPR"];
```

###### Note
It is strongly recommended that any CMP included in your application implements the IAB's GDPR Transparency and Consent Framework In-App API as an industry standard. To signal consent to the the Conversant Advertiser SDK without implementing the IAB's GDPR Transparency and Consent Framework In-App API you can overload the [NSUserDefaults](https://developer.apple.com/documentation/foundation/nsuserdefaults) the Conversant Advertiser SDK reads as a part of that specification. To signal that consent is true, set the 'IABConsent_SubjectToGDPR' preference to "0" (as a String, not an Integer). To signal that consent is false, set the 'IABConsent_SubjectToGDPR' preference to "1" (as a String, not an Integer). Any other SDK's that have also implemented the  IAB's GDPR Transparency and Consent Framework In-App API will be similarly affected.


### Using the SDK in your Application

The Conversant Advertiser SDK is event based. When a notable event occurs within your application (such as a customer viewing a product or making a purchase) the SDK allows you to 'sync' that event with Conversant.

To create an event, use the 'fireEvents' method of the 'CNVRTagManager':
```
/**
     Fire TagManager events.  This defaults to high priority.
     
     Parameter events: [String]
     
     Parameter eventGroup: String
     
     Parameter eventData: Dictionary<String, Any> must be serializable or event creations will fail gracefully
*/
    public static func fireEvents (_ events:[String], eventGroup:String, eventData:Dictionary<String,Any>) {
        self.fireEvents(events, eventGroup: eventGroup, eventData: eventData, priority: .high)
    }

```

###### Notes
Each event includes 3 types of data
* **Event Names** (Required) Used to identify specific events that occur in the application. Multiple events names can be passed.
* **Event Group** (Required) Used to hierarchically organize events within the application. Multiple groups may be delimited with forward slash '/'
* **Event Data** (Optional) Used to pass any arbitrary data that may be necessary or useful to track along with the event(s) and group(s)

It is recommended that you work with your business unit to determine which events should be tracked, and how best to organize the application in to groups. When in doubt, liberally adding tracking for events is always recommended, as the Conversant Advertiser SDK is designed to only communicate events that have tracking assigned to them by your Client Integration Engineer.


**Example Events**

Swift - A log-in event example which is sending the users MD5 email hash as a variable:
```
CNVRTagManager.fireEvents(["login", "signup-success"], eventGroup: "signup/login", eventData: ["email-hash":"emailhash"])
```

Objective-C - A retail event example for view of current product:
```
[CNVRTagManager fireEvents:@[@"product-view"] eventGroup:@"category/subcategory/product-page" eventData:@{@"productSKU" : @"AB1234"}];
```


## RELEASE NOTES

For the latest, up-to-date information on iOS SDK features and bug fixes, visit our iOS SDK Release Notes [https://github.com/conversant/Advertiser-SDK-iOS/releases](https://github.com/conversant/Advertiser-SDK-iOS/releases)

## LICENSE

The Conversant Advertiser SDK is licensed under the Apache License, Version 2.0 [http://www.apache.org/licenses/LICENSE-2.0.html](http://www.apache.org/licenses/LICENSE-2.0.html)
