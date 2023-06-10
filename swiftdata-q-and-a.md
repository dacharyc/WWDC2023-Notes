# SwiftData Q&A

## My Questions

- Supports cascading deletes - how deeply? Does it actually delete the related objects or just set the relationship to nil?
  - Q: How many levels deep can a cascading delete - delete? i.e. if a Person has a relationship to Dog, and if Dog has a relationship to Toy, can a cascading delete from Person delete Toy or just Dog?
  - A: As long as the relationship is set up as cascade all the way down, it will continue to cascade.
  
- Supported data types - data?
  - Q: Is binary data a supported data type?
  - A: Yes it is!

## Tuesday June 6 Q&A

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

## Friday June 9 Q&A

---

Q: How does the .transformable property option work? I have a property with type [String: String] and I'm seeing the following warning: Please switch to using "NSSecureUnarchiveFromData" or a subclass of NSSecureUnarchiveFromDataTransformer instead.

A: Please file a feedback item

---

Q: Do you have any advice about how to use Enum as SwiftData attributes, in addition that they must conform to Codable? Can this enum have any (Codable) associated value? It seems to work when my enum contains only one case, but with a second one, my app crashes. I filled FB12279055 about that issue

A: Indeed the associated values should be Codable and be persistable.  If you run into issues please file feedback reports with the various cases you have issues with and we can address those that may not work as expected.

Follow-up-Q: What do you mean by persistable? Is a Codable struct as associated value of an Enum is enough?

Follow-up-A: Yes. I mean that it is eligible to be a persisted property

---

Q: How do you model many to many relationships in SwiftData?

A: You can make the relationship and its inverse both collection types

Follow-up-Q: If you use an Array collection type, are the result ordered?

---

Q: How does SwiftData integrates with CloudKit?

A: To Integrate CloudKit, you can enable "CloudKit" option while creating the project Or For your target, Capability add support for iCloud & CloudKit with a container

Follow-up-Q: I did that, but the records from my public database isn't downloaded. Do I have to write code to fetch from CloudKit?

Follow-up-A: If you want to use Public databases, you will need to coexist with Core Data and use the NSPersistentCloudKitContainer

Follow-up-Q: So SwiftData can't work with Public Databases?

---

Q: I have the following two entities referring each other:

```swift
@Model class Item {
   ...
   @Relationship(inverse: \Category.items) var categories: [Category]?
}
@Model class Category {
   ...
   @Relationship(inverse: \Item.categories) var items: [Item]?
}
```

But in both relationship lines I get the error: "Circular reference resolving attached macro 'Relationship'". What am I doing wrong?
This was automatically generate from the old CoreData scheme by the way.

A: Setting the inverse on only one of the models (or neither, and letting SwiftData infer their association) should get this working. Could you file a feedback report against the Xcode migrator with your model please?

---

Q: How does autosave work with validations? What happens if the timer goes to save the context but there is an invalid object?

A: The pending changes will remain in the context until you call save explicitly, or fix the validation issue and get picked up by the next autosave

---

Q: Is there a way to use SwiftData without automatic iCloud sync? I’d like to do that manually using my own CloudKit solution or CKSyncEngine. SwiftData automatically picks up any CloudKit containers though and I have not seen an option to disable this behavior. Setting cloudKitContainerIdentifier to nil does still pick the first available CloudKit container.

A: This sounds like a bug. Please file a feedback report.

---

Q: If I save a custom ModelContext for a Container, will the other ModelContext’s derived from that Container (both custom and ‘main’) automatically be updated?

A: They will, although there are some known issues in seed 1.

---

Q: Does SwiftData handle the URLSession, CoreData, and persistence?  I guess I’m confused about what all SwiftData can handle.    I look at theLoadingAndDisplayingALargeDataFeed  and do not consider that large data.  I need to handle JSON data from several sites and wondered if SwiftData is able to handle that kind of backend data?

A: For SwiftData, you would need to decode your models and then` insert()` them in the ModelContext , there is no direct`URLSession` support but please file a feedback request with your use case

---

Q: Core Data has added new capabilities this year, such as new ways for handling migrations. Are these new capabilities also all available in SwiftData? (I realise this may be covered in a video—I have not been able to view all of them yet.)

A: Yes. In SwiftData, you’ll use VersionedSchema and SchemaMigrationPlan.

---

Q: This isn't super clear to me; does SwiftData take care of our local persistence store? Does this mean we don't need to use CoreData?

A: You can use either, or both, as makes sense for your app

---

Q: What happens when I have a view observing a model object that is deleted elsewhere? Is there a way to detect the object has been deleted? The trips sample code crashes in this situation with two windows open on an iPad (see FB12272702). One window showing list of trips, other window showing trip detail. After deleting trip from list app crashes when changing now deleted trip in detail view.

A: as a workaround, can you try calling save() after the delete() to see if that resolves the crash?

Follow-up-Q: Adding a save() did not resolve the crash

Follow-up-A: Thanks for the update, we will add that to the feedback report!

Follow-up-Q: But the real question is what should happen in this case? It would be nice to have some way to dismiss the detail view once the object has been deleted

Follow-up-A: In the view that is observing the model, could check if the instance isDeleted()

Follow-up-Q: OK, that could work, Will give that a go as a workaround and update the feedback with results.

---

Q: Do predicates support complex expressions? For example, in Core Data I often use NSExpression to fetch min and max values of a field.

A: I think the answer to this will depend on which expressions you're looking to support, so feel free to clarify or ask in the thread about specific expressions. But predicates with SwiftData do not currently have explicit support for some of the mathematical/statistical values such as min/max - feel free to file a feedback about those if you'd like to see them added!

In particular, filing feedbacks with the content of any #Predicates that don’t seem to be supported is very helpful so that we can see how you’re representing your query in Swift code

---

Q: Does SwiftData allow me to sync to iCloud? How does that process work exactly? What do I need, and where can I find some more resources regarding this?

A: SwiftData supports synchronization with CK private database using the cloudKitContainerIdentifier property on ModelConfiguration or setting it up in your entitlements. If you want to use shared / public databases, you'll need to coexist with Core Data and use the NSPersistentCloudKitContainer container
I'd suggest going through the SwiftData documentation for more details. For ModelConfiguration: https://developer.apple.com/documentation/swiftdata/modelconfiguration/

---

Q: Does SwiftData work with multiple stores? For example, when we want to use CloudKit sharing, the previous recommandation was to use a dedicated store for the private data and another one for the shared data. Both were loaded by the same container, and could use a single NSManagedObjectContext.

A: Yes, you can create a ModelContainer using multiple ModelConfigurations similarly to NSPersistentContainer and store descriptions.

Follow-up-Q: Is there any sample code showing how we can setup a ModelContainer for CloudKit shared models and the shared database, like we did with CoreData?

---

Q: Coexisting scenarios:  The ‘Migrate to Swift Data’ presentation indicates that I need to be sure to used Schema Versioning for using both CoreData and SwiftData, and it refers to the Modeling with Swift Data presentation.  This makes sense from the SwiftData perspective, but I do I need to also make sure to version my CoreData model in exactly the same way?  
In other words, with CoreData alone, I can generally just add a new optional field to a model object without creating a new version, but is this something I can no longer do?

A: That’s correct. If you choose to co-exist the Schema and Core Data model must be kept in sync and be Entity version hash equivalent

---

Q: What's the best way to create sample data with relationships in a SwiftUI preview? It appears that I need to pass a ModelContext and insert records manually before setting relationships but maybe I'm missing something.

A: There are a few options:
1. Manually create an object graph, as you say
2. Use an in-project store file to hold a representative data graph you want to work with in your preview
3. Interact with your previews to configure data in an in-memory store

Follow-up-Q: For option 1, does manually creating the object graph require passing around the ModelContext or is there a way to just treat models as plain old swift objects?

Follow-up-A: Models can exist outside (not bound to) a context, but they are not persistent.

---

Q: How do you prevent the use of CloudKit — e.g., if you want to use CKSyncEngine instead of SwiftData's own sync engine?

A: Simply do not use a CloudKit container identifier when configuring the SwiftData stack. There is currently a bug with this, so please file a feedback report.

---

Q: You explained Deferred migration in the video What’s new in Core Data. In this case, it means that the app will use a version of the store that is not up to date. How can the app handle this? Parts of the app that uses the new schema can’t work properly if the model is not up to date.

A: No, that's not what it means.  After a migration, the current model is always completely represented in the schema and the app can use all its model features.  However clean up, deletions, and dropping old indices, and some other work, is deferred.

---

Q: Is there an equivalent in SwiftData for the CoreData NSPersistentHistory classes, such as NSPersistentHistoryChangeRequest, NSPersistentHistoryTransaction, NSPersistentHistoryToken, and so on, please?

A: Not at this time.  Please file a feedback request.

---

Q: Is it silly to do something like this?
#Predicate { _ in true }
I know you don't need this for FetchDescriptor, but I wonder would there ever be a legitimate use case for this? Maybe if you wanted to have an array pf Predicates that you could choose from? This currently raises a runtime exception:
Exception    NSException *    "Unable to parse the format string \"YES\""

A: You can indeed write a predicate like that to indicate an always-true condition if that helps with your app's logic. The crash that you're seeing is noted in the release notes for the released beta at https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-17-release-notes:
SwiftData queries will not work with #Predicate { true } and #Predicate { false } will cause a crash. (109425342)
Workaround: Use a logically true expression like { 1 == 1 }.

---

Q: Can we create indexes on property(s) in our Models or is this handled automatically for us? Are indexes automatically created on unique properties and properties marked as relationships?

A: You can add indicies on your attributes by using .index with the @Attribute macro

Follow-up-Q: As a follow up, does this mean we can’t have a compound index?

---

Q: If I have a CloudKit container entitlement in my app, can I disable the SwiftData container from using it for previews and unit testing? For example, in Core Data I can delete the CloudKit options in the persistent store description before loading.

A: There are some known issues in seed 1 of SwiftData. Please file a feedback report for this issue and we will take a look!

---

Q: Did I hear it right that inserting an object with a unique constraint will upsert it automatically? What if you have a partially populated model? For instance my API returns a profile with an id and name, but I already have a version of that profile with all of its other properties like bio. With the other properties be wiped out if I try and insert the partial object?

A: Only the properties that were updated will be included in the UPSERT. Please file a feedback report with your sample project if that is not what you experience.

---

Q: I notice retrieving the ModelContext (ModelContainer.mainContext) is @MainActor . However, the model itself is not. Is it only the retrieval that needs to be done in the Main thread, or everything inside?

A: Models and ModelContext are not @sendable, so you can't smuggle them to other actors.  They are bound to the actor you create them upon.

---

Q: I used the ‘Create SwiftData Code’ action in Xcode, and it generated Model objects with bi-direction relationships (which is correct — my Core Data model includes these).  All of these, however, are marked with the error Circular reference resolving attached macro 'Relationship'
I later noticed that the Sample 'Trip' app addresses this by only having the @Relationship annotation on one side of the relationship, but both of the 'child' entities refer back to an Optional Trip, and use a 'Nullifies' cascading strategy.  
Is it possible for these child entities to refer back with a non-Optional field, and for it to use a 'Deletes' cascading strategy?  I'm considering a co-existing scenario, and I want to make sure that my two models can properly be in sync with one another

A: Please file a feedback report for this, it is a bug in code generation. Thank you!

Follow-up-Q: Thanks, will do -- to my second question, does the reference back to the 'root' entity (the one annotated with @Relationship ) need to be optional?

Follow-up-A: Yes I believe so!

---

Q: The idea of having newBackgroundContext seems to be out. I shouldn't care? What if I want to do a background task?

A: Just create one.  ModelContext(mycontainer)

Follow-up-Q: But then, based on mainContext being a @MainActor and that shouldn’t be used concurrently (see https://wwdc23.slack.com/archives/C058Q99P81F/p1686345330166609 ), does that mean my new ModelContext().mainContext cannot do any operation outside the Main actor at the moment?

Follow-up-A: no, and you should receive warnings if you try to send it across Actor boundaries

---

Q: Is there a way for a query to return a single entity, instead of an Array?

A: Not at this time. Please file a FB if you'd like to request a different API.

---

Q: Are compound unique keys supported and if so, how do they work? I have several objects that are unique based on an id and a relationship to a login. When I generate SwiftData models from that, only the id is marked unique. Can relationships be a unique constraint?

A: Please file a feedback request for Compound Unique Constraints.  A To-One Relationship is eligible to be considered for a unique constraint, if that is not the behavior you are experiencing let us know!

---

Q: In my app, I have sensitive data, that I need to encrypt locally on the iPhone. Does SwiftData have a build-in feature, that offers encryption (full database or fields)? And, if so, how can I use it?

A: You should use the built in data protection mechanisms (NSFileProtection) for iOS. This would be accomplished by setting the attribute on the file.

---

Q: Currently with CoreData and CloudKit I have to do a bunch of complicated stuff to track the persistent history using the PersistentHistoryToken so I can deduplicate records since CloudKit doesn't support the unique attribute of CoreData, is there an equivalent process to this with SwiftData?

A: Coexistence is available and for this approach you would continue to use the CoreData stack to process the Persistent History

Follow-up-Q: So the idea would be to do the deduping with coredata in the background and possibly use SwiftData for  driving the UI?

Follow-up-A: That is correct

---

Q: Is there an equivalent of NSManagedObject's isInserted property?

A: Please file a feedback request.   You can ask the ModelContext if its in the collection of inserted objects.

---

Q: Is the @Relationship property wrapper required for any property whose type is a SwiftData @model or is it just needed when you want to specify @Relationship-specific behavior like .cascade?

A: It’s only needed when you have a non-default configuration.

---

Q: CoreData has the ability to hold a context at a specific version of the data, and update to the latest version only on request. Does — or will — SwiftData have something similar? (For many things you want to be working with a stable snapshot over a period of time.)

A: Please file a feedback request with your use case

---

Q: When storing a Date property, does SwiftData also store time zone information for that date such that if the user's device time zone changes, the data in the database still respects the original time zone the date value was stored in?

A: Date objects are always in GMT. Time zones are applied when converting to or from textual or calendrical representations. If you want to store a time zone you can store its identifier separately from the Date.

---

Q: If we sync our SwiftData models through iCloud, will they be end-to-end encrypted automatically if the user has enabled Advanced Data Protection on their account?

A: You’d want to use the .encrypt property on the Attribute macro to encrypt them at rest on the iCloud server.

Follow-up-Q: Follow-up question for this. Does the .encrypt property work with the Shared CloudKit database in Core Data? If so, how does that work if you’re sharing with different iCloud users?

Follow-up-A: It’s for the private database only I believe

---

Q: CoreData doesn't allow unique values if you sync with CloudKit. What will happen if I try to sync a unique value with SwiftData?

A: You’ll receive an error that unique constraints are unsupported in CloudKit, the same as in Core Data.

---

Q: Is it possible to combine multiple ModelConfigurations into a single ModelContainer where one is in memory and one is not? I would love it if you can have a single object graph where some objects are persistent and others are not.

```swift
@Model
final class Person {
  var name: String
}

@Model
final class Post {
  var title: String
  var author: Person?
}

let fullSchema = Schema([Person.self, Post.self])
let people = ModelConfiguration(for: Person.self, inMemory: false)
let posts = ModelConfiguration(for: Post.self, inMemory: true)
let container = ModelContainer(for: fullSchema, configurations: [posts, people])
```

A: Indeed you can setup the ModelContainer with multiple ModelConfigurations, however in this example, your Post and Person are related, and thus will be part of the same Schema in all the configurations.

Follow-up-Q: yeah. So you can’t create relationships between models that span boundaries between stores?

Follow-up-A: That is correct

---

Q: I want to build a collection of SwiftUI apps that share access to the same CoreData (and CloudKit) stack. I want to ensure the CoreData Schema is consistent across the apps, so I plan to break out the CoreData Schema into a swift framework that is shared among the apps. Would I be able to use SwiftData with this use case, and are there any limitations?

A: Yes, you can use Schema’s makeManagedObjectModelMethod method to generate the model for the CoreData stack.
You won't be able to use you Model objects with CoreData though, that work will need to be in NSManagedObject or your existing subclasses.

---

Q: What is the best way to support saving one off information like user preferences with swift data? Does everything need to be a collection of records?

A: Anything saved in SwiftData needs to be modeled, so you can simply create one record and fetch it later.

---

Q: Is loading a SwiftData container async or can it potentially block the main UI thread?

A: There should be no blockage on the mainUI thread, please file a feedback report with a sample project if this a scenario you have experienced.

---

Q: What’s the recommended approach to use SwiftData in UIKit if I previously have used NSFetchedResultsController or should I continue using Core Data for now?

A: Use SwiftData with UIKit can be used with the ModelContext's .fetch() and tailoring the results with a FetchDescriptor

---

Q: It looks like SwiftData uses arrays instead of sets, but when I export my models I get a warning that ordered relationships aren’t supported. Are the relationships still unordered and unique like they would be with a set then?

A: Please file a feedback report for the issue.

---

Q: What is the best way to import bulk data (say thousands of network interfaces)?
Current thinking is to create a background process under 'ModelContextProvider', create a context pointing to the entities then insert/update each entity we return via external API.
Previously, using CoreData, we utilised the BatchInsertRequest functions

A: Please file a feedback report for batch support with some sample data if possible.

...some chatter from other non-Apple-devs...

Follow-up-Q: Are you saying we can use SwiftData for UI operations while managing batchInserts via CoreData APIs?
All in the same SQLite instance?

Follow-up-A: Yes because Core Data and SwiftData can share the same store. You’d just have to fault in the updated objects into SwiftData if you updated the store from Core Data.

Follow-up-A: Looks like we can batchInsert from SwiftData (pulled from another question):
ModelContext insert(_ batch: [[String: Any]], model: any PersistentModel.Type)

---

Q: What is the easiest way to blow away all data in dev when using the modelContainer(for:) modifier? I have been using an onAppear block where I call modelContext.container.destroy() which works but crashes the app.

A: Setup the container in your application's initializer and then pass that container to the modelContainer modifier

---

Q: Does SwiftData have the same restrictions as Core Data when concerning CloudKit syncing? Specifically not being able to declare ordered relationships (e.g. an array @Relationship property of related items now) and empty strings?

A: The same restrictions apply, please file a feedback report with your use case if these restrictions make your development cumbersome.

---

Q: When syncing with CloudKit how do I dedupe data? Can I still register for NSPersistentStoreRemoteChange notifications and fetch the history as with Core Data?

A: Indeed you can by setting up a parallel stack with Core Data that allow you consume Persistent History and do that processing.

---

Q: I have my own coredata/cloudkit sync engine, and use coredata everywhere.  Can I migrate to SwiftData but configure its stack such that my sync engines still works?

A: Yes. Pass nil for CloudKit container identifier. There is currently a bug for this so file a feedback report.

---

Q: In one of the SwiftData videos, the code showed a container configured with a CloudKit container identifier. But in an earlier Q&A period, someone asked if SwiftData can be used with CloudKit public, private, and shared containers and the answer was that you would need to use NSPersistentCloudKitContainer and CoreData co-existing with SwiftData.
What CloudKit things does SwiftData support, and what doesn't it support? And is there a document that explains this? Thank you.

A: ModelContainer supports the private database but the public and shared database require Co-Existence currently. File a feedback report with your request.

---

Q: My models are both @Model and Codable. Alas, I seem to have an issue with this, where I get ambiguous use of getValue and setValue. Known? Known workaround?

A: Please file a feedback report, you can try making your @Model not Codable to see if that is a feasible workaround.

---

Q: Can a query return data grouped into sections (like NSFetchedResultsController)?

A: Please file a feedback report for this enhancement!

---

Q: I cannot display the preview in SwiftDataFlashCareSample. The error stack includes the following from previewContainer while creating sample data:
“Compiling failed: main actor-isolated let 'previewContainer' can not be referenced from a non-isolated context”
I am running Xcode 15 on MacOS 13.3.1

A: That is unexpected, please file a feedback report with your configuration included!

---

Q: Is it possible to use a transient property in a predicate?

A: Using a transient property inside of a predicate in a SwiftData Query is unsupported since the value of the property is not present in the database backing store for the query to access

---

Q: Is there batch insertion support from a background thread for SwiftData like there is for CoreData in the "Loading And Displaying A Large Data Feed" (Earthquakes) example?

A: Indeed there is! Checkout ModelContext insert(_ batch: [[String: Any]], model: any PersistentModel.Type)

---

Q: SectionedFetchResults along with @SectionedFetchRequest are very important to all of my Apps.  Does SwiftData currently have this function? If not, will it be added in a future Beta? Or, do we have to hack a workaround ourselves using FetchResultsCollection.SubSequence?

A: Sectioned results are not supported currently. Please file a FB request and note it in this thread.

---

Q: My CoreData Model has a To Many Relationship Named "children" and an Derived Attribute "numChildren" = "children.@count" what is the best approach to recreate this in SwiftData?
Love the SwiftData class generation even with the:
#warning("The property \"derived\" on Tag:numChildren is unsupported in SwiftData.")

A: You could update the numChildren property in the children property's didSet!

---

Q: Is there a way to add Codable support to a SwiftData object?
I’m currently facing an issue in relationships when trying to add the conformance either directly or with an extension:

```swift
// Getter error:
ambiguous use of 'getValue(for:)' in expansion of macro 'PersistedProperty'

public func getValue<Value>(for key: KeyPath<Self, Value>) -> Value where Value : Decodable
public func getValue<Value, OtherModel>(for key: KeyPath<Self, Value>) -> Value where Value : Sequence, OtherModel : PersistentModel, OtherModel == Value.Element

// Setter error:
ambiguous use of 'setValue(for:to:)' in expansion of macro 'PersistedProperty'

public func setValue<Value>(for key: KeyPath<Self, Value>, to newValue: Value) where Value : Encodable
public func setValue<Value, OtherModel>(for key: KeyPath<Self, Value>, to newValue: Value) where Value : Sequence, OtherModel : PersistentModel, OtherModel == Value.Element
```

A: This sounds like a good feedback report, please send us one! In the mean time, can you try removing the Codable conformance from the @Model and moving it to an extension?

Follow-up-Q: Tried that and got the same result.. :sweat_smile: (the extension part)

---

Q: I see you can create a Predicate to query Composite Attributes for an entity. For performance reasons, would it be quicker to store the property you want to query outside the composite attribute on the entity itself? Or is performance of querying a property on a composite attribute just as good?

A: If you encounter performance problems using composite attributes, please file a FB request.

---

Q: I was wondering about inheritance in SwiftData. Unlike Core Data where entities can have super types, is there a similar structure for this where for example you can do a single fetch for all entities conforming to a protocol? (PictureMessage, TextMessage, etc. — sorted by date, etc.)

A: Please file a feedback report with your use case!

---

Q: Does SwiftData's @Query need to be used on a View, or can it be used on a view model or anywhere? I seem to have trouble using it Anywhere but a view, but that seems to break my separation of concerns between my swiftUI view, viewModel, etc.

A: You can use ModelContext and its various fetch() functions anywhere.

---

Q: In CoreData, we get the concept of, and can create, child contexts for managing new additions to the store. Is there any equivalent concept or ability in SwiftData?

A: Please file a feedback request.  You can make new peer contexts, but not nest them at this time.

---

Q: When coexisting CoreData and SwiftData, what are the best practices, those beyond keeping the models perfectly synced? One other area is naming conflicts. Any other issues or best practices?

A: This video goes into more depth for best practices for coexistence: https://developer.apple.com/videos/play/wwdc2023/10189
In addition to the ones you've mentioned, you would also need to keep track of the schema versions (this video might help: https://developer.apple.com/videos/play/wwdc2023/10195) . Additionally, make sure you don't forget to turn on persistent history tracking for Core Data.

---

Q: When should I be specifying the inverse key path for relationships (and when not)? I noticed that if I do it on both sides I get a circular reference compiler error, unlike in the previous Core Data editor where the inverse should always be provided.

A: SwiftData will do implicit inverses for you.  If you would like to specify explicit inverses, then set it on only 1 side of the relationship.

---

Q: Does SwiftData work with CloudKit?
What would need to be done to this code to make it sync via CloudKit?

```swift
@Model
struct Person {
   var name = "Bob"
}
```

Does the syncing need to be done by hand or is there a more "automatic" approach? Thank you!

A: Yes, you can pass a CloudKit container identifier to ModelConfiguration to start syncing. You’ll need to make sure your model is compatible with CloudKit as well as following some steps to create a container in CloudKit.

---

Q: NSManagedObject provided a publisher(for: …) api for observing properties internally within a model’s implementation. What’s the recommended approach for observing and responding to property changes within a @Model instance?

A: The @Model conforms to Observable and can be observed via the API provided there.

Follow-up-Q: Which API? Specifically for observing changes to explicit properties

Follow-up-A: That is done by Observable

---

Q: In  what cases is in necessary to call context.instert() explicitally.
i wast adding lots of things at once yesterday to my database (not from inside a view) and then when i went to query i got errors. but after calling save the problem was resolved

A: You need to call insert() explicitly for all new Model instances, except if you relate them to a Model that has already been inserted, in which case we'll do that for you

Follow-up-Q: ok. im not sure i totally get that to be honest but there is no danger in just always calling save right?

Follow-up-A: Models need to be inserted in order for them to be saved / persisted.

Follow-up-A: But relating a Model to an existing previous Model will automatically insert it into the same context s the pre-existing object

---

Q: When using NSPersistentCloudKitContainer to sync a public database, will my app see updates record-by-record or all at once?
For example, 200MB of public records are syncing. Will my app see these records incrementally or only after all 200MB have synced?

A: Records are imported as needed, in batches.

Follow-up-Q: So that's the latter, then? Only after all 200MB have synced do I get access to them in my app?

---

Q: For to-many relationships, can I use any type of collection or does the type need to be an Array?

A: Just Array for now!

---

Q: Can we control SwiftData's automatic saving? That is, do we have the ability to turn it off per view, and call save when we are ready, as opposed to the automatic save?

A: You can turn it off per ModelContext and call save explicitly

Follow-up-Q: So, essentially, we would spin up a ModelContext for that view, turn off automatic saving, and the control it from there?

---

Q: To set a default, unique value in a NSManagedObject such as assigning a UUID to an object's uuid property, this would be done in awakeFromInsert. In SwiftData I can instead just assign UUID() to the property as a default value in the initializer, like I would do with any other Swift class?

A: Yes, you can simply provide a default value:

```swift
@Model
class Sample {
    @Attribute(.unique)
    var id: UUID

    init(id: UUID = UUID()) {
      self.id = id
     }
}
```

---

Q: Not sure if this is better asked in the CloudKit Q&A but figure I’d ask here.
Using SwiftData with the cloudKitContainerIdentifier, I can persist data across devices.
In my app, I plan on using SharePlay for group sessions. I want to include persistent sessions, & shared sessions. A la, similar to Freeform.
I need a way of storing a “session” & their respective members. Once a session ends, they can edit it after the fact, even when solo-editing, which then would propagate those changes to the group, even if they’re offline.
I imagine I’ll need a CKSyncEngine  &/or NSPersistentCloudKitContainer?
How can I keep certain sessions exclusive to certain users, within a database?

A: ModelContainer manages data in the private CloudKit database. To use the shared database you need an instance of NSPersistentCloudKitContainer (or a CKSyncEngine if you want to roll your own sync) on the side.
The best approach for Sharing is to use a ModelContainer with both store files in a non-CloudKit syncing configuration (file a feedback request for this, feedbackassistant.apple.com). Then you use NSPersistentCloudKitContainer to manage the sync for both databases.
The confligration of NSPersistentCloudKitContainer, CKShare, and CKShareParticipant manage data availability and identity for different collections of participants. During a SharePlay activity data can be synced to / from the share, and the participants can remain active on the share after the SharePlay activity ends. NSPersistentCloudKitContainer will continue to sync activity to and from the share for as long as the participant is a member of it.
One thing to note is that if a group of collaborators disband, only the owner of the Share will keep a copy of the data. For that reason, and depending on your user experience and privacy goals, you may wish to copy the data to participants if they choose to leave the share before deleting it.
For more information please consult these documents and sessions:
https://developer.apple.com/documentation/cloudkit/shared_records?language=objc
https://developer.apple.com/videos/play/wwdc2021/10015/
https://developer.apple.com/videos/play/tech-talks/10874/
https://developer.apple.com/videos/play/wwdc2021/10086/
https://developer.apple.com/documentation/cloudkit/shared_records/sharing_cloudkit_data_with_other_icloud_users?language=objc
https://developer.apple.com/documentation/cloudkit/ckshare?language=objc

---

Q: Is there way to model a Color/UIColor in SwiftData like with Transformable in Core Data? Is storing the RGB values separately in a dictionary a good idea?

A: I would recommend using Composite Attributes. In Core Data, they’ll be modeled as dictionary coding and in SwiftData you can store them natively as long as it conforms to codable.

Follow-up-Q: And Color is still not Codable

---

Q: Does SwiftData's @Query need to be used on a View, or can it be used on a view model or anywhere? I seem to have trouble using it Anywhere but a view, but that seems to break my separation of concerns between my swiftUI view, viewModel, etc.

A: You don't need to use @Query; you can use ModelContext and its fetch() functions anywhere.

---

Q: A common scenario is to have Swift model classes with enum attributes. For simplicity, assume it's a Swift enum of the type Int and no associated value.
How should we model such a NSManagedObject Swift class so that it's as easy as possible to migrate to Swift Data when we can require iOS 17? What should the Swift class look like? And what should the Core Data model look like? I did not see any example if this in the WWDC sample code.

A: In SwiftData, have your enums confirm to codable and model them normally. In Core Data, use Composite Attributes to model the same thing

Follow-up-Q: Is Composite Attributes available prior to iOS 17?

Follow-up-A: They are only in iOS 17 and macOS Sonoma

---

Q: Let’s say I migrate my Core Data app to SwiftData. I have 10 versions of the store, with a mix of lightweight migrations and custom migrations. Do I have to write the stages for all the previous versions of the app?

A: For SwiftData, it would only need the VersionedSchema for the last release to migrate the last Core Data release.

Follow-up-Q: If the app is on customer devices with an old Core Data version I think you’ll still need all of the migrations to get them up to the current version?

Follow-up-A: Yes that is correct, and the Managed Object Model Editor has a convenient Generated SwiftData Classes to facilitate the process!

---

Q: In my app I have existing Core Data migration code for between previous versions, and a setup that manually manages the migration stepwise through a series of versions until the first version where NSPersistentCloudKitContainer was introduced, after which only lightweight migration is used. What advice do you have on whether I could at some point move to only/primarily using SwiftData in the app, while maintaining the possibility of a user with a very old version migrating forward to the current one?

A: Checkout this awesome sample that shows a Core Data app that evolves to SwiftData while coexisting between the two on the way!
https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app

---

Q: Do you have best practices/recommendations around using identifiers for SwiftData models?  I typically don't use the objectID in my CoreData models, and instead, have a UUID field in my entities.  Do you have an opinion on this practice?

A: You can have an identifier var in your model with type UUID.

Follow-up-Q: Is there a separate objectID that we can access in SwiftData, or do the objects need to be identifiable themselves?

Follow-up-A: They need to be identifiable by themselves. There isn’t an objectID

Follow-up-Q: My models seem to have a generated objectID field of type PersistentIdentifier

---

Q: When migrating my model using the automated system, it created relationships and their inverse for everything, which is cool. But I seem to be able to create relationships only one side, or else it tells there's circular references. Anything I'm missing, or it's really meant to be single-sided?

A: Indeed like keypaths in Property Wrappers, they are meant to be annotated only on one side.

---

Q: Is there an equivalent to Core Data’s NSFetchedResultsController in SwiftData, for use in situations like a view model where you'd want updates to data as it changes, but outside the context of a view?

A: Please file a FB request and post the number in this thread.

---

Q: I was also wondering if there was an equivalent in SwiftData for implementing something like triggers. For example, if a property is updated on one kind of object, update a related object? (In Core Data, you could intercept “did set key path” or something along those lines) … I realize you could perhaps observe objects but I wasn’t sure if there was a way to observe any of a kind

A: Please file a feedback report for this enhancement with your use case with a SwiftData stack.

---

Q: Is there a SwiftData alternative to SectionedFetchRequest?

A: Not at present. Please file a FB request and post the number in this thread.

---

Q: Let’s say I start using SwiftData. I release an app with a first version of the schema. I then add new features that require to modify the schema to a v2. Do I need to specify a SchemaMigrationPlan in the ModelContainer or is it optional if the migration is lightweight?

A: It is optional but a best practice to capture your releases incase you encounter a future migration that is not lightweight.

---

Q: Do we get access to a time stamp for when data is written to the store? Or do we need to handle this via properties on our model?

A: This is best handled with properties on your own model type(s)!

Follow-up-Q: Do we know when data is about to be written to the store such that we can say “Update this property now before the write!” - I am concerned that setting a value on my model might mean the date/time is significantly different to the actual date/time the store writes the data?

Follow-up-A: What are you doing where the difference between "time modified" and "time persisted" makes such a difference? Autosave triggers with the runloop so the window will be quite small when it's enabled, but you also can force the timing by calling save() manually

---

Q: How do you segregate private and public objects using SwiftData?  Thank you

A: What does public and private mean in this context?

Follow-up-Q: In core data we can create public and private stores.   So suppose we create a quiz app,  the quiz questions would be public, the each users scores are private

Follow-up-A: With NSPersistentCloudKitContainer? You mean store files using CKDatabaseScopePrivate and CKDatabaseScopePublic?

Follow-up-Q: Yes. Thank you

Follow-up-A: ModelContainer doesn’t know about anything but the private scope. You can file a feedback request for public support using feedbackassistant.apple.com and provide the Feedback ID in this thread.
But today you do this with an NSPersistentCloudKitContainer on the side to manage syncing.

---

Q: Is there a way to update the sortOrder or predicate of a query at run time similar to how @FetchRequest works?

A: Not at this time, please file a Feedback report and we'll look into it!

---

Q: If i have a huge set of static data thats not going to change is this something i should use with swift data or should i just consider loading it as a json file.
i have like 100k plus words with definitions that i want to ship with the app.

A: Yes you’d benefit from using SwiftData. I would recommend shipping a canned store file in your apps bundle and load it readonly.

---

Q: How do you create fetch indexes in SwiftData?

A: File a feedback using feedbackassistant.apple.com and provide the Feedback ID in this thread!

---

Q: Lacking abstract entities in SwiftData, what's the best way to model a relationship to a choice of types (that would have been subclasses of the abstract entity)?  Do I need to model a separate relationship for each of the choices?

A: Please file a feedback report with your use case for abstract entities in SwiftData!

---

Q: When trying to build another hobby project I notice an error at launch: Fatal error: Unable to have an Attribute named description
There is something called an ESOAttribute, which is a Codable enum. Should I file a bug report for this, I am unsure what the error means :')

A: A property cannot be named description with SwiftData

---

Q: Looking at the trips sample app, I don’t see a configuration for the persistence container to point to an App Group container, but the app has the App Groups entitlement. Are App Groups still necessary for accessing persisted data within widgets, and is SwiftData shared across app extensions?

A: Indeed the default experience will use the App Group entitlement if it is available.

---

Q: When we add an object as a relationship to another object, we don’t have a insert the object in the context. But when I want to delete an item that is a relationship to another one, do I have to delete it from the modelContext or removing it from the relationship is enough? It seems the API is not doin the same. See here https://twitter.com/dimillian/status/1666728022068658176?s=46&t=-0NC4ZS-XDh_wH7LgEoSZA

A: That code appears to remove the objects from the Array, not the SwiftData stack. Either the relationship needs to be declared with cascade delete or the objects need to be deleted from the context directly.

---

Q: You introduced combined attributes in Core Data (also used by Codable Enum/Struct in SwiftData). Can you give examples of where these combined attributes are better than having a dedicated entity? Why should we use one or the other?

A: In Objective-C, the composite attributes can be better than using a Transformable to flatten them into an opaque blob.  It is situationally dependent on your workflow which approach helps more.

---

Q: SwiftData does not support abstract entities. My app has an entity hierarchy, largely to facilitate data syncing which needs to query all of the entities to find the ones that have changed since last sync. Presently, that's easy to do by just getting a CoreData NSFetchRequest on the base entity and performing one query. Without an entity hierarchy, with SwiftData, it would appear I would have to make and collect the results from one query per entity type I have. Is that what I need to do, or is there a better way?

A: Please file a feedback request

---

Q: What would be the best way to replicate the child context from CoreData using SwiftData?

A: Current Child Context are not supported but you can have peer context. Please file a feedback assistance report

---

Q: Is there a way to subscribe to entity changes from within my app, so that another view can receive a notification for add, delete or update to an entity?

A: Please file a feedback item

---

Q: Can you create a background context in SwiftData for potentially processing things on a background thread?

A: Please see "Concurrency Support" in the documentation for SwiftData at developer.apple.com

---

Q: Is it possible to fallback from a query to a SwiftData DB to a network request (in the case of the data not being found)?

A: it is not possible, please file a feedback request and use case!

---

Q: I have some Core Data NSManagedObjects that have custom logic to prune themselves when they are no longer needed. For instance, an Organization entity will auto-prune itself if the last Department that is associated with that Organization is deleted, but not if any other Departments still exist. This is triggered by setting a flag in  the removeFromRelationship on the Department side and the willSave on the Organization side at present. Is this a pattern that could be represented in SwiftData?

A: It sounds like you could be using the Cascade delete rule

Follow-up-Q: Unless I’ve misunderstood it, the Cascade delete rule would delete the Organization when any Department was deleted, even if other Departments still had a relationship with it. I want to only delete an Organization when no Department has a relationship with it any longer.

Have I misunderstood the Cascade documentation?

---

Q: If I have these models:

```swift
@Model class One {
    var two: Two
}

@Model class Two {
    var three: Three
}

@Model class Three {
    var name: String
}
```

when passing a list of models to be persisted to the .modelContainer modifier, can I just use [One.self] and Two and Three will be inferred automatically?

A: That is correct!

---

Q: I'd be interested in the story of testing in regards to SwiftData. Lets consider I have some service that loads data from SwiftData and I'd like to make sure that it loads the correct data in my tests. Can I just create a SwiftData Model inside my tests that I can compare to the data the service loaded from disk and the test would pass if all fields would be the same or would the test fail because equality checks would also compare some persistent identifier as well, which would be nil in case I've created the object from scratch without saving it in the database.

A: Generally for model layer work, we recommend just testing actual Models to either a temp file or an in memory container.  So, yes, just use the same SwiftData APIs like the real app code.

---

Q: How can we fetch an object from another (background or main) context back to the  default main context ?

A: You can pass the object's PersistentIdentifier along and use fetchIdentifiers to manifest it!

Follow-up-Q: Can we always identify the object with PersistentIdentifier

Follow-up-A: With one caveat; the identifier changes from a temporary to permanent id when the object is first save()d to disk.

But… in this case the object wouldn't be accessible in another context until it had been saved anyway

Follow-up-Q: So, essentially the same system as in CoreData.

Follow-up-A: No coincidence, that :)
