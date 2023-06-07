# Model Your Schema with SwiftData

Presenters: Rishi Verma, SwiftData Engineer

## Declare a model

Add `import SwiftData` and decorate the model class with `@Model`

SwiftData Trip model:

```swift
import SwiftUI
import SwiftData

@Model
final class Trip {
    var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

## Customize schema with @Attribute

You can add the `@Attribute` decorator to customize your schema. In addition to the examples below, you can use `@Attribute` to:

- Store large data externally
- Provide support for Transformable
- Spotlight integration
- Hash modifier

## Require an attribute to be unique

Adding a unique attribute:

```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

Unique constraints support primitive value types:

- Numeric
- String
- UUID
- To-one relationship

## Remap an attribute name

When you don't want to create new properties, remap the original name to the new name:

```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

## Relationships

When I add a new bucket list item or a living accommodation to my trip, SwiftData will implicitly discover the inverses between my models and set them for me. The implicit inverses do not require any annotations. They just work. Implicit inverses use a default delete rule that will nullify the bucket list items and living accommodation properties when a trip is deleted.

With the `@Relationship` macro, you can:

- Set an `.originalName`
- Specify minimum and maximum `toMany` count constraints
- Hash modifier

### Cascading Deletes

If you want a bucket list item and living accommodation to be deleted along with the trip, add the `@Relationship` macro with a cascade delete rule:

```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?
}
```

## Transient properties

When you don't want a property to be persisted, decorate with `@Transient`:

Provide a default value to ensure they have logical values when data is fetched.

```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?

    @Transient
    var tripViews: Int = 0
}
```

## Migration

To evolve your schema:

1. Encapsulate your models at a specific version with `VersionedSchema`
2. Order your versions with `SchemaMigrationPlan`
3. Define each migration stage
   - Lightweight: No additional code required
   - Custom: Define code to execute in the migration

### Defining versioned schemas:

```swift
enum SampleTripsSchemaV1: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV2: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV3: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        @Attribute(originalName: "start_date") var startDate: Date
        @Attribute(originalName: "end_date") var endDate: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}
```

### Implementing a SchemaMigrationPlan: 

```swift
enum SampleTripsMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SampleTripsSchemaV1.self, SampleTripsSchemaV2.self, SampleTripsSchemaV3.self]
    }
    
    static var stages: [MigrationStage] {
        [migrateV1toV2, migrateV2toV3]
    }

    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SampleTripsSchemaV1.self,
        toVersion: SampleTripsSchemaV2.self,
        willMigrate: { context in
            let trips = try? context.fetch(FetchDescriptor<SampleTripsSchemaV1.Trip>())
                      
            // De-duplicate Trip instances here...
                      
            try? context.save() 
        }, didMigrate: nil
    )
  
    static let migrateV2toV3 = MigrationStage.lightweight(
        fromVersion: SampleTripsSchemaV2.self,
        toVersion: SampleTripsSchemaV3.self
    )
}
```

Configuring the migration plan:

```swift
struct TripsApp: App {
    let container = ModelContainer(
        for: Trip.self, 
        migrationPlan: SampleTripsMigrationPlan.self
    )
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

## Related Sessions

- Meet SwiftData
- Build an app with SwiftData
