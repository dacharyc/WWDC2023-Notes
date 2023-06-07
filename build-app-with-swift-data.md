# Build an App with SwiftData

Presenter: Julia Vashchenko, SwiftUI Engineer

Build a flash cards app that works across macOS, iOS, watchOS, and tvOS

This is a code-along, so download the prepared project in Resources

Leverage embedded interactive previews in Xcode

## Meet the app

Flash cards app. Select a card and it "flips over" and shows you the answer. At start, project does not persist data.

**PREVIEW WON'T LOAD**
- Default project is set to build for macOS, but I'm not running macOS 14. Had to change the Preview target.
- Preview loads in the Start directory/scheme but not the End directory/scheme. Error related to actor confinement:

```clang
CompileDylibError: Failed to build ContentView.swift

Compiling failed: main actor-isolated let 'previewContainer' can not be referenced from a non-isolated context
```

Switching to use the old-style Preview syntax resolves this error and lets the preview run:

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
        #if os(macOS)
            .frame(minWidth: 500, minHeight: 500)
        #endif
            .modelContainer(previewContainer)
    }
}
```

## SwiftData Models

In the Model/Card file:

- Import SwiftData
- Add the `@Model` macro to the class declaration
- The `ObservableObject` conformance now causes the `@Published` properties to throw errors. This is because `@Model` is automatically `@Observable`. So we have to remove `ObservableObject` conformance and the `@Published` property wrappers.

```swift
@Model
final class Card {
    var front: String
    var back: String
    var creationDate: Date

    init(front: String, back: String, creationDate: Date = .now) {
        self.front = front
        self.back = back
        self.creationDate = creationDate
    }
}
```

Now the CardEditorView throws an error because we were using `@ObservedObject` to watch for changes to the Card. Now, we can change that to `@Bindable` 

```swift
struct CardEditorView: View {
    @Bindable var card: Card
    @FocusState private var focusedField: FocusedField?

    ...
}
```

## Querying models to display in UI

In the ContentView:

- Import SwiftData
- Replace `@State` property wrapper for the `cards` variable with `@Query`

@Query:
- Provides the view with data
- Triggers view update on every change of the models
- A view can have multiple @Query properties

"Lightweight syntax:

```swift
@Query(sort: \.created) private var cards: [Card]
```

### Model Container

Need to provide a model context/ModelContainer

`modelContainer` View Modifier

```swift
.modelContainer(for: Card.self)
```

To use SwiftData, the app must set up at least one ModelContainer. It creates the storage stack, including the context. A View has a single model container.

Update the SwiftDataFlashCardSample file with a ModelContainer:

```swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Card.self)
    }
}
```

Some apps need multiple model containers. Can add different model containers for different windows:

```swift
@main
struct FlashCardApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Card.self)

        Window("Card Designer") {
            CardDesignerView()
        }
        .modelContainer(for: Design.self)
    }
}
```

SwiftUI allows for a granular setup on a view:

```swift
struct CardDesignInspector: App {
    var body: some View {
        VSplitView {
            Form { ... }
            ObjectLibrary()
                .modelContainer(for: Template.self)
        }
    }
}
```

You can add sample data to the preview:

- Open Support/PreviewSampleData
- Open the File Inspector, and click the `Target Membership` checkbox to add it to the project's target membership

Back in the ContentView file, add the container to the Preview:

```swift
ContentView()
    #if os(macOS)
        .frame(minWidth: 500, minHeight: 500)
    #endif
        .modelContainer(previewContainer)
```

NOTE: This crashed for me again with the actor-confinement error listed above. Replacing the `#Preview` macro with the old-style Preview declaration syntax successfully builds this for me:

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
        #if os(macOS)
            .frame(minWidth: 500, minHeight: 500)
        #endif
            .modelContainer(previewContainer)
    }
}
```

## Creating and updating

To perform CRUD operations, you need to access the model context. SwiftUI provides a new `modelContext environment variable:

```swift
@Environment(\.modelContext) private var modelContext
```

This provides access to the model context. A View has a single model context.

Back in the ContentView, add the model context as an environment variable:

```swift
struct ContentView: View {
    @Query private var cards: [Card]
    @State private var editing = false
    @State private var navigationPath: [Card] = []
    @Environment(\.modelContext) private var modelContext
    ...
}
```

Set up the code to save the card to the model context by adding `modelContext.insert(newCard)` after you initialize the new Card object:

```swift
struct ContentView: View {
    @Query private var cards: [Card]
    @State private var editing = false
    @State private var navigationPath: [Card] = []
    @Environment(\.modelContext) private var modelContext

    var body: some View {
        NavigationStack(path: $navigationPath) {
            CardGallery(cards: cards, editing: $editing) { card in
                withAnimation { navigationPath.append(card) }
            } addCard: {
                let newCard = Card(front: "Sample Front", back: "Sample Back")
                modelContext.insert(newCard)
                withAnimation {
                    navigationPath.append(newCard)
                    editing = true
                }
            }
            .padding()
            .toolbar { EditorToolbar(isEnabled: false, editing: $editing) }
        }
    }
}
```

SwiftData saves the card object to the persistent store without you needing to do anything else

If you need to create data without View updates, such as before sharing SwiftData storage, call `save()` explicitly.

Now you can run the app and add cards

## Bonus: Document-based apps

You can add code to the app to share flash cards as SwiftData-backed document apps. 

```swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        #if os(iOS) || os(macOS)
        DocumentGroup(editing: Card.self, contentType: <#UTType#>) {
            <#code#>
        }
        #else
        WindowGroup {
            ContentView()
                .modelContainer(for: Card.self)
        }
        #endif
    }
}
```

### Content Type

SwiftData document-based apps need to declare content types

Documents - content file can be a binary data, package document, etc.

Declaring the content type lets the OS associate the content type with our app

Add the definition to the Info.plist

Specify the file extension. Optionally, add a description. Specify the content type.

```swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        #if os(iOS) || os(macOS)
        DocumentGroup(editing: Card.self, contentType: .flashCards) {
            ContentView()
        }
        #else
        WindowGroup {
            ContentView()
                .modelContainer(for: Card.self)
        }
        #endif
    }
}
```

The App opens with the Open panel - standard behavior for document-based apps. You can create and save stuff.

## Related Sessions

- Meet SwiftData
- Discover Observation with SwiftUI

## Resources

- [Forum - tag wwdc2023-10154](https://developer.apple.com/forums/tags/wwdc2023-10154)
- [Building a document-based app using SwiftData](https://developer.apple.com/documentation/SwiftUI/Building-a-document-based-app-using-SwiftData)