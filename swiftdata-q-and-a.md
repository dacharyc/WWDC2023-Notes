# SwiftData Q&A

## My Questions

- Supports cascading deletes - how deeply? Does it actually delete the related objects or just set the relationship to nil?
  - Q: How many levels deep can a cascading delete - delete? i.e. if a Person has a relationship to Dog, and if Dog has a relationship to Toy, can a cascading delete from Person delete Toy or just Dog?
  - A: As long as the relationship is set up as cascade all the way down, it will continue to cascade.
  
- Supported data types - data?
  - Q: Is binary data a supported data type?
  - A: Yes it is!

## Other Qs

Q: With CoreData, you cannot use constraints with CloudKit. With SwiftData there is the “@ Attribute(.unique)” property wrapper (?, if that’s what it’s called lol). Does this work with CloudKit?

A: No, you should file an enhancement request for unique constraint support when working with SwiftData + CloudKit.

---

Q: Does SwiftData have the capability to run fetch on updates in the View Model of a MVVM architecture?  Most VM objects provide data prepping for the view/swiftUI struct view and swiftData might not be able to support MVVM?  As a conversation topic SwiftData might eliminate View Models? Maybe MVVM architectures are becoming obsolete due to the nature of swiftUI’s temp struct views?

A: SwiftData harnesses Observation to update the SwiftUI views and so the PersistentModels can be your VM objects and work great!

---

Q: Hi, we have multiple apps that have severel 100k of database entries. Our first tests, with your WWDC23 demo project, show that writing as well as reading takes very long, at least some 6-7 seconds and consumes lots of memory on an iPhone 11 Pro (our reference device). Is this the performance we have to expect from SwiftData? Can we improve this to come at least under 1 second?

A: Hi Simon.  There are some known issues in seed 1.  You can file a bug report to get updates

---

Q: If we have a coexistence of Core Data or fully adopt SwiftData and need to have threads updating the managed context at the same time is SwiftData and Core Data contention protected?  I have used Operation queues to protect the multiple threads from update and create a-periodic contention, but would like to understand if SwiftData is protecting with async-await in the background and if so how can Core Data and SwiftData avoid threaded managed context aperiodic problems.

A: @Model and ModelContext are not sendable.  You can't pass them to other actors.  You can create other ModelContexts using the ModelContainer (which is sendable) on other actors.

---

Q: Is it possible to use public, private and shared CloudKit databases with SwiftData ?

A: Yes, but only via co-existence. ModelConfiguration doesn't know how to express the capabilities of NSPersistentCloudKitConfiguration. You should file an enhancement request for that. Today you will need an NSPersistentCloudKitContainer on the side to manage the store descriptions, mirroring, and sharing API invocations via CoreData objects (NSManagedObject, etc). Your application can still use SwiftUI + SwiftData to manage its data, but via a non-syncing ModelConfiguration to the same store file.

---

Q: What's the recommendation if my app is mixed Objective-C and Swift using Core Data? My model layer is all in Swift. But I have many views and controllers that are written in Objective-C that still need to access Core Data entities. Can I use SwiftData models from Objective-C?

A: SwiftData uses Swift native classes and cannot be used with Objective-C.  However you can coexist with Core Data and use new views with SwiftUI to start your SwiftData adoption!

---

Q: Does SwiftData allow for optional Int/Double types? For CoreData those values are always defaulted to 0 in fetched objects, seemingly because of their bridging to Objective-C.

A: Indeed SwiftData does allow for optionals of scalar types.

---

Q: I see that BackingData is declared as a protocol, with CoreDataBackingData being a concrete implementation. Does this mean we can insert a custom persistence mechanism into this system?

A: Not at this time

---

Q: In Core Data I often used child contexts to allow my SwiftUI views to manipulate a copy of a managed object that I could either discard or save depending on the user's action. Is there a similar mechanism in SwiftData or can we only have 1 instance of a given underlying model object due to SwiftData not having child or background model contexts?

A: SwiftData doesn't currently offer child contexts, but you can create background ModelContexts as much as you like.

---

Q: It does not seem possible to group results from a Query (like it is possible with NSFetchRequest, NSFetchedResultsController or SectionedFetchRequest). Do you confirm this is not supported? Here is my FB12236319, because I really think this should be in the framework (in both Query and FetchDescriptor)

A: thanks

---

Q: Ive recently started transitioning my unreleased app core data. It previously used a JSON file to save and load data, which is not ideal because the entire file was loaded each time. Would it be better to pause my transition to core data and wait for Swift Data, or should I continue the transition to core data and then to Swift Data later on?

A: Depending on your backwards compatibility goals, SwiftData is iOS 17+ and Core Data goes back many releases.  You will most likely want to coexist for a few releases

---

Q: How should we handle large data import that used to be made on a background context associated with a background queue? Do we also need another Core Data stack (with a NSPersistentContainer) that coexist with the ModelContainer used in the UI?

A: Make a modelContext on a separate actor

---

Q: After your data store is converted to SwiftData, can you read that store with existing UIKit CoreData and through SwiftData?

A: Yes, using co-existence. The MOM and the Schema must be entity version hash equivalent

Follow-up-Q: How does one ensure they’re entity version hash equivalent? Sorry if that’s a dumb question. Is it just maintaining the exact same set of attributes, relationships, and properties in the model for an entity?

Follow-up-A: Yes, you can verify by checking entityVersionHashesByName

---

Q: If SwiftData and CoreData exist in the same app, do they both use the same underlying store?

A: Yes, the two stacks are talking to the same persistent store

---

Q: Does SwiftData offer a complete replacement for Core Data at this time? In other words, in a fully Swift codebase, could the entirety of an existing CoreData + NSPersistentCloudKitContainer setup be switched over to SwiftData, with all existing capabilities?

A: There are a few aspects that are not possible such as Abstract Entities but please let us know via Feedback of any enhancements that block your replacement

---

Q: Does SwiftData support context merging? And if not, what is the standard way to create and edit entities without affecting the main context?

A: it does, although there are some known issues in seed 1.  You can create additional ModelContexts with its init method

---

Q: Does SwiftData support extensions? For example, can changes to persisted storage from an extension be told to the main app?

A: Yes, using a shared app container, they extension & your main app can write to the same database.

--- 

Q: Does a Model that has a unique attribute automatically conform to Identifiable?

A: PersistentModel is Identifiable but it does not use the unique attributes for this, please file a feedback with an enhancement request with your use case!

---

Q: Are there any ways to organize mutation of SwiftData store in transactions similar to sqlite?

A: Mutations to model objects are captured in a ModelContext until save is called. save necessarily creates a transaction.

---

Q: Uhm, If I've understood correctly, if I want to save my SwiftData data into a CloudKit  private/shared database, I have to manually mirror the SwiftData models to cloudkit entities / NSPersistentCloudKitContainer managed objects, so what you mean is that SwiftData can coexist with CloudKit, but SwiftData doesn't knows anything about cloudkit and can't automatically mirror data on it like CoreData can do instead, right ?

A: There is no need to manually mirror, setting up a ModelContainer with a ModelConfiguration noting your CK Entitlement will setup a persistent backend with CK syncing enabled!

Follow-up-Q: Ok, there is a session where do you explain how to do that that i can see ?

Follow-up-A: I do believe that may be noted in the Dive deeper into SwiftData

---

Q: Are there any ways to encrypt data on db level? How I can request to fully encrypt the database?

A: you can define your Model Attribute with option @Attribute (.encrypt) to encrypt your property.

Follow-up-Q: Any chance to fully encrypt database file similar to SQLCipher?

Follow-up-Q: I thought the encrypt property was just for CloudKit? At least that’s what the label says in Xcode in the modeler. Allows cloud encryption. Does that also encrypt on disk? If so, what key does it use to encrypt?

---

Q: Does SwiftData provide support for CloudKit record sharing in a similar way as NSPersistentCloudKitContainer does?

A: SwiftData supports synchronization with the CloudKit private database via the cloudKitContainerIdentifier (or just having one in your entitlements) property on ModelConfiguration. https://developer.apple.com/documentation/swiftdata/modelconfiguration/
If you want to use the Shared or Public databases you will need to use NSPersistentCloudKitContainer.

Follow-up-Q: Are there any resources on how to do this? On how to use SwiftData to mirror shared CloudKit objects.

---

Q: Is there a recommended SwiftData analog for the "Loading and Displaying a Large Data Feed" sample Core Data project? My app inserts lots of non-deterministic entities one-by-one over short time periods

A: Implement an object that conforms to the ModelActor protocol to allow you to do your work on a background queue.  Use your ModelContainer to create the ModelContext that is used to initialize an instance of DefaultModelExecutor.  Initialize this object on your background queue and do your work on a function on this actor using it's context.

---

Q: I am trying to add a property that is a codable, but getting a crash when trying to load an empty data set. Is there something I should be doing to handle enums?

A: Please check the release notes for any know issues, if you do not find the issue there, please file feedback and we will dig into the issue.

---

Q: how does transformable attribute, like UIColor, works in swift data ?

A: I would recommend using NSCompositeAttributeDescription for this.

---

Q: Are batch/bulk operations supported in SwiftData?

A: Yes, you can perform batch operations.

---

Q: From the demos it looks like using persistent history to coordinate between apps and their extensions is not longer required or internally handled. Is that the case?

A: Yes, SwiftData automatically turns on persistent history tracking

Follow-up-Q: So I assume that means changes in extension are merged into the main application and vice-versa and transaction history are intelligently deleted.

Follow-up-A: Indeed the remote changes will be used by the main application to consume the changes

---

Q: Does SwiftData support SUBQUERY()? And how about derived attributes?

A: Yes! You can use #Predicate to create predicates to use for your fetch queries which can include operations like calling .filter(_:) on collections, which is supported with SwiftData. If there are additional operations you'd like to support in your predicates with SwiftData that you don't see supported, feel free to send us a feedback request! However, derived attributes like in CoreData are not supported. You can write a computed property and label it with the @Transient attribute instead which may help but not quite the same as CoreData's derived attributes.

---

Q: Do we have the equivalent of CoreData abstract classes ?

A: No, if this is something you'd like to request support for, please file a Feedback Report.

---

Q: How does model versioning work in SwiftData?

A: We recommend using enums to version your models. You can use typealias to always point to the most recent one

```swift
enum ModelVersion1 {
  @Model public class Entity
  ...
}

enum ModelVersion2 {
  @Model public class Entity
}
```

---

Q: Are SwiftData models thread restricted in the same way as NSManagedObject models? That is, can you only use them on the thread in which they were created? If so, what is the best practice when using these models in a system that utilizes swift concurrency?

A: @Model and ModelContexts are not sendable.  You can create new model contexts on other actors with the sendable ModelContainer

---

Q: What do we need to know about the multithreading story? Is it functionally different from CoreData in any way?

A: the compiler will enforce sendability

---

Q: We are using view model controllers to prepare model data for our views (sorting, grouping, augmenting, transforming…) to make the logic testable. How can SwiftData fetches be performed except for the @Query property wrapper that only works in views I suppose?

A: You can perform fetches directly on a ModelContext instance for full control, which is the same API that @Query uses under-the-hood.

---

Q: I see there is an externalStorage property option, does Swift Data handle the location of the object and hold a reference? If I wanted to store images would this be what I should be using and should I use the SwiftUI Image type or raw data?

A: Yes, SwiftData will manage the location. Use `Data`

---

Q: Can you briefly talk about persisting encrypted data using SwiftData?  Does SwiftData offer any form of encrypted storage (beyond the traditional data protection levels in iOS) or would I need to encrypt/decrypt the data myself before storing and after fetching?

A: You should be using the traditional data protection levels for encryption at rest on device. You can use .encrypt for encryption in CloudKit.

Follow-up-Q: the .encrypt attribute (cited on another answer as well) is just for CloudKit encryption?

Follow-up-A: Correct

Follow-up-Q: I don’t think the standard iOS protection levels work on Mac though, do they? Would be great to be able to specify an encryption key for CoreData / SwiftData to separately encrypt the entire store.

Follow-up-A: I believe they do

---

Q: Following up on versioning, will SwiftData let you control when a lightweight migration is performed instead of doing it automatically on instantiation as it is in CoreData?

A: To open a store, SwiftData must migrate the store to the version you indicated is the most recent. There is however VersionedSchema and SchemaMigrationPlan to give you more fine grained migration control.

---

Q: Are there any limitations in concurrent access from different processes, let's say App and SSO Extension?

A: There are no limitations in the concurrent access as long as the two use the Shared App Group

---

Q: I understand that I need to use context.save to confirm transactions. What is the best practice on how often this can be done? is it very demanding doing soon after each change or it depends on whether we use CloudKit vs local storage?

A: You should batch save transactions according to your requirements, but a save for each change is aggressive. CloudKit has a different event loop for mirroring those changes.

---

Q: Is there a way of viewing the datastore behind SwiftData so as to verify data is being stored as expected? For example, when using the GRDB library with Swift, I can view the SQLite database file in a SQLite client to ensure data is being persisted as expected.

A: The same can be done with your underlying persistent back end with SwiftData

Additional-A: you can view the underlying Core Data (sqlite) file.  Mutating it is not supported

---

Q: As I understand it, SwiftData model objects are not thread-safe, just like NSManagedObjects. Are there any additional mechanisms to make managing this easier for us than it was in traditional Core Data? e.g., compiler warnings before passing an object out of its context?

A: We have provided the ModelActor protocol & the DefaultModelExecutor to make SwiftData work with Swift Concurrency.  Use your ModelContainer to initialize a ModelContext in the initializer for your ModelActor conforming actor object and use that context to initialize a DefaultModelExecutor.  This will allow you to use that context with async functions on your actor.

Additional-A: Swift will enforce sendability requirements

---

Q: When we fetch data, is the whole data then directly read and written to memory? Or is the fetch result a descriptor and data will only be read when it is actually used?

A: FetchDescriptor has an option to set fetchLimit & fetchOffset so you can page the data in chunks.

---

Q: Do SwiftData model objects return faults, just like NSManagedObjects, where only the specifically fetched properties are safe to access?

A: SwiftData supports internal Futures like CoreData, but at different granularities since value type properties can't support side effects.

---

Q: I’m wondering if there’re any plans to support other store types in SwiftData? Is it possible that I could use NSIncrementalStore and NSAtomicStore etc. with SwiftData in the future?

A: Please file a feedback report.

---

Q: Will SwiftData render my app obsolete if I don't migrate?

A: No

---

Q: can a swiftData model be searilized or stored in some format that i can just restore when the user loads the app for the first time?

A: Yes a file can be saved as a Resource in the application bundle to be used as the seed data for the first launch experience.  Much like it was done in Core Data, the same approach can be used in SwiftData for the canned data in the resources.

---

Q: Does SwiftData support CloudKit sharing with ShareLink and Transferable. If not, can we still achieve this by having some sort of CoreData code on the side? If so, are there any sample code for this?

A: SwiftData doesn’t know about the shared CloudKit database. So you will necessarily need to have an instance of NSPersistentCloudKitContainer on the side to manage that.

That said, your SwiftData model can conform to Transferable and assemble a suitable transfer representation for you to identify it in NSPersistentCloudKitContainer.

ShareLink is orthogonal to NSPersistentCloudKitContainer, NSPersistentCloudKitContainer will be invoked at the point that you resolve your transfer item to a CKShareMetadata.

For more information on CKShareTransferRepresentation see this document, which also links a number of pieces of sample code including one that uses NSPersistentCloudKitContainer: https://developer.apple.com/documentation/cloudkit/cksharetransferrepresentation?changes=_1

---

Q: Is it possible to specify how to sort optional properties for nil values using @Query?

A: Please file a feedback report

---

Q: Is SwiftData faulted like CoreData?

A: SwiftData supports internal Futures like CoreData, but at different granularities since value type properties can’t support side effects.

Follow-up-Q: What, exactly, is an internal future. I tried looking it up, with no success

---

Q: With CoreData you can define an index to performance. Does  a similar thing exist for SwiftData?

A: Yes, you can use .index on the @Attribute macro

---

Q: Can the @Query property wrapper handle nested/sub queries? If I need to fetch some subset of Model A as part of my query for Model B what would be the best way to do that?

A: You can write traditional CoreData SUBQUERYs using operations like .filter(_:) in your #Predicate in order to compose complex operations. For example, if your Model A had a one-to-many relationship to Model B you can use filter(_:) in your predicate to get a subset of model B's to perform an operation on in your query.

Additional-A: If that's not quite the type of subquery you're trying to do, feel free to provide an example of your model relationships or an example of the query you're trying to perform and we can help you figure out the best way to write it!

---

Q: CoreData has support for R-Tree indexes. Can you do something similar for SwiftData?

A: Please file a feedback report

---

Q: Will a synced SwiftData be updated when the app is not running? If yes, can we schedule a background task in this event for a timely update of the app state? E.g. sync a timer and use the event to update widgets or schedule user notifications.

A: You should enable the background activity capability in your application, the system may choose to launch your application for networking activities and when that happens you can initialize your ModelContainer to do work with CloudKit.

---

Q: can a @Model contains a property of the same type, in a recursive way? Like a @Model Person having a 'father: Person' and 'children: [Person]'?

A: Yes! Modeled types that refer to each other are manifested in the schema as relationships.

Additional-A: Where the type being the same makes a difference is that you may have to indicate the relationship's inverse explicitly because they won't be something the runtime can infer.

---

Q: Does SwiftData define index in a different way? I couldn't see anything like @index or any other APIs includes index, is it possible to define index in SwiftData?

A: Yes in a future seed!

---

Q: How are dynamic predicates handled with @Query? Is that covered in later sessions?

A: Hi Kevin! If you're referring to dynamically changing the predicate passed to a @Query property, the current recommended approach is to manually instantiate the Query in your view's initializer. For example, this view uses a boolean value passed into the view's initializer to construct the query:

```swift
struct ContentView: View {
    @Query private var favoritePeople: [Person]
    
    init(showFavoritesOnly: Bool) {
        _people = Query(filter: #Predicate { $0.isFavorite || !showFavoritesOnly })
    }
    
    var body: some View {
        List(favoritePeople) { person in
            Text(person.name)
        }
    }
}
```

A view that contains ContentView in this example can drive the showFavoritesOnly parameter in any way it needs to, such as from a @State property or other source.

You can also find this pattern in this year's Backyard Birds sample app, which should be available soon
