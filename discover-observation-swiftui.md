# Discover Observation in SwiftUI

Presenter: Philippe Hausler, Software Engineer

Observation lets you define your models using standard Swift syntax, and have the UI respond to changes in that model. Makes developing with SwiftUI seamless and intuitive.

## What is Observation?

Make SwiftUI respond to changes in your data model. (i.e. invalidate views when Observed dta changes)

Annotate a model with `@Observable` to use:

```swift
@Observable class FoodTruckModel {    
    var orders: [Order] = []
    var donuts = Donut.all
}
```

Uses Swift macros to abstract away all the boilerplate to make this work.

Using the `@Observable` macro expands your types so they can support observation. This lets SwiftUI track access to those properties and observe when the next property *will* change out of that Observation.

### Using Observable in SwiftUI Apps

No SwiftUI property wrapper needed to use this in your app:

```swift
@Observable class FoodTruckModel {    
  var orders: [Order] = []
  var donuts = Donut.all
}

struct DonutMenu: View {
  let model: FoodTruckModel
    
  var body: some View {
    List {
      Section("Donuts") {
        ForEach(model.donuts) { donut in
          Text(donut.name)
        }
        Button("Add new donut") {
          model.addDonut()
        }
      }
    }
  }
}
```

In this case it can detect that the property `donuts` is accessed when executing the body of the donut menu view. When body is executed, SwiftUI tracks all access to properties used from `Observable` types. It then takes that tracking information and uses it to determine when the next change to any of those properties on those specific instances *will* change.

- When a new donut is added, the View redraws
- When a new order is added, the View *doesn't* redraw, because it's the property isn't part of the tracked properties used when executing the body of the view

### SwiftUI Computed Property Tracking

When a computed property in an observable model that is used in a View changes, the UI updates

```swift
@Observable class FoodTruckModel {    
  var orders: [Order] = []
  var donuts = Donut.all  
  var orderCount: Int { orders.count }
}

struct DonutMenu: View {
  let model: FoodTruckModel
    
  var body: some View {
    List {
      Section("Donuts") {
        ForEach(model.donuts) { donut in
          Text(donut.name)
        }
        Button("Add new donut") {
          model.addDonut()
        }
      }
      Section("Orders") {
        LabeledContent("Count", value: "\(model.orderCount)")
      }
    }
  }
}
```

## SwiftUI Property Wrappers

Simplifies the SwiftUI property wrapper story. Now you only need:

- `@State`
- `@Environment`
- `@Bindable`

### @State

When a View needs to have its own state, use `@State`

```swift
struct DonutListView: View {
    var donutList: DonutList
    @State private var donutToAdd: Donut?

    var body: some View {
        List(donutList.donuts) { DonutView(donut: $0) }
        Button("Add Donut") { donutToAdd = Donut() }
            .sheet(item: $donutToAdd) {
                TextField("Name", text: $donutToAdd.name)
                Button("Save") {
                    donutList.donuts.append(donutToAdd)
                    donutToAdd = nil
                }
                Button("Cancel") { donutToAdd = nil }
            }
    }
}
```

### @Environment

Environment lets values be propagated as globally accessible values. Good for sharing things in many places. Observable types here provide updates based on access.

```swift
@Observable class Account {
  var userName: String?
}

struct FoodTruckMenuView : View {
  @Environment(Account.self) var account

  var body: some View {
    if let name = account.userName {
      HStack { Text(name); Button("Log out") { account.logOut() } }
    } else {
      Button("Login") { account.showLogin() }
    }
  }
}
```

### @Bindable

Allow bindings to be created from that type. Use the `$` syntax to get the binding to that property - typically an observable type.

```swift
@Observable class Donut {
  var name: String
}

struct DonutView: View {
  @Bindable var donut: Donut

  var body: some View {
    TextField("Name", text: $donut.name)
  }
}
```

### Which property wrapper to use?

- Does the model needc to be state of the view itself? `@State`
- Does the model need t obe part of the global environment of the app? Use `@Environment`
- Does the model just need bindings? Use the new `@Bindable`

If no to all of the above, use the model as a property of your view.

## Advanced Uses

Because SwiftUI tracks access to fields per instance, it means that you can use arrays, optionals, or any type that contains your observable models. This lets you build your models how you want. You can have arrays of models being observed, or model types that contain other observable model types.

The general rule is for `@Observable`, if a property that is used changes, the view will update.

```swift
@Observable class Donut {
  var name: String
}

struct DonutList: View {
  var donuts: [Donut]
  var body: some View {
    List(donuts) { donut in
      HStack {
        Text(donut.name)
        Spacer()
        Button("Randomize") {
          donut.name = randomName()
        }
      }
    }
  }
}
```

### Manually Observe

Something like a a computed property does not have any stored properties requires two extra steps to make it work with Observation. 

Tell Observation:

- When the property is accessed
- When the property changes. 

This is how Observation synthesizes access to properties normally, except here we've rewritten those custom access points manually so that the non-observable location can be read and store the name.

```swift
@Observable class Donut {
  var name: String {
    get {
      access(keyPath: \.name)
      return someNonObservableLocation.name 
    }
    set {
      withMutation(keyPath: \.name) {
        someNonObservableLocation.name = newValue
      }
    }
  } 
}
```

If a computed property is composed from stored properties, Observation just works. But if that's not the case, you can use this manually.

## ObservableObject

Migrating `ObservableObject` code to Observable

### In Models

ObservableObject version:

```swift
public class FoodTruckModel: ObservableObject {
    @Published public var truck = Truck()

    @Published public var orders: [Order] = []
    @Published public var donuts = Donut.all

    var dailyOrderSummaries: [City.id: [OrderSummary]] = [:]
    var monthlyOrderSummaries: [City.id: [OrderSummary]] = [:]
    ...
}
```

New Observable version:
- Remove conformance to ObservableObject
- Remove the `@Published` annotation
- Add the `@Observable` macro

```swift
@Observable public class FoodTruckModel {
    public var truck = Truck()

    public var orders: [Order] = []
    public var donuts = Donut.all

    var dailyOrderSummaries: [City.id: [OrderSummary]] = [:]
    var monthlyOrderSummaries: [City.id: [OrderSummary]] = [:]
    ...
}
```

### In views

As an ObservableObject:

```swift
struct AccountView: View {
    @ObservedObject var model: FoodTruckModel

    @EnvironmentObject private var accountStore: AccountStore
    @Environment(\.authorizationController) private var authorizationController

    @State private var isSignUpSheetPresented = false
    @State private var isSignOutAlertPresented = false
}
```

New Observable version:

```swift
struct AccountView: View {
    var model: FoodTruckModel

    @Environment(AccountStore.self) private var accountStore: AccountStore
    @Environment(AuthorizationController.self) private var authorizationController

    @State private var isSignUpSheetPresented = false
    @State private var isSignOutAlertPresented = false
}
```

## Related Content

- Write Swift macros
- Expand on Swift macros
