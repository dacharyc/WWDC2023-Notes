# Meet SwiftData

Presenter: Ben Trumbull, Engineering Manager

SwiftData is a framework for data modeling
Uses macro system to provide "seamless" API experience
Works with other features like CloudKit and Widgets

## Using the model macro

### The @Model decorator

`@Model` is a new Swift macro used to define your schema with code
Adds SwiftData functionality to Model Types (persisted & Observable?)

```swift
import SwiftData

@Model
class Trip {
    var name: String
    var destination: String
    var endDate: Date
    var startDate: Date
 
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

Transforms the class's stored properties and turns them into persisted properties

### Supported property types

- Support for basic value types
- "Complex" value types:
  - Struct
  - Enum
  - Codable
  - Collections of value types

### Relationships

Relationships are inferred from reference types

- Other model types
- Collections of model types

### Property Metadata

You can add decorators to a `@Model` class for control over how properties are inferred:

- `@Attribute`: Add uniqueness constraints
- `@Relationship`: Control the choice of inverses and specify delete propogation rules
- `@Transient`: Don't persist the data in this property to DB

Model annotated with additional decorators:

```swift
@Model
class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var endDate: Date
    var startDate: Date
 
    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

Supports cascading deletes - how deeply? Does it actually delete the related objects or just set the relationship to nil?

## Working with your data

Working with SwiftData means working with a Model Container

- Persistence backend
- You can customize it with configurations
  - Change your URL
  - Change CloudKit identifier
  - Change Group container identifier
- You can pass schema migration options

Default initialization just requires you to specify the list of model types you want stored

```swift
// Initialize with only a schema
let container = try ModelContainer([Trip.self, LivingAccommodation.self])

// Initialize with configurations
let container = try ModelContainer(
    for: [Trip.self, LivingAccommodation.self],
    configurations: ModelConfiguration(url: URL("path"))
)
```

Then, fetch and save data with model contexts

You can use SwiftUI's View and Scene modifiers to set up a container and have it ready in the View's environment

```swift
import SwiftUI

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(
            for: [Trip.self, LivingAccommodation.self]
        )
    }
}
```

### Model Context

Model contexts observe all the changes to your models and provide many of the actions to operate on them

- Tracking updates
- Fetching models
- Saving changes
- Undoing changes

In SwiftUI, access the model context in the view as an environment object

```swift
import SwiftUI

struct ContextView : View {
    @Environment(\.modelContext) private var context
}
```

Outside the view hierarchy, you can ask the ModelContext to give you a shared main actor-bound context/container

```swift
import SwiftData

let context = ModelContext(container)
```

### Fetching data

New Swift-native types:

- `Predicate`
- `FetchDescriptor`

Improvements to `SortDescriptor`

`Predicate` works with native Swift types and uses Swift macros for type safety. Type-checked replacement for `NSPredicate`
Autocompleted keypaths

#### Building a Predicate

```swift
let today = Date()
let tripPredicate = #Predicate<Trip> { 
    $0.destination == "New York" &&
    $0.name.contains("birthday") &&
    $0.startDate > today
}
```

#### Fetching with a FetchDescriptor

let descriptor = FetchDescriptor<Trip>(predicate: tripPredicate)

let trips = try context.fetch(descriptor)

#### Fetching with Fetch and SortDescriptor

`SortDescriptor` updates:

- Updated to support all `Comparable` types
- Swift-native keypaths

```swift
let descriptor = FetchDescriptor<Trip>(
    sortBy: SortDescriptor(\Trip.name),
    predicate: tripPredicate
)

let trips = try context.fetch(descriptor)
```

#### Additional FetchDescriptor Options

- Relationships to prefetch
- Result limits
- Exclude unsaved changes

And more

## Modifying Your Data

Perform basic CRUD operations using the model context:

- Inserting
- Deleting
- Saving
- Changing

```swift
var myTrip = Trip(name: "Birthday Trip", destination: "New York")

// Insert a new trip
context.insert(myTrip)

// Delete an existing trip
context.delete(myTrip)

// Manually save changes to the context
try context.save()
```

## Use SwiftData with SwiftUI

SwiftData was designed for easy integration with SwiftUI

### View Modifiers

- Leverage Scene and View modifiers
- Configure data store with `.modelContainer`
- Changes are propagated throughout SwiftUI environment

### `@Query` in SwiftUI

With the `@Query` property wrapper, you can load and filter query results

- Observes changes - no need for `@Published`
- SwiftUI automatically refreshes

```swift
import SwiftUI

struct ContentView: View  {
    @Query(sort: \.startDate, order: .reverse) var trips: [Trip]
    @Environment(\.modelContext) var modelContext
    
    var body: some View {
       NavigationStack() {
          List {
             ForEach(trips) { trip in 
                 // ...
             }
          }
       }
    }
}
```

## Related Sessions

- Model your schema with SwiftData
- Migrate to SwiftData
