## CoreData Q&A

Q: When you create a new project in Xcode 14, select Use Core Data and Host in CloudKit, the template code does not enable persistent history tracking via description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey). Is this enabled by default when using NSPersistentCloudKitContainer? I thought this was required to power sync but my app seems to work just fine without enabling it so I am curious what this does. Thanks!

A: That is correct, the NSPersistentCloudKitContainer will enable Persistent History by default

---

Q: I have an app that uses CoreData with CloudKit syncing, and it uses both a private and a shared database. Now, I want to allow my users to associate photos with one of the objects in my existing model. I was experimenting with storing the image in a Binary Data field on one of my objects, but I wasn’t sure if this was the best way to accomplish what I want. Is there a better way?

A: Large files should be kept across a relationship to the objects that need them, in a Binary Data field with the Allows External Storage box checked.

If the files are really large (over 100mb), they need to be kept as files on disk. iOS applications cannot allocate contiguous buffers larger than that.

These files can be synced to CloudKit on the side, in a zone and record you own using CKSyncEngine or native CloudKit operations.
This technique allows them to exist entirely separately from your CoreData graph and NSPersistentCloudKitContainer / your CoreData code won't be tempted to fault them in to memory.

---

Q: I have an app that uses NSPersistentCloudKitContainer. How does the introduction of SwiftData affect this setup? Is there a good migration path forward?

A: there is a session about co-existence and migration available tomorrow.  Basic use of NSPersistentCloudKitContainer is easily migrated, but sharing isn't available in SwiftData at the moment,

---

Q: Hello, when using Core Data syncing with CloudKit (NSPersistentCloudKitContainer), I would like to have the ability to download data fields on demand. Let's take an image app (like Photos) as an example: currently, if you have a data field for an image and you have a large number of images (gigabytes, for example), all the images are downloaded regardless of necessity. What I would like to have is a way to selectively download the image data (or any other blob field). Currently, I can achieve this by using string fields as a type of “link” for the asset IDs on CloudKit, but this approach is error-prone, particularly when it comes to sync deletion. Moreover, I have to manually upload and download this data using CloudKit APIs, which is not an ideal scenario. Do we have any native solution for this or any plans for implementing one in the future?

A: Please file a feedback report.

---

Q: I want to use Core Data to sync with a CloudKit public database. Is it okay if I put about 300MB of data in that public database for my users to sync against?

A: Yes, the public database can support this use case. You will want to structure your object graph in a way that preserves performance and memory growth on customer devices, like ensuring large blobs are not accidentally faulted in to memory.

---

Q: Can NSPersistentCloudKitContainer be given a shorter name?  I know all of the words in the name, but I can never remember what order they go in!

A: Yup!  It's name has been deleted in SwiftData entirely!  Just set the entitlement in your project.  The entitlement named uhm named ...

Follow-up-A: named:  https://developer.apple.com/documentation/cloudkit/enabling_cloudkit_in_your_app

---

Q: I have a huge amount of private data on my test device that is syncing CoreData with CloudKit using NSPersCloudkitContainer. When I last tried to install the production version of the app, most of the on-device data disappeared. I immediately deleted and hoped that the development container had successfully synced it all. It had thankfully. But how do we migrate data from a dev container to the prod container? What happens when you install an app in xcode and add a bunch of data, and then follow up later by installing it via TestFlight?

A: There is no migration between the development environment and the production environment. You can manually move records from one store to another, so for example you could facilitate a backup / restore function to a local store in your TestFlight builds. But in general our recommendation is not to allow non-engineering workflows in the development container.

Doing so makes it extremely difficult to reliably make changes to the schema in the development container. cooperative devices can saturate fields you didn't mean to promote if they are still active, and resetting the development environment to undo a mistaken change can cause data loss for users.

---

Q: How does SwiftData save enums and structs into Model classes? Is it a bit like fetched properties in Core Data? Is it performant for very large datasets (100K+), or should I use another approach for saving data in very large datasets?

A: SwiftData requires that enum and structs conform to Codable in order to be persisted for the Model Classes

In Core Data, the equivalent property would be represented in the NSManagedObjectModel as a NSCompositeAttributeDescription - this property on the entity is a NSDictionary representation of the Codable enum or struct

---

Q: Is there some best practice / sample / guide on making Core Data conform to Transferable, so copies of the object graph could be easily shared between users of the app?

A: NSManagedObject is not Sendable, Transferable, or otherwise portable to work outside it's managed object context.

Transfer representations should reference a managed objectID (for use against the same store file) or a unique identifier you own (for use across devices, stores, etc). They can include whatever metadata you need to saturate a row on the receiving device.

---

Q: I have a home screen widget that displays info from an NSPersistentCloudKitContainer accessible via App Groups. If I add a button to the widget, can I change the data in this database, and will it sync back up to iCloud? I assume the app is launched in the background briefly so I'm thinking the answer is no but wanted to get your thoughts

A: If the application is set up with NSPersistentCloudKitContainer, then when it runs it will sync the widgets changes.  The widget itself doesn't really get any background time, so it's difficult for network ops to complete before its suspended

---

Q: Can NSPersistentCloudKitContainer's Sharing capability be used with Swift Data?

A: CK sharing isn't currently available in SwiftData.  You can have CoreData and SwiftData share a data file, and build a UI with SwiftData (no syncing) and simultaneously have CoreData NSPersistentCloudKitContainer pointing at the same URL handle the CK mirroring and sharing.

---

Q: Calling the initializeCloudKitSchema is only required once after a model update. It does not have to end up in the production app. What is the best practice to perform this call? Once in a private TestFlight build and then remove it again for production?

A: Saturating your new schema in CloudKit should be a controlled activity done “at desk” by you prior to release and not in a TestFlight build. You could have this code run if an environment variable is present or some other way.

---

Q: I have had a really hard time understanding the data storage limitations for CloudKit. I've got a huge model with gigabytes worth of data (photos, videos, audio). If a user uploaded 5 gb of data in one day.. is that going to just totally stop syncing for a user because they went past their quota? What is their quota? The website seems to have scrubbed all mention of it

A: It is not possible to know the amount of quota space a user has, they manage that in their iCloud settings.
Users that exhaust their quota space will receive a quotaExceeded error: https://developer.apple.com/documentation/cloudkit/ckerror/2325197-quotaexceeded

NSPersistentCloudKitContainer logs these errors in various NSPersistentCloudKitContainerEvent instances that you can use to discover if the device has run out of iCloud space.

You do not need to do anything special, NSPersistentCloudKitContainer automatically handles these errors and recovers from them when space becomes available (it tries once per app launch or so).

---

Q: SwiftData looks awesome and I'm looking forward to starting to adopt it. My only question at the moment is related to using it in conjunction with an existing NSPersistentCloudKitContainer with CloudKit shared data. Is it possible to go from a @Model object to its NSManagedObject counterpart? Or more importantly, obtain the NSManagedObjectID for the SwiftData object that can be used as needed to do the sharing aspect?

A: You can file a feedback request.  @Model offers an objectID which is a PersistentIdentifier which has a URI format which might take you places.

---

Q: In SwiftUI, Core Data objects conform to ObservableObject, so our views can update automatically when the object changes. For example: struct DetailView: View { @ObservedObject var item: Item ... } With Observable replacing ObservableObject/ObservedObject property wrappers, will CoreData objects automatically conform to Observable? What happens if ObservedObject is deprecated?

A: ObservableObject is not deprecated.  Please file a feedback request regarding Observable support.

---

Q: I have an app that uses NSPersistentCloudKitContainer, and I'm adding support for CloudKit Sharing this summer.  This is an environment that seems more likely to run into update collisions, where two family members make changes to data near the same time.  
How is this scenario accounted for using NSPCKC, and do I have any ability (or need) to help the users resolve this collision?

I asked this question earlier during a CloudKit lab, but they didn't have experience with the CoreData interface, and referred me here.  They did indicate that in CloudKit, I would need to listen for a ServerRecordChangeError - is that exposed to me, or does NSPCKC handle it somehow?

A: NSPersistentCloudKitContainer does not expose server conflicts. You can file a feedback report to describe how you would like to handle these conflicts and what your expectations are from such an API.

In general we encourage clients to use graph structures that avoid conflicts (by capturing contributions instead of flat values) for collaborative experiences.

Follow-up-A: I recommend you take a look at the sample app for this. It has recently been updated as well. Works for me https://developer.apple.com/documentation/coredata/sharing_core_data_objects_between_icloud_users

Follow-up-Q: Thanks Nick - I'm not entirely sure what I expect, which is why I asked the question.  Digging in to your point about graph structure, is that to say that if I have a 'root' NSManagedObject called 'Budget', and two users independently add two separate 'Payment' objects into a 'payment' array, that those changes would not result in a collision, because they're on separate objects?
(where Budget.payments is an array of Payment objects -- another NSManagedObject.  Sorry if that's hard to understand)

Follow-up-A: Yes, all relationships are implicitly mergeable on the to-many side. The to-one side can suffer from collisions. Many-to-many relationships will always bias towards inserts, one-to-many relationships are last writer wins merged, so there is some randomization of the outcome there.

---

Q: It might be an obvious question but just wondering, should I use SwiftData for very large DBs (as in messages and media)? Or is it only suitable for small ones like the ones shown in the sessions?

A: SwiftData aims to scale, if you find any issues we'd love your feedback!

---

Q: Is it possible to set up hierarchical sharing manually with SwiftData and CloudKit?

A: No.  Zone sharing is the way.

---

Q: How do I get the graphical editor when looking at a Core Data schema? I can't seem to find it. Has it been removed recently?

A: We uh… encourage you to file a feedback report with Xcode. Thanks.

---

Q: Which background mode should be activated to enable the system to wake up an app when NSPersistentCloudKitContainer  has changes to download? „Background Fetch“ and „Remote Notifications“?

Is
BGAppRefreshTask the way to go here? Does the system execute our BGAppRefreshTask when CoreData downloads new data from CloudKit most of the time? I thought execution is based on usage patterns of the app not events like changes downloaded from the cloud.
If not, which delegate callback must be implemented that the system calls when data has been downloaded from iCloud?

A: NSPersistentCloudKitContainer schedules activities on behalf of your application. The system should be able to launch your app in the background as long as you have the Background Notifications capability enabled.

In addition, you may choose to support additional background modes for which you schedule your own tasks, and within those modes use NSPersistentCloudKitContainer to sync.

---

Q: Is there a way we can visually see the schema, relationships, etc of Models in SwiftData?

A: Please file a feedback request.  Not at this time.

---

Q: Hello! Would having an entity with about 100 properties be an issue for CoreData?

A: Not a problem.

Follow-up-A: You do get warnings in console when you have an entity with a lot of properties. But it seems to work for me. I have an entity with more than 100 attributes and relationships.

---

Q: In the "Model your schema with SwiftData" session, there is a recommendation to use a VersionedSchema as a root for all types. I assume the recommended way is to actually provide the enum itself in a simple file that contains the list of models, and then to add extension to the enum in other files for all the @Models , since I got hundreds of different classes.

Another question on the VersionedSchema: in the end of the presentation, there is a Trip.self, but it doesn't seem to lie in a VersionedSchema. It's typedefed to the outside?

A: VersionedSchema should be used for the Schema from a past release, not the current release as shown in the session.  The Trip.self is the current version of Trip that is not namespaced into a VersionedSchema by encapsulation in an enum

---

Q: Can you specify a custom CKRecordZone name when using NSPersistentCloudKitContainer? My app is a document app and each document can be synced separately. Even to different sync services. CloudKit is just one option. So it would be great to be able to sync the entire zone separately for each document.

A: No. Please file a feedback report: http://feedbackassistant.apple.com/

Include details about your use case, the expected number of documents your application wants to manage (there is a 1000 zone limit), and any other details about your intended product experience you can (i.e. do you intend to support collaboration, sharing, etc).

---

Q: After modifying data from a shortcut or widget using SwiftData, my UI is not refreshed when opening the host app. I'm observing my entities using a @Query parameter. Is there any way to force a refresh?

A: Please file a feedback request.  There are some known issues in this space for Seed 1.

---

Q: I save an UITextView's NSAttributedString directly into a transformable attribute. I hadn't set the NSSecureUnarchiveFromDataTransformer before, and now I get a lot of warnings in the console about this. When I tried to set a custom secure transformer on this, the app crashes, because apparently, deep inside NSAttributedString, it's saving some private classes that don't conform to NSSecureCoding (like _UITextInputDictationResultMetadata). What's the best way for me to handle this situation?

A: A feedback report with a test case that reproduces this would be helpful.

---

Q: Is it possible to use SwiftData to persist a single object? I basically have a Singleton that I want to use to store my entire App state, and sync it across platforms. Would SwiftData be a good choice for that?
I tried using it that way but got an error that @Query only works with Collection

A: @Query is designed to return a collection of Models to drive your SwiftUI View, however you can also use fetch() on the ModelContext to get the collection and find the singleton object and then setup your View with that singleton result.

---

Q: I would like to integrate SwiftData with my SwiftUI app. I would like to ask that will .modelContainer(for: [Class.self]) init brand new database instant every time? Or it can detect the database creation status automatically? What if I have multiple modelContainer mods with the same instance class? I am wondering this because I my mind, the database should be initialised by a dedicated code, and I want to know that why use the modifier methods to create database structure.

A: The ModelContainer should create a database the first time and use the same one on subsequent launches.  Additionally other instances of the ModelContainer should be interacting with the same file.  If this is not the behavior you are experiencing please file a feedback report with an example project.

---

Q: I have an app that uses multiple datasets that have no relationship to each other. Does it make sense to create four separate Core Data databases using separate instances of NSPersistentContainer? What are the advantages and drawbacks of using separate databases? Any guidance on how to do this will be appreciated.

A: Multiple store files increase the idle memory usage of your application by a few hundred KB per store file.
That said, they produce greater isolation and resource scaling (store files grow in relation to their contained dataset, offer distinct concurrency, etc).

In applications, we don't typically have specific recommendations for the number of store files an application should manage, do what feels right.

In more constrained experiences like widgets and extensions, it may be convenient to host the widget graph in a single file that can be loaded in isolation to avoid this idle heap growth.

---

Q: This is a follow-up to my previous question on storing photos in core data… I wanted to clarify that the binary data field with external storage enabled can be on the primary record? Your response sounded to me like maybe you were recommending I create a second child object with just the binary data field. I wasn’t sure if there would be any benefit to doing that?

A: The benefit is that the relationship isn't faulted any time you read a non-prefetched value from your row. If it's in the row, it will fault in to memory with the rest of the object wether you intend to use it or not.

---

Q: Hello! A few questions for my project.
Can lightweight migration handle changes in data types? For example an Int64 to a Int16.
A related questions in if the Int64 will take up that much more space then Int16, and what Int type you would recommend for normal uses. For example, logging a monthly report for a single person (hours spent on something).

A: Int64 → Int16 is tricky (and will fail) because of the chance of integer truncation, but the other direction should work. The good news is that SQLite's record format stores integers in a variable-width form, so even small values of Int16 only occupy one byte of storage.

At the moment the migration will fail even if all your values fit in Int16, but we could look into being more lenient and actually checking for outliers if you file a feedback report
