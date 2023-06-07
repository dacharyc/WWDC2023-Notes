# CloudKit Q&A

Q: How to determine if I should use CKSyncEngine or NSPersistentCloudKitContainer? What are their best use cases? I plan to use SwiftData, public, private and shared CloudKit databases and deploying on iOS 17.

A: Please see https://developer.apple.com/documentation/cloudkit/deciding_whether_cloudkit_is_right_for_your_app

---

Q: I’m getting an error - “Operation throttled by previous server http 503 reply. Retry after 57.7 seconds. (Other operations may be allowed.)”

The predicate is something like:
ANY {"x", "y", "z", "a", "b"} == someValue
Is this okay? if not any recommendations

A: That error indicates that a previous operation failed, and doesn't necessarily indicate any issues with your query.  The general form of your query is okay, some more detail is available at https://developer.apple.com/documentation/cloudkit/ckquery
The best way to understand what caused the original 503 is to file a Feedback via FeedbackAssistant, and include with it a sysdiagnose.  Including additional details from your app layer (approximate timestamp of failure being the most important) will help us narrow in on the earlier failure.  Bonus points for installing the CloudKit logging profile at https://developer.apple.com/bug-reporting/profiles-and-logs before reproducing, as that generates even more useful data in the sysdiagnose.

---

Q: CKSyncEngine looks great, but I won't be able to require iOS 17 for a bit. I already have some custom CloudKit code - is there any way I can approximate some of its behavior with my code? Specifically, I'd like to be able to determine when the system and/or network is too busy to start a sync.

A: I'm glad you like the look of CKSyncEngine, but sad you can't use it yet! You can determine network reachability using SCNetworkReachability, but there's currently no way for you to determine whether the system is in a good state to sync. Would you mind submitting some feedback requesting an API do to that?

Here's the documentation for SCNetworkReachability: https://developer.apple.com/documentation/systemconfiguration/scnetworkreachability-g7d

There's also some sample code for that here: https://developer.apple.com/library/archive/samplecode/Reachability/Listings/Reachability_Reachability_m.html#//apple_ref/doc/u[…]ontLinkElementID_9

---

Q: hello CloudKit team, CKSyncEngine looks like a great helper framework. I'm using CoreData for local offline storage of user data (I'll probably switch to SwiftData) could you offer any tips on how to bridge between CoreData/SwiftData classes and the structs used by CKSyncEngine as used in the sample code? Can I use classes directly with CKSyncEngine (assuming that the are not supported by SwiftData)?

A: You can bridge these to the CKSyncEngine by implementing your own serialization from your data model to CKRecord.
Any structs, SwiftData Models, or other objects will be serialized in to your desired record schema.

It sounds like you might be implementing sync in your application for the first time. Before starting on an implementation please take a look at Core Data Mirroring first. Depending on your use case, that may be an easier approach!
https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit
https://developer.apple.com/xcode/swiftdata/

---

Q: Why there is a delay from saving a record in CloudKit database to be able to retrieve it via a query?
I have an app which I save my users “favorites” in CloudKit, but then when I “reload” their favorites (fetching from CloudKit database) not all of the records are returned, but after 5-10 minutes they are.

A: Unfortunately, that's a side effect of the underlying architecture, however we do not expect 5-10 minutes of delay. You should ideally be seeing a few seconds or less. Feel free to file a Feedback about the specific CKOperations where you see that amount of delay.

You can also use the fetch changes APIs to not hit this, but I recognize those aren't available in the public database, which it sounds like you might be using here.

---

Q: This is kind of a user interaction question, but when a user initially installs an app, does Apple recommend turning on CloudKit sync automatically (with an option to disable it), or waiting for the user to actively turn on cloud syncing? I would prefer to turn it on, and let users opt out only if they don't need it.

A: This entirely depends on UX for your app, but it is pretty common and not unexpected for your data to be there on launch.

---

Q: Is there a limit to how big of a file you can upload? What is that limit? There are no hard answers. I'm trying to store videos backed by CloudKit.

A: There are varying specific limits for different APIs, but if you’re using CloudKit.framework (on device, in a native app), typical video file sizes should be fine. It’s worth filing a feedback for improving our documentation regarding these limits!

---

Q: Any guidance on why/when you might use a CloudKit public database versus App Store On-Demand Resources for shared media assets for all users of your app?

A: To the best of our knowledge, On-Demand Resources in the AppStore require a new binary each time assets are updated, but a PublicDB in CloudKit can be dynamically updated at any time. Recommend checking with the App Store team on what On-Demand Resources capabilities are with respect to modifying or adding new assets

---

Q: Today I created a new iOS sample project in Xcode 15 beta. I chose SwiftUI and Swift and SwiftData and I checked the CloudKit box. When I opened the project, it appeared it had done all the necessary capabilities for using CloudKit that you used to have to manually do in the target project capabilities screen. Is this all that needs to be done to make SwiftData work with CloudKit? It appeared to be so.

A: That should be all you need to do! Are you seeing the data sync properly across devices?

---

Q: Since unique attribute constraint isn’t possible with CloudKit, is it a good idea to make a request for an attribute value (which is supposed to be unique) and check if there is any item resulting before adding the new item to my model?

A: In general, try and avoid any read-decide-write flows, as you're going to be subject to losing races, and writing outdated info.

You can use leverage the uniqueness of a CKRecord's recordID to accomplish this, via denormalization; incorporate the unique field value into your CKRecord.ID.

Note that some CKRecordZones have the CKRecordZone.Capabilities.atomic capability, which allows you to save multiple records and have all succeed, or all fail, which you might be able to leverage.

Follow-up-Q (by another dev): Did I understand this question correctly: you can’t use @Attribute( .unique) for @Model model properties with CloudKit?

Follow-up-A: It could be that I misunderstood the question; I took it as a reference to CloudKit not offering a way to mark a CKRecord field value as "must be unique across CKRecord instances" as a property of the container's schema.

---

Q: When I create a CKShare from my app, with no public permissions, and send someone else the URL who isn't a part of the share's participants, they just get an alert with a message "Item Unavailable, The owner stopped sharing, or you don't have permissions to open it".... is there a way to make a more specific message or UI in this case?

A: It isn't possible for apps to customize the "item unavailable" alert for a private share when using the system share acceptation flow (i.e. user clicks/taps a share URL before your app gets a chance to launch).

We're curious how you're ending up in this state though so if you could file a Feedback describing more of the context it would help us determine whether you're running into a bug which results in an intended participant not getting invited correctly or if your query was limited to customizing the error alert when this scenario is forced (e.g. copying the share URL and sending it out of band to an uninvited user). Thanks!

Follow-up: It's more the latter. We can get there frequently due to a user error in setting the correct participants in the CKShare (since it doesn't use the system CKShare view controller). If the user sends the link to a misconfigured participant, they see this error, which doesn't make much sense to them. It would be great if this was more helpful, or open our app (which created the CKShare in the first place) so we can give them better support

---

Q: If I'm going to use SwiftData can I use Cloudkit sharing?

A: There are a couple of options for sharing with SwiftData.
1. Use NSPersistentCloudKitContainer on the side. Consider the session Migrating to SwiftData for more information on that.
2. Use CKSyncEngine or native CKOperations to mirror the data in the ModelContainer to CloudKit.

There are a number of relevant documents:
https://developer.apple.com/documentation/cloudkit/deciding_whether_cloudkit_is_right_for_your_app?language=objc
https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app?language=objc

---

Q: Maybe a little beyond scope... but what are some potential workflows for getting media assets into an app's public database without using the console?  For example, in AWS a new object in an S3 bucket could trigger a Lambda function.  Could I use the CKToolNodeJsModule in a node lambda to then write the asset into CloudKit?

A: Hi John. There are a couple options here including a CLI tool we have included in Xcode developer tools. More below!

You can find more info on cktool specifically here and it can be invoked on your command line with xcrun cktool. cktool has commands for saving records and supports asset uploads. For authorization you'll need to use a user session token retrieved from CloudKit Console.
