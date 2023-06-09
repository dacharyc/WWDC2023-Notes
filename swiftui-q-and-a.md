# SwiftUI Q&A

Q: In iOS 16, we could use @StateObject to declare data that a View owns, while @ObservableObject was for data injected into a View that the View didn't own, and it was pretty easy to create bindings to a property on either a @StateObject or an @ObservableObject.

Now, @State takes over the role of @StateObject, but @Bindable is what's used if you want to be able to bind to a property on an object. So if a View owns a reference-type source of data (like a view model), but you need to be able to create bindings to that view model's properties, should you use @State or @Bindable?

A: You can can get a Binding from @State. @Bindable is for all the cases where you need to get a binding, but don’t need to use @State.

---

Q: Unlike @Observable available starting from iOS17, @State attribute is not new, but it didn’t use to handle reference types before. If I target iOS 16 (or earlier), should I continue using @StateObject, or should I start using @State even while using the old ObservableObject protocol and @Published properties?

A: Yes, continue to use @StateObject for these use cases if you're deploying to earlier operating systems.

Follow-up-Q: If I accidentally use @State on a class that does not have the Observable macro (ie before iOS 17) is there a warning at compile/runtime?

Follow-up-A: There is not.

---

Q: Is the new operator .scrollPosition(id:) intended to actually scroll to a new position? In other words, is ScrollViewReader deprecated?

A: You can use the scrollPosition(id:) modifier to accomplish similar kinds of things you can do with the ScrollViewReader API and it does additional things that ScrollViewReader can't do. But you might still want to use a ScrollViewReader for scrolling to views in List and Table. Or if you want more fine grain control over scrolling to nested views within non-lazy stacks.

---

Q: Is generally an anti-pattern to use GeometryReader in SwiftUI layouts?

A: It's worth considering alternatives due to its costs and how it interacts with the view hierarchy, for example GeometryReader takes up all available space. Often there's a more idiomatic way to achieve the results that you want, and the Layout protocol has been added recently https://developer.apple.com/documentation/swiftui/layout

---

Q: From looking at the expanded @Observable macro, it looks like Observation tracks “what is used” but not “who used” (I see nothing about the caller, I expected something like _enclosingInstance…).
Yet when I have 2 views, each using a different var from the same class, only the relevant view is updated (I’m using _printChanges to check).
Does the @Observable class know not to notify the other view of the change? Or does SwiftUI know not to diff the other view? Something else?
How does it work or what am I missing here?

A: To give a concrete example: if you have a view A that reads the model.x stored property and view B that reads model.y from the same model instance. If you write only model.y only B would be run again its body.
Say that A is also reading model.z that is computed property from model.y. If you write .y both A and B will run their body.
Every individual view keeps track of the properties that is reading and invalidate only when those are changing.

---

Q: There doesn't seem to be a good swiping gesture solution for List or ScrollView in swiftUI. Either swiping left and right in List doesn't customize the background of the button, etc., or swiping left and right in scrollview using DragGesture conflicts with scrollview gestures, such as stuck and delayed. How is it possible in the Diary app for iOS 17 to swipe to the right to show the Favorites button, and the swipe gesture does not conflict with the vertical gesture for List or scrollview? Or is it implemented in UIKit? Is it also necessary to face and solve these gesture problems in spatial-computing?

A: Hi! You could try use the .swipeActions modifier on the row view to add buttons and customize the background appearance of the button using .tint. Is there a specific goal you want to achieve here?

---

Q: Is there anything built into SwiftUI + SwiftData that lets the user reorder items in LazyVGrid and persist that ordering?

A: You could store information in your model about the ordering to persist it. LazyVGrid doesn't have built in support for reordering.

---

Q: When you access an @Observable object via the @Environment property wrapper, is it possible to create bindings to its properties?
Using @Bindable doesn't work here, right?
Example:

```swift
@Observable @MainActor final class MyObservableModel {
    var name: String = ""
}

struct ContentView: View {
    @Environment(MyObservableModel.self) var model
    
    var body: some View {
        TextField("Name", text: $model.name) // Does not compile
    }
}
```

A: You can derived a @Bindable from your view body in this case

```swift
    var body: some View {
        @Bindable var model = model // add this line
        TextField("Name", text: $model.name) // works!
    }
```

---

Q: Hi, I wanted to ask how to change the background colour of Button on macOS. The .tint modifier not work (FB12215240)

A: Hi Jan! Thanks for your question. You can use tint modifier with buttonStyle(.borderedProminent) to customize the background color.

Follow-up-Q: Hi Jason! I wish you were right but that's not the case. Even the Apple code in documentation for Button tint does not work. See attached screenshot and FB12215240

Follow-up-A: Have you had a chance to try the macOS Sonoma beta? It looks like a bug that's been fixed in that version. And I appreciate the feedback about the documentation!
Alternatively, you can tint the Button's label. That could look like one of these examples, depending on whether you want to inherit the tint color from ancestor views.

```swift
Button(action: myAction, label: {
    Text("Hello")
        .foregroundStyle(.green)
})

Button(action: myAction, label: {
    Text("Hello")
        .foregroundStyle(.tint)
})
.tint(.green)
```

---

Q: One question for SwiftUI ScrollView: What is the best way to get ScrollView's contentOffset?

A: There's not an API that provides you a binding to a CGPoint as a scroll view's content offset. However, there are several new APIs that allow you to transform views based on its relative position within a ScrollView.

The scrollTransition() API lets you apply customizations like scale, offset, and rotate that change as the view traverses a scroll view's visible region:
https://developer.apple.com/documentation/swiftui/view/scrolltransition(_:axis:transition:)

And the visualEffect() API lets you do even more flexible things both inside and outside a scroll view by providing you a geometry proxy and letting you do customizations based on that:
https://developer.apple.com/documentation/swiftui/view/visualeffect(_:)

If there are other cases where you'd like to read a scroll view's content offset, please do file feedbacks detailing your request!

---

Q: In SwiftUI, Core Data objects conform to ObservableObject, so our views can update automatically when the object changes. For example: struct DetailView: View { @ObservedObject var item: Item ... } With Observable replacing ObservableObject/ObservedObject property wrappers, will CoreData objects automatically conform to Observable?

A: @Model macro used to mark SwiftData models automatically provides conformance of the model to Observable. This way, the models have the capabilities that you'd expect.

While SwiftData can coexist with CoreData and connect to the same database, these two worlds don't mix. NSManagedObject conforms to ObservableObject, and @Model-tagged types conform to Observable.

---

Q: I'm curious about @backDeployed attribute from Swift 5.8
https://github.com/apple/swift/blame/main/CHANGELOG.md#2023-03-30-xcode-143
Is it something that cannot be used for new SwiftUI APIs or just was not adopted yet?
For example .scrollTargetBehavior modifier. It seems like it requires completely new type for parameter hence is not back deployed. Are there any @backDeployed APIs in SwiftUI this year?

A: Only functions and computed properties can be back deployed; anything that relies on new types or system functionality can't be back deployed.

---

Q: Is there a way to get a matchedGeometryEffect working across views from different "view controllers", like during a push/pop transition of NavigationStack or a present/dismiss of a sheet?

A: A matchedGeometryEffect doesn't currently work across most complex container views, like List, NavigationStack, or NavigationSplitView.

---

Q: (On watchOS,) is it possible to programmatically set the focus to a text field and begin text entry, with dictation as the text input method? The use case I have in mind is in the context of a note-taking app, where I want to reduce as much as possible the required physical interaction to start (and end) taking a note. Presently, that involves launching the app, tapping a text field, and then tapping again on the mic icon to start dictation (and then tapping the done button). Ideally, I'd like to get the entire interaction down to just one or two taps in total (e.g. open the app, then tap to start; or even use an app intent fired from the Apple Watch Ultra's action button to make it simply one physical button press to start; and then read the in-flight text entry to scan for a keyword to commit entry.)

A: TextField always launches to the keyboard first on watchOS, and you can easily get to dictation with the mic button. Keyboard is a wonderful and quick way to enter text, and it's always the preferred input method on watchOS. Please file feedback, explaining your use case, for why you'd want to launch to dictation first.

---

Q: I wonder if there is any performance difference between .opacity(0) and .hidden()? Also, is there any way to cause call of onDisappear on a subview, but preserve its state (for instance, if it's a ScrollView)? Simply removing view from hierarchy does the first, .opacity(0) does the second, but I'm struggling to find a way to do both.

A: onDisappear is tied to the view's lifetime so whenever its called the state inside that view will be reset. So there is no way to achieve saving that state but also calling onDisappear. However, you can always lift the state higher than the view and keep the state around yourself.

---

Q: I have an app that I’m adding CloudKit Sharing support to, using the Core Data+CloudKit integration.  I have an NSManagementObject that I’ve conformed to Transferable using the CKShareTransferRepresentation(exporter:), and that works great for creating new shares.  However, if I later open the ShareLink on an entity that is already shared, I get basically the same view, not the view that shows the existing participants.  Is this expected?  Is there a different control I should use here?

A: This does not look like expected behavior. Have you filed any feedback on this?

---

Q: Is there a way for a parent view to communicate with an off screen child inside a Lazy Stack or Grid? I want a child that has been scrolled off screen to know when the parent view has done something and adjust the behavior of its next onAppear, maybe with a boolean flag or something similar.

A: When the view is scrolled offscreen in a lazy stack, it won't receive updates but you can communicate information by passing the info directly to the view.
For example, you could have a counter the parent keeps track of and pass the count to the child view. When the view next updates it will receive an up to date value of count.
Hope that helps!

---

Q: Observation seems to bring good performance improvements, as far as I've heard that's due to Observable objects make SwiftUI only recalculate the body of views when observed properties change. But SwiftUI still diffs changes (right?), are there good tips to consider, like if it's better to use many Observable objects vs 1, or If it's ok to nest Observable objects inside Observable objects?

A: The diffing behavior remains unchanged in that once the view is invalidated we will still run body and avoid invalidating any downstream view that hasn't changed.
We do support Observable inside Observable and that is perfectly fine, and SwiftUI will track only the instance that you read in body.
I think it still important for you to consider how to better model your domain and what works best for you specific solution. With these new changes you should be able to have more expressive power in your design but still retain great performances.

---

Q: When transitioning from an ObservableObject to the new Observable macro, do we still need to use @State for our conforming view models?  Or is that only if we need to write to a property in a model?

A: You no longer need to use @State for displaying or writing properties of a Observable. I recommend watching the session "Discover Observation in SwiftUI" to learn more.

---

Q: Since MapKit annotation clustering is not yet available for SwiftUI, what is the better approach: using the UIKit solution or building this logic by myself on top of SwiftUI MapKit?

A: If annotation clustering is a feature you’d require, I’d recommend sticking with the UIKit APIs for now. Please do file a feedback report with your clustering use case though as we use those feedback reports to help inform our future design direction!

---

Q: In the SwiftUI animation talk, Kyle shows the new animation modifier which takes a content to avoid passing the animation down the attribute graph. Like so:

```swift
content
  .animation(.smooth) { $0.scaleEffect(...) }
```

Can the same be achieved with extra modifier to cancel the animation after the attribute I wish to animate? Like so:

```swift
content
  .transaction { $0.animation = nil }
  .scaleEffect(...)
  .animation(.smooth)
```

A: That's right – both approaches control the animation that is set on the transaction at the level of the modifier (.scaleEffect in this case).

The benefit of using the new .animation(...) { ... } modifier is that it only impacts the modifiers that you apply within the closure, and passes the original transaction+animation down to the rest of the hierarchy.

Using a separate .transaction modifier to 'reset' the animation would require you to know what the original animation was (unless you know that it should actually be set to nil for the rest of the hierarchy, as in your second code block above).

---

Q: How does SwiftUI keep track of what properties were used inside a body to invalidate views?

A: Hi Valentin, great question! There's a lot we could say about that, but I'll try to provide a brief overview:

In general, SwiftUI will compare the values of the properties stored in your views to detect if those values have changed. If any property of a view changes, the view will update and body will get called. Part of how SwiftUI can perform these checks efficiently is due to SwiftUI views being lightweight structs.

However, views can also store references to reference-type model objects, such as ObservableObjects or classes using the new @Observable macro and protocol. For ObservableObjects, if any published property is set, SwiftUI will update all views depending on it. This is why it's often important for performance to avoid having many views in the same app all reference the same ObservableObject, since they will all get updated regardless of the properties they use.

With the new @Observable API, we're able to improve on that significantly. When a view calls its body property, SwiftUI will record any getter accesses on properties of @Observable instances, and start listening for changes to those specific properties. If any of them change, SwiftUI will reevaluate the dependent views, re-record any property getter accesses that occur during the execution of body, and repeat the process. This ensures that views will only update if their dependent data has changed. We think it will be a big win for performance in most apps compared to ObservableObject!

---

Q: Do you have any guidance or code samples on adding and removing child SwiftUI views as part of a UIKit view hierarchy? As opposed to the an entire view controller being a UIHostingController.
I've run into issues removing SwiftUI child views from the UIKit hierarchy, where the SwiftUI view couldn't just dismiss itself since it's not composed in another SwiftUI view.

A: SwiftUI views inside a UIKit hierarchy must be inside a UIHostingController or part of a UIHostingConfiguration. Those types provide the necessary infrastructure to maintain/update the SwiftUI views.

---

Q: There's a fairly well known "flow layout" which can be described like this: it puts components in a row, sized to their preferred size. If the remaining horizontal space on the same row is too small to put the next component, the component is then shifted to the next row.

What would be the best way to achieve something similar in SwiftUI? Thanks in advance

A: The rough way for doing this would be to implement a custom layout conforming to Layout. Let's say you are implementing a horizontal flow you'd then use HStackLayout inside your implementation to measure and check each row.
* Propose a size to each child (nil width) to get back it's ideal width. Use this to determine how many of the children will be in the current row.
* Once we have a row we can use HStackLayout to stretch and place them.
* We now continue to the next row and we stop once all children has been consumed.

---

Q: If we have a horizontal scroll view inside a list within a section, is there a way to make just that scroll view go end to end while the rest of the list get paddings?

A: I imagine listRowInsets() would support this:
https://developer.apple.com/documentation/swiftui/view/listrowinsets(_:)
You can provide no insets by doing the following:

```swift
ScrollView(.horizontal) { ... }
  .listRowInsets(EdgeInsets())
```

---

Q: Is there an easy way to decide on a view contents based on space available? For instance, I have two buttons, first has icon + text in label, and second has just text. They are put in HStack and I want to remove text on a first button (leave just its icon) if one or both labels are going to be truncated.
For now to achieve this behavior I have to build monstrous constructions which involve hidden "probe" button instances and geometry readers to determine "preferred" text length by placing it in Group { label }.frame(width: 1000).frame(width: 0) where first .frame gives enough space for text to render and second .frame shrinks our "probe" Group to prevent it affecting overall layout.

A: That's a great question! ViewThatFits might be a good solution to this. It allows SwiftUI to pick from a few possible Views according to which is appropriate for the available space.
You can see more about it here: https://developer.apple.com/documentation/swiftui/viewthatfits

---

Q: How to run an @Observable class on the MainActor? (Assuming this code)

```swift
@Observable @MainActor final class SomeModel {}
struct ContentView: View {
   let model = SomeModel() // Doesn't work
   var body: some View {...}
}
```

A: Since you annotated your model @MainActor, you'll need the same annotation on the parent scope per Swift concurrency rules.

```swift
@MainActor // <====
struct ContentView: View {
  var model = SomeModel() // should work now!
}
```

---

Q: We arrived at a policy of never using @State because subviews of a ForEach would very often cause an edit to the model that supplied the array of that ForEach, among other cases that caused @State to get obliterated. Environment and ObservableObjects became our goto. Were we doing it wrong?

A: It sounds like the identity of your view might not be stable across whatever edits you're making, or might depend on the properties that are being edited, which would case the @State data to be cleared. If the IDs of the model type you're using in your ForEach change after an edit, that would change the view identities and clear the associated state. The Demystify SwiftUI session (https://developer.apple.com/wwdc21/10022) has more information about view identity.

---

Q: Is there a way to wrap text around some other view? Something similar to a dropcap effect

A: This is not supported in SwiftUI. You'll have to use UIKit / AppKit to do something like this via exclusion paths.

---

Q: I have a bottom sheet with scrollable content in my app, similar like the one in Apple Maps. In some conditions, e. g. when the sheet is fully presented and the scroll offset is 0, I would like, that you can only scroll in one direction.
At the moment, I use gestureRecognizerShouldBegin of UIScrollView in an UIViewRepresentable for that.
But I would like, to accomplish that just with SwiftUI. Unfortunately ScrollTargetBehavior is only called, when a scroll gesture ends. Is there something similar, for when a scroll gesture starts? Or do you have other ideas, to achieve that behavior?

A: I dont believe this is possible at the moment but a feedback detailing your request would be appreciated!

---

Q: Is there a way to programmaticaly control the progress of an animation in SwiftUI? A kind of UIViewPropertyAnimator but for SwiftUI?

A: There is not a tool directly analogous to UIViewPropertyAnimator, but there are a few different approaches that you can use depending on the situation.

One new API this year is KeyframeTimeline, which lets you directly compute keyframe-driven interpolated values for a time that you specify, which you can then apply to the view using modifiers. You could drive this using a gesture, or any other source of 'animation progress.'

---

Q: In SwiftUI there is a PropertWrapper to write and read UserDefaults. Is there something similar for KeyChain?

A: Hi - thanks for your question! This isn't something that we have API for at the moment. A feedback for it, along with any information you can provide about your use case would definitely be appreciated.

Follow-up-A: https://medium.com/@ihamadfuad/swiftui-keychain-as-property-wrapper-ca60812e0e5e

---

Q: What is the best way to know if a child view has been scrolled off screen as opposed to the parent view being taken off screen? (Basically different causes for onDisappear.) The best I've been able to find so far is using a GeometryReader and checking to see if the child view's geometry is within the parent's.

A: You could maybe use the new scrollPosition(id:) modifier though that won't give you the state of every currently visible view.
Using the GeometryReader is probably the best solution if you really want to care about only the visible region.

---

Q: Just out of pure curiosity why do we have a ForEach loop for views and for loop for everything else?

A: Really interesting question! For loops are used in a few different ways throughout Swift and SwiftUI, so I think the best way to answer this is to take a quick look at each, and highlight why the solution we chose was the right one for SwiftUI.
The forms we most commonly see for loops in swift are the following:
For loops the control structure for i in 0..<10 { print(i) }
For loops in result builders
ForEach in SwiftUI
In SwiftUI for each use case, we’re not running imperative code (executing each case of the loop one after the other), so use case 1 isn’t super applicable. Instead, we’re building up a declarative representation of the multiple views specified by the for each. That leaves the remaining two approaches as possibilities.
At a quick glance, option two seems like it could work. Result builders don’t just execute each call in sequence, they transform things like for item in items { … } into values by combining the results produced in the for loops associated block. This could work for creating a primitive SwiftUI-like syntax. You could write something like:

```swift
for person in contacts {
   Text(person.name)
}
```

So why doesn’t SwiftUI do that? This works at the surface level, but imagine if the contents of the contacts array were to change. How would you know how to animate that change?
Say you went from a list containing “Amanda, Ben, Carly” to “Amanda, Carly, Ben”. Did the list just get reordered, or did Ben change their name to Carly, and Carly change their name to Ben? You can’t disambiguate between these two fundamentally different changes.
SwiftUI needs additional information for each item in the list, something to provide it a unique identifier. We use that unique identifier to keep track of which item is which even when they change locations in the underlying data structure.
That’s why we went with option three, a unique type, ForEach, which requires you to provide a way to determine the unique identifier for each element in the collection.

---

Q: Is there any way to make a view refresh animated upon changes to a @Query parameter when deleting an object from a ModelContext while also avoiding the view animating upon the first load of the data from the ModelContext?

A: Thanks for the question!
There's not a simple way to do that today. I'd love a Feedback with that use case!
If your model is equatable you could probably hack something together along these lines:

```swift
@Query... var models
@State private var hasLoaded = false

MyView()
    .animation(hasLoaded ? .spring : .none, value: models)
    .onChange(of: models) { 
         hasLoaded = true
    }
```

Or you might use task instead of onChange and enable animations after some short delay.

---

Q: I think I saw something about Text being embedded in another Text view. Is this correct? If so, what features does it support? Could it be an alternative to using AttributedString?

A: Yes, Text does support doing string interpolation with other Text (and Images too!)
You can write something like
```
Text("Hello \( Text(person.name).foregroundStyle(Color.green) ) !")

---

Q: Is it possible to now have a ScrollView that can move in either direction (up/down) with no limits (infinite scroll to any date in the past or future, think calendar) without any stutter/jump? The problem would occur when inserting items at the top of the list.

A: There have been improvements to scrolling backwards in iOS 17.0 in general and if you use a scrollPosition modifier, the ScrollView will try to keep the view currently visible at the same relative offset when your data changes.
However, there still isn't a concept of an "infinite" scroll view. It has a finite size for its content so you would be responsible for adding / removing views as a user gets close to / away from the end.

---

Q: Can Boolean operations be applied to shapes or views? For example, can a circle be subtracted from a shape to create a transparent cutout, like in vector editing software?

A: There are operators like https://developer.apple.com/documentation/swiftui/shape/subtracting(_:eofill:) https://developer.apple.com/documentation/swiftui/shape/symmetricdifference(_:eofill:) https://developer.apple.com/documentation/swiftui/shape/union(_:eofill:)

---

Q: How can we add jump bars like in contacts to our ListViews, ScrollViews, etc?

A: SwiftUI doesn't currently support a ScrollView or List index.

---

Q: With the new SwiftData, if I understand correctly we access the model using an @Environment variable on the view. Doesn't this contradict the idea of MVVM which the idea is to have your view be as simple and be driven by the View Model. What is Apples recommended architecture or method to handle this?

A: The environment is a data flow tool for helping to conveniently pass data through large portions of your view hierarchy, as well as scope data dependencies to only the views that need them.
It's not required for models to be passed through the environment. You could pass models directly between views as normal properties, to give a simple example. The right way to pass data through your app often depends on the specific data your working with and how it's used.
So the environment is just a tool, and should be used for the cases where it works best. For example, many apps leverage the environment to make it easy to pass larger models between views that represent the main screens of the app, helping avoid boilerplate. But then within those screens, that break apart those models into smaller pieces of data that are passed as normal properties to smaller component views, like a specific custom control, so that each view's data is scoped to just what that particular view needs, i.e. allowing those views to be as simple as possible.
When architecting your views, I'd focus on that general guidance: for this particular view, what data does it need? If it's a smaller view that's driven by a few specific properties, then it might not be appropriate to pass in a whole model via the environment. If it's a larger view that's constantly being iterated on, like an entire screen of the app, then  accessing the full model conveniently via the environment might be the best and most convenient option.

---

Q: Hi! What would be the best way to have global default values for @AppStorage properties? Right now, I seem to have to replicate the defaults in every view I want to access them...

A: Hi - thanks for your question! You have a couple of options for this, I think.
- Global constants, optionally wrapping them into a struct if you want to namespace them.
- Creating a custom DynamicProperty that encapsulates AppStorage with your constant value.

I don't have an example handy, but the general idea is to make a new type that conforms to DynamicProperty and internally uses @AppStorage with your key and default value specified. Then, you can use this new dynamic property in the same places you were previously using @AppStorage.

---

Q: Is there any new API in Sonoma where I can set the sidebar toolbar color? The current API in Ventura will set the entire toolbar color for both sidebar and content.

A: Hi - we don't have anything new for this in macOS Sonoma. If you could file a feedback for this, and include any info about your use case, that would be helpful. Thanks!

---

Q: I have a SwiftUI List that can receive dropped images via .onInsert(of:), which passes NSItemProviders. How can I get a NSFilePromiseReceiver from that provider? I want it to support file promises to get full-resolution images from apps that provide them, e.g. the Photos app (on macOS, though presumably similar on iOS).

A: Can you clarify how you want to use the promise receivers as opposed to tiff or jpeg data?

Follow-up-Q: When dragging a photo from Photos on Mac to my list, it only provides a low-resolution as a JPEG. The full resolution is included as a file promise. I want the full resolution, so need to resolve that.

Follow-up-A: Have you tried using dropDestination API? If there are any promises provided, we attempt to load them for you

Follow-up-Q: Interesting; I haven’t tried that. I’d have to have a separate drop view for that, though. I’d prefer to be able to drop multiple photos in a list, to insert them in arbitrary locations.

Follow-up-A: If you have a ForEach that builds up the List, dropDestination behaves similarly to onInsert (it is available for DynamicViewContent). Can this work for your use case?

---

Q: When I put a Text in the second column of a columns style Form, the SwiftUI view gets extended beyond the available horizontal space (a fixed width constraint on the UIHostingController's view) seemingly due to not line wrapping quite right. Is there a way to ensure it won’t grow larger than the containing view or could this be a bug? I have a sample project in feedback FB11809780. :)

A: This looks like a bug; thanks for filing it!

---

Q: Is there a way to combine some kind of an overlay button in a SwiftUI TextField, like the ‘clear button’ that can be used in UIKit’s UITextField?  I’ve tried experimenting with this in the past, but have never found the right combination (seems more likely that I’m doing something wrong than something that’s not possible)

A: There isn't a really clean way to do this today. We'd love a Feedback to help us prioritize making this easier.

In the meantime, you might try adding an .overlay(alignment: .trailing) { … } to the TextField. A button in the overlay could clear the text binding that’s passed to the field.

---

Q: I see that iOS 17 brings TextEditorStyle which we can make ourselves as it has a makeBody method https://developer.apple.com/documentation/swiftui/texteditorstyle/makebody(configuration:) Is there anyway to do this for TextField as it's TextFieldStyle is not public in the same way.
I'm trying to make a TextField's that have a border that become green when focused, but I can't seem to find a way to get the FocusState in a view that wraps TextField. As in

```swift
    MyTextField(...)
    .focused($focusState, equals: .email)
```

in MyTextField:     @Environment(\.isFocused) will not be set to true when the focusState above it is set to .email.
Is it possible to make a design that reacts to the focus state?

A: The isFocused environment value tells you if you have an ancestor view that has focus. That won't be the case for MyTextField, since the text field that it wraps is what actually has focus.
Instead, I'd expect some combination of @FocusState/focused(_:equals:) to work for you. For instance, with MyTextField, you could use a local private focus state binding to see when focus is in the wrapped text field, and draw a border accordingly. Something like this:

```swift
struct MyTextField: View {
    @FocusState private var isFocused: Bool
    ...

    var body: some View {
        TextField(...)
            .focused($isFocused)
            // stuff for drawing border if isFocused == true
    }
}
```

---

Q: Can we use @Query outside of views? For example, in a DataFetcherService class? If not, how to fetch SwiftData outside of views like this, in a way that will make views update themselves when the underlying data changes?

A: Query is designed to work dircetly with SwiftUI views. SwiftData provides imperative APIs for accessing data. In a "service", for example you could construct a ModelContainer, and fetch or modify models via its mainContext. Your "view model" can be an Observable object that stores result of those fetches, and it'll work great with SwiftUI. The docs for working with ModelContainer can be found [here](https://developer.apple.com/documentation/SwiftData)

---

Q: I have a child view nested within a parent (parent) view, nested within its own Lazy Stack parent (grandParent). If I scroll on grandParent such that parent goes out of view, an then some more, when I scroll back to parent the child view is now gone. I tracked the memory allocations via Instruments and found that the root view inside the child (there's a whole UIViewRepresentable structure in there) is getting deallocated/removed from memory. Is there a way to either
a) Prevent a descendant view from being destroyed when it is out of view?
b) Force a child view to be re-initialized when a view comes back into view?

A: Thanks for the question!
Directly doing a) or b) isn't possible. I'd suggest trying to lift your state above the lazy container so you're just passing a reference or a binding to it into the parent and child views. This allows the grandparent to keep the state alive even as the deeper views come and go.

---

Q: If I fetch a large dataset (using @FetchRequest or @Query) into List or LazyVStack, should I consider batching in the fetch request, or will performance be good because these load data lazily?

A: In general List and LazyVStack should do work proportional to the view port, so even if the dataset is very large they should be able to handle it. Check out https://developer.apple.com/videos/play/wwdc2023/10160/ tomorrow for more info on this.
At the point, when the dataset is so large keeping all the model objects in memory... that's the time to start looking at batching.

---

Q: When using the searchable view modifier and tapping the search bar the navigation bar is hidden with an animation, is it possible to keep it shown? I think the hidesNavigationBarDuringPresentation property is used for this behaviour in UIKit. Is there an equivalent in SwiftUI?

A: There is no equivalent API in SwiftUI for that property, but a feedback requesting this would be appreciated!

---

Q: How should we handle situations where a segmented control is appropriate but the actual design of a system segmented control makes it unfeasible (either there are either too many options or the labels are too long)?

A: Hey Sam, Thanks for the question.
Assuming iOS, in cases where you there are too many options AND the labels are very long, we typically recommend using the wheel picker style or the navigationLink picker style if contained within a List.
However for most cases many options or medium length labels the menu style is always preferred.
Do you mind expanding on your use case? It seems like you would prefer a segmented style but or system design poses a barrier to you. Could you go deeper on the specifics?

Follow-up-Q:  Thanks for your reply! It's similar to Twitter's profile tabs, switching a profile view between different types of posts: "Posts", "Posts + Replies", "Media", and "About"
Right now, it's just those four segments but I can easily see this snowballing into many more. I don't think the standard system segmented control is sustainable here with those labels — five items will make it extremely tight esp with internationalization

On Android/Material, they've got these scrolling tab bars but there's not a clean parallel to that on iOS. I've seen something like that for the chip filter row in Maps (when you tap a pre-defined category like "Restaurants" but that seems more like a filtering thing rather than a view mode switcher)

Follow-up-A: I see. So ideally you'd would want a "minimally styled" segmented picker that could scroll horizontally in certain cases? A bit like the new palette picker works within menus? This is an interesting use-case.
Could you please file a radar describing your specific use-case? That would be highly appreciated.

---

Q: To remove the translucent background of the sidebar on macOS when using a NavigationView I was recommended to set .scrollContentBackground(.hidden). But I have not been able to get this to work. What is the correct way to implement this?
Thanks!

A: Hi Fredric, a great question. Can you give a little more detail what you're trying to achieve here? Is the intent to have a solid background color for the sidebar? Or just the system background color, only non-translucent?

Follow-up-Q: I basically want to set my own solid background color for the entire sidebar. Right now I am forced to use introspect to set the background color of the List combined with setting the background for the entire sidebar.

Follow-up-A: could you give this code a run and see if it is close?

```swift
@main
struct TestApp: App {
    var body: some Scene {
        WindowGroup {
            NavigationSplitView {
                ZStack {
                    Color.green.ignoresSafeArea()
                    List {
                        ForEach(0..<100) { int in
                            NavigationLink("Option \(int)", value: int)
                        }
                    }
                }
            } detail: {
                Color.yellow
            }

        }
        .windowStyle(.titleBar)
        .windowToolbarStyle(.unified(showsTitle: true))
    }
}
```

Follow-up-Q: I want a solid "plain old" color and I am using sidebar list styling. I am not using NavigationLinks but instead change the state of a selected row (folder) that in turn  updates the content view. I will attach a screenshot so that you can see what it looks like.

Follow-up-A: I think what I've recommended above should work for your use case, but if you're trying to use a material there in the sidebar, you might run into some trouble.

---

Q: How can I implement a confirmation, when the user pulls down a sheet? Exactly how adding an event in the Calendar app asks if you want to discard or keep editing the new event.
There is View.interactiveDismissDisabled() to prevent it being pulled down, but I can't find a way to react to the user trying to pull it down.
In UIKit it would be https://developer.apple.com/documentation/uikit/uiadaptivepresentationcontrollerdelegate/3229888-presentationcontrollerdidattempt

A: Thanks for the question!
interactiveDismissDisabled() is the best we have with a pure SwiftUI solution today.

---

Q: Any guidance on creating an adjustable  vertical split view?  In portrait orientation, I want a view on the top half and bottom half of the screen, and I want the user to be able to tap and hold a divider line  between these 2 views to increase/decrease how much of each view is show by dragging it up and down

A: Ah, on macOS there is VSplitView but on iOS, this should be doable via the Layout protocol

---

Q: Is it possible to implement a full text search in TextEditor? Where after entering a few letters, it would scroll to the first matching string, and offer buttons to move to other instances of matching string

A: TextEditor has built in support for find and replace as of iOS 16.0. You can control the presentation of this UI with the findNavigator() modifier:
https://developer.apple.com/documentation/swiftui/view/findnavigator(ispresented:)

---

Q: Is there any way to enable the same behavior in SwiftUI as usesPredominantAxisScrolling in NSScrollView. Most use cases of graphical nature such as a drawing canvas should scroll freely in both axes.

A: There is no equivalent to that API in SwiftUI though a feedback detailing your use case would be appreciated!

---

Q: I'm trying to use a SwiftUI button in a list without using a NavigationLink.
I'm trying to replicate the effect you get when a NavigationLink is tapped, where the row background changes to systemGray4.
Using a default button, the list row highlights if you tap and hold, but not after a quick tap like you see when using a NavigationLink.
Is this a known issue- or is there a better way to achieve this?

A: It's a bit of a hack, but you could use a custom button style and change the background if the button is pressed OR if the value that the navigation path contains the associated value.

Essentially, your style would show the “pressed” state if the button was pressed or if its associated view was presented.

---

Q: What happens when an observable property changes, but the view observing it is out of visible bounds? Does the view get updated or update happens when the view becomes visible?

A: The view does get updates regardless of visibility of the element displaying the object's property.

---

Q: Is it valid to use nested buttons in SwiftUI? If so, are there recommendations for how to solve the associated accessibility issues (where the inner button is cannot be focused by Voiceover)? If not, what is the recommended approach?

A: You’re right that only the outermost Button can be focused by VoiceOver, but the actions of the nested buttons can still be performed by VoiceOver users!
A nested Button will be added to VoiceOver’s custom actions menu that displays other types of actions an element can perform, such as the equivalent of a right-click to show a menu. (On macOS, that can be activated by pressing the VO keys + Command + Space.)
For example, you might have a Button or NavigationLink that represents an item in a grid, each with a “favorite” button within it. SwiftUI will automatically add the “favorite” button to VoiceOver’s custom actions menu.
Alternatively, you could consider using a ZStack to instead position one button above the other, instead of nesting them.

---

Q: Will there be additions to SwiftUI for keyboard navigation instead of what we currently use NSEvent local monitoring for?

A: As far as keyboard input goes, there's a new onKeyPress() view modifier that lets you react to keyboard input directed at the focused view. That may get you what you need, though of course it depends on what you're currently using event monitors for.
If you have specific cases where you still need to fall back on AppKit event monitors, please consider filing feedback reports describing them!

---

Q: Just want to confirm, among all the new animation updates in SwiftUI this year, there's nothing that lets us customize the presentation of sheets  - either size, position, or custom transition animations, right? In other words, there's not really a bridge to the UIViewControllerTransitioningDelegate for presenting sheets or pushing new views onto the navigation stack?

A: That's correct. We'd love a feedback request to better understand the effects you're trying to achieve.

---

Q: In "Design with SwiftUI," Will showcased a "ticker" animation for the walking radius label at 8:26. I once tried to create a similar animation by adding transitions to separate Text views for each digit in an HStack. Unfortunately, this caused the font's kerning to be incorrect. It seemed to me like the implementation displayed in the aforementioned session had correct kerning. Can you talk about how something like that could be implemented?

A: Thanks for the question.
I don't have access to the specific code that Will was demo'ing. My guess is that he used the monospaceDigit modifier on the Font to get the font variant that kerns digits without shifting as they change.

You might also look at the contentTransition modifier which has some built-in options for transitioning between Text values that include numbers.

---

Q: UITraitCollection has a trait called userInterfaceLevel. Do we have something similar in SwiftUI? I want to be able to tell when my view is in an elevated context.

A: You can use the new UITraitBridgedEnvironmentKey protocol to bridge the user interface level from UIKit to SwiftUI. However, depending on what your goal is, it may be simpler to use the builtin SwiftUI views and colors that automatically adapt to the current level.

---

Q: Most of my apps are MVVM, will the SwiftData @Quary work if it’s inside an @Observable viewModel instead of a View?

A: @Query uses the underlying ModelContext of the View that is provided via Environment.
Because of that, @Query works only when it can access the Environment.

Follow-up-Q: Thanks, so how do we go about using SwiftData from where it cannot access the Environment? eg. viewModel or some kind of manager?

Follow-up-A: You'll have to do all the same operations manually: pass the ModelContext to the model, ask it to provide you the models, etc.

---

Q: Is there a multi-platform way to just have a single Window in SwiftUI?
There is "Window" for macOS and the "UIApplicationSupportsMultipleScenes" Info.plist key set to false for iOS, but doing a single thing instead of having to do both would be nice.

A: Hi - thanks for the question! You are correct that Window is only available on macOS. We'd certainly appreciate a feedback for this, though. If you can include any information about your specific use case, that also helps. Thanks!

---

Q: Do the new additions to ScrollView provide hooks into the pan gesture that powers the view's scrolling? I'd like to build a custom pull-to-refresh control for my content, but I'm not sure if this is possible without understanding when the pan gesture is in progress and when a user has lifted their finger.

A: There are no new APIs around hooking into the state of the scroll view's pan gesture.
However, there is a new scrollTransition() and visualEffect() API that lets you transform a view based on its relative position within the scroll view's visible region.
You could use potentially use this to create a custom refresh indicator without needing to interact with the pan gesture. I'm thinking put a refresh indicator at the top of your scroll view that reveals itself as you scroll up.

---

Q: With current ObservableObject conformance in iOS 16 we were able to use @published vars with the $ prefix to get access to the publisher and then could set up combine chains to react to updates to the data. With moving to @Observable, is there a new recommendation on how to react to changes to the vars in your data classes? Otherwise it seems that these changes further push code into the views and move away from separation of concerns.

A: I recommend checking out and even participating in the active open source Swift Evolution review of @Observable: https://forums.swift.org/t/second-review-se-0395-observability/65261
In the "Future Directions" section of the proposal it says: "An earlier version of this proposal included asynchronous sequences of coalesced transactions and individual property changes, named values(for:) and changes(for:)."
However even without values(for:) to track individual property changes, I'd recommend keeping as much logic in your model as possible. Computed properties on your model can come in really handy for this. SwiftUI will intelligently invalidate when any of the stored properties accessed from the computed property change.

---

Q: Is there a way in SwiftUI to remove the shadow from a Navigation bar?
For example, in UIKit, I would use:
UINavigationBarAppearance().shadowImage = nil and/or UINavigationBarAppearance().shadowColor = .clear
Using .toolbarBackground(.someColor, for: .navigationBar) still displays a shadow.
Is there a way to remove that shadow?

A: There is currently no equivalent to that in SwiftUI but a feedback detailing your request would be appreciated!

---

Q: With Swift Charts, when I update the underlying data model object,, my charts do update, but there is no animation. Do I have to write code or modifiers to enable animated transitions? I was under the impression that changes would be animated by default.

A: Are you using withAnimation, for example withAnimation(.easeInOut) { ... change the data ... }?

Follow-up-Q: no, should I? I have not seen this code in the sample code (SwiftChartsExample), where the transitions do animate. Hence my impression...

Follow-up-A: Please, try it and file a feedback in case of a mismatch between what you expect based on talks/docs and what behavior you see. You can file a request using Feedback Assistant by visiting https://developer.apple.com/bug-reporting/

---

Q: Suppose I was using @SceneStorage to persost some bits of pertinent scene state for later restoration. But SceneStorage can only accommodate some basic types and the docs recommend not to store anything large there. (A JSON string could of course store virtually anything.) So all is well, I can make a UUID string and store it in @SceneStorage and put the big data elsewhere so I can restore the whole scene.
But when the user or OS eventually deletes a scene which was once persisted, how will the app come to know it's no longer there so it could reclaim disk storage?

A: Hi - thanks for your question! You are correct around the current API for SceneStorage. I don't believe we have anything that could help with your specific case here, but we'd love a feedback with these details so that we can improve on the API.

---

Q: Is there a way to know when a compact date picker is open or closed? For example I have a compact date picker and when the date is changed I want to save the date to CoreData. Currently I’m using onChange but I feel like it’s not so efficient, I wish it would support .focused unless there’s a better solution

A: Currently, there is no support for detecting when a DatePicker is opened or closed, but this does seem like it could be useful. Please file a feedback with some more information about your use case since we use feedbacks to help inform our future design direction. Thanks!

---

Q: I create a custom SwiftUI component and follow the style approach that we use for native buttons to customize this component from the call site if need. My style is just a protocol. Is there a way to mimic the behavior of native styles like ButtonStyle where we can access the environment directly and allow automatic body invalidation even though it’s not a view?

A: There is! You can create types that receive updates to SwiftUI properties just like View by conforming them to the DynamicProperty protocol. Whenever SwiftUI encounters a property conforming to DynamicProperty inside a View, it will also update any SwiftUI properties stored inside it.
You should be get started with the following example:

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        MyCustomControl(state: .constant(true), style: MyStyle())
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}


struct MyCustomControl<Style: MyCustomControlStyle> : View {
    @Binding var state: Bool
    
    var style: Style
    
    var body: some View {
        style.makeBody(configuration: .init(isOn: $state))
    }
}

struct MyCustomControlConfiguration {
    @Binding var isOn: Bool
}

protocol MyCustomControlStyle<Body> : DynamicProperty {
    associatedtype Body : View
    
    typealias Configuration = MyCustomControlConfiguration
    
    
    @ViewBuilder func makeBody(configuration: Self.Configuration) -> Self.Body
}

struct MyStyle : MyCustomControlStyle {
    @Environment(\.colorScheme) var colorScheme
    
    func makeBody(configuration: MyCustomControlConfiguration) -> some View {
        Group {
            colorScheme == .light ? Color.yellow : Color.red
        }
        .onTapGesture {
            configuration.isOn.toggle()
        }
    }
}
```

---

Q: I have a question about making text clickable with hashtags and mentions. What is the most effective way to detect clicks on hashtags and mentions within text? I would appreciate any guidance or suggestions you can provide. Thank you

A: Have you tried using markdown or AttributedString with .link attribute? These links should be handled through the environments \.openURL action.

---

Q: With the new Observable macros, will this change the suggested app architecture away from MVVM? To me it seems like we almost no longer need the view model?  Thanks!

A: Our goal is always to make it so that SwiftUI views can be as simple as they need to be, and that the most intuitive code to write also provides great performance.
The new @Observable macro will definitely help! Unlike with ObservableObject, views track dependencies to @Observable objects based on the specific properties of those objects that they actually use within body. This means that there are a number of cases where apps may have previously needed to restructure their ObservableObject model code to break models into smaller pieces just to achieve good performance, where @Observable will provide better performance without needing to do similar tricks. This may eliminate the need to create bespoke, per-view models in some cases.
Please try it out and share your feedback with us! If there are places in your code where @Observable is not an improvement over using ObservableObject, we'd love to hear from you.

---

Q: onKeyPress does not work inside a ScrollView. Do you know if this a known bug or feature?

```swift
struct ContentView: View {

    @State var text = ""

    var body: some View {
        ScrollView {
            TextField("placeholder", text: $text)
                .onKeyPress("a") {
                    print("Pressed a!")
                    return .ignored
                }
        }
    }
}
```

A: This is a known issue in the seed. In general, onKeyPress has trouble interoperating with bridged AppKit views that also try to handle key events.

---

Q: If I want to add a SwiftUI view to a UIView, do I also have to reach the view controller and add the Hosting Controller as a child view controller? Or can my leaf UIView just retain the Hosting Controller, and just add the main view of that hosting controller as subview to my UIView?

A: The UIHostingController should be added as a child view controller, otherwise some functionality may not behave correctly.

---

Q: First of all let me say the new modifiers for Scrollview are excellent, well done :v:
Now - can we tell ScrollView to start/end scrolling programmatically?
I want to build an app with a daily timeline that scrolls and may have an event placed on top of it. This event can be moved around and stretched/contracted, as in the iOS Calendar app. When the event is stretched to the edge of the visible area, the scrollview should start scrolling automatically to reveal more of the timeline. In the iOS Calendar app, moving or stretching an event beyond the visible timeline causes the entire timeline to smoothly auto-scroll and reveal the area into which the user wants to move or stretch the event.
Unfortunately I was unable to implement this. Already submitted FB11955036 in the past. There should be a method to start/stop scrolling of Scrollview or List, along with a setting for scrolling speed. Something like: ‘.scroll(isOn: Binding<Bool>, direction: .beginning/.end, speed: CGFloat? = nil)’

A: The new API doesn't support this use case.
You might experiment with rendering your timeline using a ZStack or Canvas rather than a ScrollView. This would let you maintain an offset manually and adjust the contained views when the user's gesture reaches the edges of the containers bounds.

---

Q: Are there any changes to TextField in iOS 17? In particular, can we have a way to clear the text?

A: What do you mean by "clear the text"? You can write to the binding of the text field to be an empty string which will "clear the text".

Follow-up-Q: Like a button to clear the text?  (I asked something similar above)

Follow-up-Q: Yes, mine was the same as Matthew’s question. I’d prefer that a “clear” function was built-in to TextField in similar fashion to AppKit and UIKit.

Follow-up-A: Ah, no there is no API for that currently in SwiftUI though a feedback requesting this would be appreciated.

---

Q: Is it possible to use a List within a ScrollView without having to manually set the .height() of the List to make its cells visible within the ScrollView? Thanks!

A: Hi! We want to know more about your intended usage - could you explain more on the 'visible' part? Is it that you are trying to make List itself non-scrollable and let the outer ScrollView to drive the scrolling? We would also appreciate a feedback so that we could keep track of it and communicate with you on this topic, thank you!

---

Q: When using a number pad with a TextField there is no done button, is there any plans to add one? If not what’s expected to happen once the user is done inputting (other than manually implementing a done button)

A: We recommend adding your own "Done" button in your UI.

---

Q: Do the new (or existing) ScrollView APIs allow access to the content offset or away to apply effects to a view based on its position? E.g. an scroll view of album art views that rotate or have a transform applied as the list scrolls.

A: Yes! There are two new APIs you should check out.
scrollTransition():
https://developer.apple.com/documentation/swiftui/view/scrolltransition(_:axis:transition:)
visualEffect():
https://developer.apple.com/documentation/swiftui/view/visualeffect(_:)

---

Q: Any advice on how to get matchedGeometryEffect to work with View content that is inside a NSScrollView implemented by NSViewRepresentable?

A: As you've found, matchedGeometryEffect does have limitations when SwiftUI views are split between different hosting views/controllers. We would really appreciate a Feedback with specifics of your use case to help prioritize improvements here.

---

Q: I was trying to customize the colors inside the Picker, but it seems this is currently not possible. For example with the code below I see all items as black and white, not colorized (and as you see  tried different color properties - none of them helped):

```swift
enum Flavor: String, CaseIterable, Identifiable {
    case chocolate, vanilla, strawberry
    var id: Self { self }
    var color: Color {
        switch self {
        case .chocolate:
            return .brown
        case .strawberry:
            return .red
        case .vanilla:
            return .yellow
        }
    }
}


struct TestView: View {
     
    @State var selectedFlavor: Flavor = .chocolate
    
    var body: some View {
        List {
            Picker("Flavor", selection: $selectedFlavor) {
                ForEach(Flavor.allCases) { flavor in
                    flavorView(flavor)
                }
            }
        }
    }
    
    func flavorView(_ flavor: Flavor) -> some View {
        HStack {
            Text(flavor.rawValue.capitalized)
                .foregroundColor(flavor.color)
            Spacer()
            Image(systemName: "birthday.cake.fill")
                .tint(flavor.color)
                .foregroundColor(flavor.color)
                .accentColor(flavor.color)
        }
    }
}
```

Is this by design or a bug? If by design, what would you recommend as an alternative UI element for this kind of situation (i.e. when we need to provide a single item selection, but also have items use images / text with colors)
Thanks in advance.

A: This is a known limitation, but we would appreciate feedback via Feedback Assistant. As a workaround, you can use ImageRenderer to create a rasterized version of your Image with the tint applied, and use that in your labels.

```swift
@MainActor
func image(for flavor: Flavor) -> Image? {
    let renderer = ImageRenderer(content: Image(systemName: "birthday.cake.fill").foregroundStyle(flavor.color))
    guard let cgImage = renderer.cgImage else { return nil }
    return Image(decorative: cgImage, scale: renderer.scale)
}

@MainActor
func flavorView(_ flavor: Flavor) -> some View {
    HStack {
        Text(flavor.rawValue.capitalized)
            .foregroundColor(flavor.color)
        Spacer()
        if let image = image(for: flavor) {
            image
        }
    }
}
```

---

Q: How can we use BGProcessingTask with SwiftUI? For BGAppRefreshTask, we can use the viewModifier backgroundTask(_:action:) on a Scene, but it does not seem to support long processing task.

A: Similar to the BackgroundTask framework's BGTaskSchedule, registering for a background processing app refresh task using the SwiftUI background task SceneModifier is the same as registering for a plain app refresh task. For both, using the .appRefresh("Identifier") task type as the property to the .backgroundTask modifier will give you your desired callbacks and runtime.

---

Q: In UIKit, we have access to semantic colors for standard and grouped content background colors (like systemBackground). These colors are not in SwiftUI and we need to use the uiColor initializer in Color to get these colors. Is that intentional?

A: The .background() view modifier gives your view the appropriate default background style. If you are missing other semantic colors, please file feedbacks with your use cases.

---

Q: Question for PreferenceKey: does reduce get called with default value? According to this article https://www.fivestars.blog/articles/preferencekey-reduce/, it shouldn't. But I notice often that my reduce method is triggered with a default value of 0 even though none of my views set the preference key to 0.

A: The reducer does not get called with the default value, but there could be another view in your app that's setting the preference to 0.

---

Q: How can we create a HUD in SwiftUI, that should appear above the top View (root stack or modal etc.). In UIKit, the solution was to use a UIWindow on top of the main app UIWindow.

A: An overlay anchored to the root view is the usual approach in SwiftUI. You might also experiment with the zIndex(_:) modifier.

---

Q: Thanks for all the help so far :) Whats the best way the best way to move focus to the next textfield when the user taps the return key on the keyboard?
I use onSubmit like (with two textfields in a VStack for email and password and corresponding FocusState)

```swift
onSubmit {
  if focus == .email {
     focus = .password
  else if focus == .password {
    // do the login
  }
}
```

But this causes the keyboard and the UI (that has moved due to keyboard avoidance) to jump down and up. It's quite jarring for the user. I guess the textfield has already started dismissing the keyboard before onSubmit is called.

---

Q: How does SwiftUI locate metal shaders in a project, and how might you debug a shader not being applied to a view?

A: Hi! Xcode compiles .metal files into a default library called default.metallib. Though if you want to customize the process of SwiftUI looking up for a shader, you could use a custom ShaderLibrary using its constructors that accept Data / URL types. As for debugging the shader not being displayed, I would suggest first make a simple shader that only displays a simple color - make sure it shows up first. It could be that the signature does not match with what  colorEffect, distortionEffect, or layerEffect requires, so please check with documentation.

---

Q: SwiftUI shaders are mind blowing stuff. I've been working with MTKViews to achieve similar visual effects in the past. One thing I haven't been able to achieve that way with SwiftUI was to create effects that work on the background of a view, for example to create a custom backdrop (gradient) blur effect. Do these new shader-related APIs support that use case?

A: Hi - thanks for the question. We don't believe this particular use case is supported by the new Shader APIs, but we'd love a feedback with these details, to help inform future updates to the API. Thanks!

---

Q: I have 2 questions about offset: Say I have a button with offset(y: 200)
1. Does the touch area of the button moves along with the visual effect of moving the button?
2. Does the offset somehow affecting the layout of the parent view?

A: 1. Yes, the touch area does move to match what you see on screen.
2. No, the offset effect is layout neutral and only impacts the presentation of the view. This also explains why offset is available through the new .visualEffect modifier, which only supports effects that are layout neutral.

---

Q: The new layerEffect modifier requires a parameter of type SwiftUI::Layer for its Metal function. However, I'm not sure how to import the SwiftUI namespace in my Metal files. Is there an #include <SwiftUI/metal> directive I should use? I couldn't find any mention of this in the documentation.

A: Hi! Please add this line #include <SwiftUI/SwiftUI.h> at the top of your Metal files to have proper reference to SwiftUI::Layer. Thanks!

---

Q: Does ToolbarItem support wrapping up a Menu object instead of a plain Button? I’m getting mixed results when used in a DocumentGroup app on SwiftUI.

A: You can put any kind of view inside of a toolbar item. Could you file a feedback detailing the bug you are seeing?

---

Q: I would like to integrate SwiftData with my SwiftUI app. I would like to ask that will .modelContainer(for: [Class.self]) init brand new database instant every time? Or it can detect the database creation status automatically? What if I have multiple modelContainer mods with the same instance class? I am wondering this because I my mind, the database should be initialised by a dedicated code, and I want to know that why use the modifier methods to create database structure.

A: The overloads taking model types are conveniences for quickly adding a container.
If you'd like to take full control over instantiating containers, there is an overload that takes a ModelContainer instance instead.

Follow-up-A: As Curt said, the modifiers are a convenience and you can create ModelContainer instances directly for full control. A few more answers to your questions:
The modifier will not create a new database every time, it will connect to the same persistent backing store.
If you use multiple modelContainer() modifiers with the same configuration, that's equivalent to creating and using multiple ModelContainer instances with the same configuration: they will resolve to the same, shared, persistent backing store, but will vend different model contexts and use separate in-memory caching state.
One example of where modelContainer() is helpful even in apps that create their own containers is in #Preview code, where you can conveniently configure a preview to have it's own, isolated, in-memory container.

---

Q: I’ve been having trouble with a common animation scenario and wondering if there’s a way to solve this in Xcode 15. The scenario is that I’m animating a container view onto the screen using a .move transition while simultaneously downloading the container’s data from the network. After the data is download, we display it in the container using an .opacity transition. The problem occurs when the data’s transition starts before the container’s transition is completed. One would expect the data to be moving along with the container while it fades in. However, the data fades in with its final geometry, completely disconnected from the container’s moving geometry.
We could wait to display the data until the animation is completed, but the app feels so much more responsive when the data appears as soon as possible. Is there a way to accomplish this in SwiftUI? I have feedback with a sample playground – FB12260978

A: Hi William – this occurs because animations occur at the level of the children within the container by default. When a new child appears mid-transition, it comes in at its destination position.
As a workaround, you can apply the transform(.identity) modifier to the container, which will cause layout animations to occur at the level of the container. This should give you the behavior that you are after.

---

Q: How can I make @Observable work with the UIViewReprestable API? Thanks, again!
The following UILabel does not update as textObject changes:

```swift
@Observable
class TextObject {
    var text = ""
}

struct UIText: UIViewRepresentable {

    var textObject: TextObject

    func makeUIView(context: Context) -> UILabel {
        UILabel()
    }

    func updateUIView(_ uiView: UILabel, context: Context) {
        uiView.text = textObject.text
    }
}

struct ContentView: View {

    @Bindable var textObject = TextObject()

    var body: some View {
        VStack {
            UIText(textObject: textObject)
            TextField("placeholder", text: $textObject.text)
        }
    }
}
```

A: Hi Jiayou, thanks for asking about this — this is currently a known issue.

---

Q: Is there any app delegate behavior that we should/or can move to App? App delegate behavior seems tacked on in SwiftUI (for example @NSApplicationDelegateAdaptor). Or are app delegates here to stay? Also, thanks for being here!

A: Hi - thanks for your question. Is there anything specific you had in mind? We do have some scene level modifiers for some of what app delegates do currently.

Follow-up-Q: state restoration and push notification handling immediately come to mind.

Follow-up-A: Ah, ok. For state restoration, is there something beyond what @SceneStorage offers that you are looking for? I'll also say that we'd appreciate any feedbacks in this area as well as the app delegate functionality.

---

Q: Is it possible to separate buttons (like with a Divider()) in the ellipsis menu that builds automatically on toolbars with customizable content? I can’t find the right way to do it. Thx!

A: Yes! You should be able to wrap your views in a ControlGroup which will render individual toolbar items but put them in a section in the overflow menu.

---

Q: Is there a way to configure TextEditor to have controls for formatted text? At current I have to wrap NSTextView to get that.

A: There's no support for formatting controls with TextEditor. But we'd appreciate your feedback detailing your expectations via the Feedback Assistant app.

---

Q: For Pie/Donut charts, what options are available for annotations and/or legends?  .overlay works, but isn’t great for narrow segments.  Is it possible to have annotations off to the side of the chart, with lines pointing to the segments (especially on wider devices)?

A: Linked labels are currently not supported. Please file a suggestion for this feature.

---

Q: I have a SwiftUI ScrollView with a VStack that has a lot of vertical content. Toward the bottom is a simple Map with no annoations. For whatever reason having the map in the hierarchy causes the first load of my ScrollView to be slow. I can resolve this by using a LazyVStack, but then I lose the ability to use a ScrollViewReader's scroll(to:). Is there a way to work around this? Some sort of lazy loading method I'm not thinking of

A: If you have a lot of content to scroll vertically, you can improve scrolling / loading performance by using a LazyVStack. ScrollViewReader does support scrolling to offscreen views but the id of the view has to be visible to the stack.

```swift
LazyVStack {
  ForEach(items) { item in 
    // ...
  }
}
```

For example, you can scroll to views with an id of any of the items id's in the above example.

---

Q: Is it possible for the Table to present data in Sections?  I have a Bill Tracking/Budgeting app that organizes data for the user in sections like ‘Overdue’, ‘Pending’, ‘Next Pay Period’, etc., and I think a Table view would be excellent on the wider devices, but I don’t think there’s a way to maintain the Sectioned layout (FB12272254)

A: Hi! If you use the rows parameter of the Table initializer to build your content, you can create sectioned rows!
An example would look something like:

```swift
Table(for: Person.self) {
    TableColumn("Given Name", value: \.givenName)
    TableColumn("Family Name", value: \.familyName)
} rows: {
    Section("Favorites") {
        TableRow(sam)
        TableRow(matthew)
    }
    Section("Contacts") {
        ForEach(contacts) { person in
            TableRow(person)
        }
    }
}
```

Q: In the docs there's an issue about views not updating when a dependency is inside a content closure. See also this post: https://m.objc.io/@lickability@mastodon.social/110503839533962620. It seems to work fine though. Are the docs outdated or is this behavior going to work as described in the docs?

A: Hi Chris. Those views will indeed update correctly. We will be updating the documentation to reflect that soon.

To be more specific, in that case the view will behave as if you had factored out that BookView yourself, but automatically, giving you the best performance by default.
Specifically, the List's closure is an escaping closure, so not executed within LibraryView's body property. This means LibraryView will not form any dependencies to book properties in that example. Instead, SwiftUI runs the list content closure internally when creating internal wrapper views for each individual list row, so those list rows will form their own independent dependencies to their respective book.title properties.
This means that if a specific book's title updates, the only part of the UI that will be reevaluated for that change will be the specific row view that displays that book, not the whole list! 

---

Q: What kind of architecture does the team use/prefer? MVVM or other? Is SwiftUI designed to work with any specific architecture?

A: SwiftUI is designed to work with a variety of architectures. Different members of the team use different architectures. I've recently enjoyed using SwiftData to build the model layer in my hobby apps.

Follow-up-Q: But, did you have any kind of considerations for any specific arch or, did you have any specific arch in mind when designing SwiftUI features?
UIKit had MVC in mind, that's why this question is important to me. And Furthermore, I have seen a few articles about how MVVM can be counterproductive to the core concepts of SwiftUI.

FOllow-up-A: There are a lot of opinions about app architecture, so we want to support as wide a variety as is reasonable. We don't want someone with a strong opinion about app architecture to feel like they can't use SwiftUI and have to revert to using a non-native UI toolkit.

At the same time we want to provide a turnkey solution for people that, among other things, makes it as easy as possible to get started. That's why we're so happy that we're finally able to share SwiftData

---

Q: Does the Swift charts team plan to support building custom chart views? Also what is the performance of swift charts when rendering data at scale?

A: What exactly do you refer to with "custom chart views"? Please explain or link.
Regarding performance, it's best to do a preliminary testing with the approximate quantity of data, and the type of interaction and device floor you plan to support, before iterating a lot on the design.

---

Q: What's the proper way of implementing multi-select in a List with value-based navigation?

A: In a NavigationSplitView you can put a List in the sidebar. Bind a Set as the selection to get multi-select. Then make the root of the detail column use that selection to decide what to show.

When the NavigationSplitView collapses to a single column, as on iPhone or iPad in Slide Over, it will push based on single selection. That’s the platform convention. You can add an EditButton to your toolbar to enable multi-select in that case.

Follow-up-Q: Is there a way to collapse the NavigationSplitView’s sidebar to TabBarView?
I.e. on compactSizeClass it can be in TabBar and in regular it can be in Side bar?

Follow-up-A: You can use the horizontalSizeClass environment value to switch between a NavigationSplitView in regular width and TabView in compact width.

---

Q: Can the new Metal shaders API for SwiftUI be used with SwiftCharts?

A: The foregroundStyle modifier on ChartContent accepts anything that conforms to ShapeStyle, so you should be able to provide a Shader.

---

Q: For Pie Charts is it possible for a segment to have additional dimensions of data?  For example, if your Segments are primarily based around expense categories (‘Home’ expenses, ‘Auto’ expenses, etc), could each of the pie segment be further split into ‘Overdue’, ‘Due This week’, ‘Due later’, etc?

A: How would the second layer of breakdown (partitioning) work in your specific case? Can you describe how it'd look? Do you refer to sunburst charts?

Follow-up-Q: I think each segment would be broken into multiple colors -- for example, the outer portion representing Overdue bills would be colored red, the next section in, representing Paid, but Uncleared bills would be colored yellow, and the internal portion of the segment, representing future bills would be colored green.  I use this today with the BarMark, and it works well -- I believe it could also work well in a Pie Chart.

Follow-up-A: You could indeed do it with coloring, which you described, because you can use foregroundStyle to specify colors directly. But bear in mind that it's generally not good practice to show a lot of sectors, because it's then next to impossible for the viewer of your charts to associate the sectors with the meaning (category).
We don't have direct support for sunburst charts. In theory, two charts with different innerRadius / outerRadius can be stacked, but it may or may not work out for you, for example, legends won't work.
Consider a main pie chart for the top level, and a list/grid of pie charts for the breakdown of each category, or let users interact with the main pie chart, where a tap would break down the tapped category in a secondary pie chart (or other visualization)

Follow-up-Q: I see - I could use the foregroundStyle(by:) modifier, and pass in PlottableValues representing the three different categories I want each segment to be colored by, as opposed to the PlottableValues used to segment the overall data?  That makes sense, but I also understand what you're saying about being difficult to visually tie each segment to a category -- my original thought was based on the ability to create an overlay that connects the Category names to the segments with a line, but I learned in a prior question that this is not possible.
I like the interaction model, and will explore that as well -- thanks for the detailed feedback!

---

Q: When working with @ObservableObjects, you can declare @Published var properties with initial values specified in the initialiser. Is it possible to achieve the same using the @Observable macro? It seems to result in compiler errors requiring an initial value:

```swift
// Compiler Error
@Observable final class Foo {
    var bar: String // @Observable requires property 'bar' to have an initial value (from macro 'Observable')

    init(bar: String = "") {
        self.bar = bar
    }
}

// No compiler Error
@Observable final class Foo {
    var bar: String = "" // <- The initial value is required here

    init(bar: String) {
        self.bar = bar
    }
}
```

A: This is actively discussed in the Swift community: https://forums.swift.org/t/pitch-init-accessors/64881

---

Q: I'm overjoyed to see the addition of isPresenting: @Binding to .searchable(). Is there any way to back-deploy that (or fake it) for iOS 16? The hacks I'm doing right now are... 

A: No, its implementation uses new logic that can't be back deployed.

---

Q: When creating a label if I want 2 lines of text and the icon centered to the text, I couldn’t find a way to do it without making a custom label, is there a better way to do it? (You can see that Apple does it settings for example in Settings > Notifications for each app it has the app name and under the app name it’s says “banners, sounds”) I filed a feedback FB12251896, but maybe I’m just doing it wrong

A: You can write a custom LabelStyle implementation that places the icon and content in a center-aligned HStack.

---

Q: I've only seen @Observable macro used with classes. Can it be used with structs or does this not result in anything?
For example, changing a property in a struct will cause the whole struct to change, therefore reloading any views that are dependent on the object, regardless of what properties it reads. I'm guessing the Observable behaviour of reloading a view only when a read property changes doesn't count for structs.

A: The @Observable macro is only used for reference type. To be more precise you should only use @Observable on classes.

---

Q: I am working on an iOS app that shows user ratings for movies/TV shows/etc. It is a perfect use case for CircularProgressViewStyle, but that isn't really supported on iOS. I know you can't speak about future plans, but could you give any insight why that is? Or am I holding it wrong? For what it's worth, FB12272960.

A: Hi Casey, thanks for submitting that feature request! You are correct that CircularProgressViewStyle does not currently support showing determinate progress in iOS apps, only indeterminate progress. I can't speak to future plans but this is definitely a request we've received before and that we are tracking.

---

Q: I have an @Observable class from which I use multiple properties in my views. In the same class, I'm also updating multiple variables that I don't use in any View. Should I somehow decouple the architecture or is this OK?
The variables I'm updating come from some motion data (like gyroscope), so it's really a lot of updates, however I'm just updating the variables that my Views use once in a while.

A: Variables not displayed in view bodies don't cause view view rendering. Your approach of selectively updating view-rendered variables sounds great!

---

Q: How to programming invoke refresh control inside a list otherwise shown using refreshable modifier?

A: This is currently not possible but it would be a great feedback to file.

---

Q: My question is about view modifiers added to items in lazy containers: should sheet/confirmationDialog/alert modifiers be directly added to items in a lazy container (List or ScrollView + LazyVStack) or added to the lazy container itself? For example, the swipeActions are directly added to the rows, and not to the List, which make me question about performance issues.
Use case: tap a row to edit an item in a modal sheet. We can imagine having a subview for the row that contains the sheet to present, or to attach the sheet to the List/ScrollView and pass the action to present the sheet to the row.
Is there any performance impact in replicating these modifiers for every row?

A: Thanks for the question!
In general it's best to apply the presentation modifier outside the lazy container.
If you apply it to an item in the container it won't trigger when the item is scroll out of view. Worse, it may trigger just because the user scrolled the item into view, depending on the associated state.

Interestingly, this issue is why we recommend that developers apply navigationDestination outside a lazy container, and why we deprecated NavigationLink(isActive:destination:) last year.

---

Q: What is the best way to scroll/focus an item in a horizontal list on tvOS after online data loading? In my code, it doesn't work 100% of the time and I don't know why.

A: Depending on whether you're using a lazy layout or not the solution could be different.
to change focus to a specific view you should use FocusState. This will work for non-lazy layouts, like HStack, etc.
you will have to scroll the view to be focused to visible first though if you're using a lazy layout, like LazyHStack, etc.

---

Q: Could you shed some light how Apple team achieved the background "area" mark that connect each sleep stage health's app sleep chart? I mean this one:
https://support.apple.com/library/content/dam/edam/applecare/images/en_US/Health/ios-16-iphone-13-pro-health-browse-sleep-history.png

A: This is not possible with Swift Charts today. Feel free to file Feedback with your use case!

---

Q: An scenario I’ve always found awkward and would like some assistance with in SwiftUI.
Assume I have some header text centered horizontally on the screen. But now I want to add a button or something else to the leading edge of the screen but I don’t want it to shift the header text off center.
I know I could use a ZStack so that they don’t affect each other but then if the header text gets long it can overlap and go under or over the leading button.
I know I could also put a translucent view on the trailing edge with the same size as the leading button but that feels hacky, and also prevents that trailing space from being used by the header if it gets long.
The best I’ve come up with so far is to wrap the leading button in a frame with infinite max width and leading alignment, put a spacer on the other side of the header also with infinite max width, and then give the header text layoutPriority(1). This seems to be the most robust and handles long and short headers and keeps it centered when it can be, but it also feels a little janky.
Just curious if there is a recommended way to handle this type of scenario. Maybe a custom Layout? But is there a more built-in way?

A: This feels like good place to use a custom layout since there's a bunch of special case rules (like using all remaining space when the header is long).

---

Q: Can shaders be applied to views meant for interaction, such as a form? Such as a CRT effect shader applied to a form for the settings in a game.

A: You can apply a shader in any place you can apply a foregroundStyle / tint! That said, certain controls which have a system-styled appearance may not respect the applied shader (a good example of this would be Toggle). To work around this, for controls that have a style protocol (such as ToggleStyle) if you implement a custom style fully in SwiftUI, it will support the shader!
So with some combination of custom control styles and the new shaders, you should be able to achieve something like this!

---

Q: Is it possible to style a .searchable search bar without resorting to global .appearance() modifications?

A: SwiftUI has no APIs to explicitly customize the appearance of the configured search field. I'd love a feedback detailing some of the things you'd like to customize though!

---

Q: Is it possible to use keyboardShortcut with a menu picker?

A: Hi Fredric. Unfortunately, keyboardShortcut is currently unsupported by menu pickers, but we’d appreciate a feedback request with more detail about why you’d like this capability!
You can learn how to file a request using Feedback Assistant by visiting https://developer.apple.com/bug-reporting/.

---

Q: I want to have an option in my app to change the tint of the whole app. What should I use? I tried to update the tint color in the root view but it is not working for places where I use the .accentColor (like a Text, a Shape, etc). Is there a way to update the .accentColor in the app or should I specify the color in every View that needs to use it?

A: You can specify the accent color for the entire app as an asset in your Xcode project as documented here: https://developer.apple.com/documentation/xcode/specifying-your-apps-color-scheme/

---

Q: Do you have any tips on using OutlineGroup on macOS? I have performance issues when creating an OutlineGroup with nested data of more than a few hundred tree items. It seems like there is a cost for items even if they are collapsed.

A: Hi Austin! Before going into details, I want to make sure if you were referring to OutlineGroup in List, or Table, or just OutlineGroup in a stack, and they have different implementation details so it is important to know which one we are talking. We have also published a video on optimizing performance which includes some of the best practices. https://developer.apple.com/videos/play/wwdc2023/10160/

FOllow-up-Q: Sorry, I was referring to an OutlineGroup contained in a List

Follow-up-A: Thanks for letting me know the details! I think the session covers most topics I could think of, and still, if after the video you still find OutlineGroup in a List to be not as fast as you want, we would appreciate a feedback with your use case and sample code so we could keep track of the issue!

---

Q: Hej! I really love custom view controller transitions (via UIViewControllerAnimatedTransitioning) - how do I approach building these with SwiftUI? For example, I'm looking to add a shared element transition between a view and the sheet it presents. Thank you!

A: Thanks for the question!
SwiftUI doesn't currently support the full power of UIViewControllerAnimatedTransitioning. I'd love a Feedback with your particular use case to help prioritize future work.

---

Q: In the session on interactivity in Swift Charts you have an interactive chart that shows the sales the last 30 days so you set chartXVisibleDomain(3600 * 24 * 30) and snap to the first day of the month when swiping using the chartScrollTargetBehavior() modifier.
How would we go about showing all data for an entire month, no matter if that month is 28, 30, or 31 days? It seems that’s what the Health app does when scrolling the chart while viewing data for a month.
I’m thinking we can’t use chartXVisibleDomain(3600 * 24 * 30) since the domain is variable. So what can we do instead to replicate the behavior of the Health app?

A: Assuming a fixed screen range for the plot area (in points/pixels) and snapping to the beginning of the month, it is not possible to vary the number of bars within the visible plot area, because the bar count would vary between 28 and 31. So it'd require some kind of autozooming to adjust the "zoom magnification".  It is a bit questionable interaction-wise, because the user would scroll, and as the chart settles, it'd change the zoom level.
You could perhaps achieve what you want by using a 31-day window, and coloring months differently, or using gridlines at month boundaries

---

Q: With SwiftData and Observation, what is the difference between @Bindable and @Binding? Should we only use @Bindable when we pass a @State to a subview that needs right access to the variable (like a Bool, Date, or String for simple types)?

A: Hi Axel! @Bindable is a utility to allow arbitrary @Observable objects to be able to create bindings to any of its properties.
You can store an @Observable object in @State, and create bindings to it using the state $ prefix operator syntax, without needing to use @Bindable at all.
However, there are some cases where you need to create a binding to an object's property, but you only have a reference to the object, and it's not appropriate to store it in @State. @Bindable is there to provide that capability.

Bindable and Binding are separate concepts, serving different purposes:
Binding is a read-write connection to a particular piece of data.
Bindable is a utility for creating Bindings from @Observable object references, and does not apply to non-object data types.

To compare to ObservableObject, SwiftUI has the @ObservedObject property wrapper to declaring dependencies to ObservableObject instances in your view. @ObservedObject also provides a way to get bindings to properties of those objects.
With @Observable, you no longer need to use a property wrapper at all for views to track changes to those object's properties. However, there is still a need to create bindings to object properties. @Bindable provides that specific capability for @Observable objects, replacing @ObservedObject.

---

Q: What is the recommended way to have a view that updates when it's parent ScrollView's offset changes?
Two approaches we've tried are
* Grabbing the underlying UIScrollView/NSScrollView and reading contentOffset
* Putting a zero-height view at the top of the ScrollView and reading it's minY with a GeometryReader
Neither one of these feels ideal.

A: It depends what you are trying to do. If you'd like to visually alter a view based on its relative position within the scroll view's visible region, you have a few new APIs available to you.
scrollTransition() good for common operations like opacity, rotation, offset. specific to ScrollView:
https://developer.apple.com/documentation/swiftui/view/scrolltransition(_:axis:transition:)
visualEffect() good for deeper customizations based on a GeometryProxy, useful in ScrollView and other contexts:
https://developer.apple.com/documentation/swiftui/view/visualeffect(_:)

---

Q: Is there a way to change the background color of the searchable modifier that is in navigationBar? I have my toolbarBackground set up as a color but my searchable blends.

A: There are no APIs to configure the appearance of the search field configured by a searchable modifier but feedbacks detailing your use case would be appreciated!

---

Q: I’ve discovered a use case where the width of the sidebar is not persisted when quitting the app.
This seems to happen for both NavigationSplitView and NavigationView when the detail view has a frame set.
If the app is restarted via Xcode the width is persisted as expected.
This has been reported in FB12196768.
Is this a potential bug or am I doing something wrong?

A: This is a bug. I've screened your feedback and we'll do our best to address this quickly.

While it may not fit your use case, as a temporary workaround, omitting the binding argument altogether allows proper persistence.

---

Q: AppKit apps have the ability to indicate that a window has been edited (by putting a dot in the close button and adding "— Edited" to the title), and also asking for confirmation if the user attempts to close the window while there are unsaved changes.
Is it possible to do this with SwiftUI and/or Mac Catalyst apps?

A: That behavior can be achieved by using a DocumentGroup scene type in a SwiftUI app.
You can see more about that, including sample-code, here: https://developer.apple.com/documentation/swiftui/documents
If there is additional behavior or functionality that would be useful in this area, a Feedback report would be appreciated!

---

Q: Hey! I was wondering whether I'm using the keyframe animation API correctly. I wanted to move between two states. State A has offset (0,0) and state B has offset (150,150). It animates fine when I go from A to B, but tapping again has a weird glitch. Am I using the API wrong or is this a bug? Here's the code I tried out: https://gist.github.com/chriseidhof/f2506dc54363e7204deeef9837b90dad

A: Thanks for the question!
I wonder if the initialValue in your example needs to be predicated on the value of active?

---

Q: When I have a frame modifier that's not scoped to be within an animation, it often still animates. For example in this gist: https://gist.github.com/chriseidhof/b7aacc4dd971ea81e9973fa0b4ab0c77. Is this considered a bug? I have tested the new .animation(_:body:) API which works great as a workaround, thank you for that. (And also for custom transaction keys, very helpful).

A: This is expected behavior. Sizing and positioning in SwiftUI happen at leaf elements (i.e. the Color in this example). This means that when a layout modifier like .frame is applied, its effect isn't applied until the leaf views it affects. This means that the transaction at the leaf view will be what determines how the layout change animates.

---

Q: Can SwiftCharts be brought into RealityKit? Would it be possible to have a 3 axis, 3D in RealityKit (let’s say X and Y represent lat and longitude while Z represents temperature)

A: You can use Swift Charts to create 2D charts and use it like any SwiftUI View when it comes to AR. But we don’t have the right expert on RealityKit available to answer this. Please ask on https://developer.apple.com/forums/

---

Q: When using @Observable, if one of the properties is a struct, how will that behave when changing the properties inside (for instance title inside Item below?
e.g.

```swift
@Observable class Model {
  var item: Item
}
struct Item {
  var title: String
  var description: String
}
```

A: Because struct have value semantic, when you mutate a property of the struct you're effectively mutating the whole struct.
In your example if you mutate the item's title, Model will publish a change for it's item.

---

Q: Is there a way to "remember" the position of a ScrollView in a View to scroll to that position after the user dismisses the View, with @SceneStorage maybe? I'd like to scroll to that position on the user's next time in that View.

A: You can use the new scrollPosition() modifier along with the scrollTargetLayout() modifier to accomplish this. Just ensure the binding you provide has a non-nil initial value and the ScrollView will ensure the view with that id (within the layout configured with scrollTargetLayout()) is visible when the scroll view appears.
See the docs for more information here:
https://developer.apple.com/documentation/swiftui/view/scrollposition(id:)

---

Q: Is it possible to apply matchGeometryEffect between sheet presentation or push/pop navigation?
If not, what should be the approach to achieve such effect?

A: Thanks for the question!
This is not currently supported. Please file a Feedback with your specific use case. That really helps us prioritize future work.

---

Q: #Preview macro doesn't work if the project targets previous iOS versions, is this the final behaviour or will you add support to make it version agnostic?

A: The #Preview macro is not backdeployable in Seed 1. Feel free to file Feedback!

---

Q: Is there any way to have the Y-axis labels update in a scrolling chart as the chart is scrolled or when the scrolling settles after snapping? This behavior would be similar to what the Health app does where the Y-axis reflects the minimum and maximum of the currently visible area.

A: You would currently need to update (with animation) the Y-axis yourself by setting the  Y domain according to the temporal window the user settled on, presumably with a debounce, so it activates some hundreds of milliseconds after the scroll interaction ended

---

Q: Hi. Is there any possible way to detect when user finishes scrolling a ScrollView? (Similar to UIKit UiscrollView delegate scrollViewWillEndDragging).

A: There is no SwiftUI API that provides this functionality though I'd love a feedback detailing your use case.

---

Q: What is the recommended way to pass data from child to parent view in SwiftUI? I underderstand this could be done using PreferenceKey, but I wonder what alternatives there are and what is recommended

A: Thanks for the question!
A classic engineering answer: it depends!
Preferences are a reasonable mechanism in some use cases, though you need to be careful to avoid cycles.
Creating some state in the parent view and passing a binding to the state down to the child is also a good technique sometimes.

A bit more on cycles with preferences: Suppose the preference comes up and changes the parent. Then suppose the parent changes the child. If that causes the child to send a different preference value on the next update, then a cycle will occur.

---

Q: I really love the look of the new ScrollView enhancements this year! Is there a way to get them to work with Lists as well?

A: APIs like scrollPosition() and scrollTransition() currently only work with ScrollView though I'd love a feedback detailing your use case with List

---

Q: Can you explain, when we should use @State with an @observable class?

A: You can store @Observable objects within @State properties, when the view needs to create and own the lifetime of a particular object instance.
ObservableObject required the use of @StateObject to handle dependencies to object-based state correctly. With @Observable, you can just use @State for the same purpose.

Follow-up-Q: So we use @State instead of @StateObject, and just a plain reference instead of @ObservableObject?

Follow-up-A: Correct! 

With @Observable, the experience is much closer to what you write today when just struct-based state.

---

Q: I've been trying to implement .focusable in my Ventura macOS app. Unfortunately something seems broken as I can't get the focus ring to appear when I apply .focusable on any view. Do I need to apply something else to get it to work?

A: Prior to macOS Sonoma, the focusable() modifier only has an effect when the user enables keyboard navigation in System Settings. There's no way to configure this to be otherwise.
We changed this in Sonoma to be more flexile:
* focusable() now makes your view focusable whether or not keyboard navigation is enabled.
* To get the old behavior, you can specify that your control only wants focus for "activation" interactions. The system will treat the view more or less like a button for focus—never focusable on click, only focusable on tab with keyboard navigation enabled.
TapGestureView()
    .focusable(interactions: .activate)
We're releasing a talk tomorrow that covers this (and some other common focus questions) in a little more detail: The SwiftUI Cookbook for Focus.

---

Q: Regarding San Francisco's “high legibility” alternate style set: since there isn’t a proper font modifier for this, is it safe to assume using the stylistic set key is shelf-stable? (I think it was kStylisticSetSix or something)

A: Please file a Feedback. Meanwhile you can create a CTFont requesting this font feature and then use that to create a Font.

---

Q: How can I make a reversed scroll view?
I’ve seen some hacks like rotating the view but it messes with the scroll indicators.
Would it be possible to use some of the new APIs to implement that?

A: The scrollPosition() API might help you here:
https://developer.apple.com/documentation/swiftui/view/scrollposition(initialanchor:)
There’s an initialAnchor variant that you can use if you don’t care about observing changes to what view is currently visible.

```swift
ScrollView { ... }
  .scrollPosition(initialAnchor: .bottom)
This will tell the scroll view to start scrolled to the bottom and try to stay scrolled to the bottom until the user scrolls.
If you do care about the state changes, you can use the id version and ensure the binding has an initial value to the last element in the lazy stack.
ScrollView {
  LazyVStack {
    ForEach(items) { ... }
  }
  .scrollTargetLayout()
}
.scrollPosition(id: $position)
```

---

Q: How can I make a reversed scroll view?
I’ve seen some hacks like rotating the view but it messes with the scroll indicators.
Would it be possible to use some of the new APIs to implement that?

A: What do you mean by a "reversed scroll view"?

Follow-up-Q: Like in a messaging app,
You start scrolling from the bottom rather than the top

Follow-up-A: The scrollPosition() API might help you here:
https://developer.apple.com/documentation/swiftui/view/scrollposition(initialanchor:)
There’s an initialAnchor variant that you can use if you don’t care about observing changes to what view is currently visible.

```swift
ScrollView { ... }
  .scrollPosition(initialAnchor: .bottom)
This will tell the scroll view to start scrolled to the bottom and try to stay scrolled to the bottom until the user scrolls.
If you do care about the state changes, you can use the id version and ensure the binding has an initial value to the last element in the lazy stack.
ScrollView {
  LazyVStack {
    ForEach(items) { ... }
  }
  .scrollTargetLayout()
}
.scrollPosition(id: $position)
```

---

Q: It seems like decorative gradients are a big trend this year — any specific pointers on creating nice, tasteful gradients?

A: The builtin color gradients (e.g., Color.blue.gradient) all look great. There's also a Meet the Presenter for the Design with SwiftUI session, and they may be able to provide further guidance.

To add on a little to this: It’s hard to give generally applicable advice here aside from those system provided gradients because it’s all about your design sensibilities here. Areas like this are where you can bring your own special touch to your apps, so play around with different color combinations and stops and see what looks great to you! 

---

Q: Is there a way to get the transaction within an onChange(of:...) callback? This would be useful for when I'm animating something there (and use the same animation for my new change).

A: There's currently no way to do this, but it does sound like it would be useful. Could you file a feedback with a request for this?

---

Q: Apparently, we can use the @Observable macro together with ObservableObject, but it throws an error when trying to compile: "'ObservationRegistrar' is only available in iOS 17.0 or newer"
This is the chunk of code:

```swift
@Observable class Test: ObservableObject {
    var name: String? = nil
}

struct ContentView: View {
    @ObservedObject var test: Test
    
    var body: some View {
       Text("hola")
    }
}
```

A: Hi Luis, @Observable is only available for use starting in iOS 17.

---

Q: When using a NavigationSplitView is it possible two have either a two-column or three-column layout depending on a condition? For example, in Notes it's three columns when viewing in a list and two columns when viewing as a grid.

A: Thanks for the question!
The best way to do this today is to switch between a two- and three-column NavigationSplitView inside an if.

If you do this, be sure to lift any state above the view containing the if, as changing the condition changes the view’s identity.

---

Q: When using @Query to fetch from SwiftData in a SwiftUI view, are the sorting and predicates dynamic like they are with @FetchRequest for CoreData? I'm having a hard time setting them programmatically in a view.

A: If you need to update your predicate or sorting criteria, pass them into the view that your query resides. Let's say you need to change the sort order of recipes, the related code might look something like

```swift
struct ContentView: View {
    @Query var recipes: [Recipe]
    init(order: SortOrder) {
        _recipes = Query(sort: \.name, order: order)
    }

    var body: some View { ... }
}
```

---

Q: Elaborating my previous question, I have several UIKit views that I use in SwiftUI through UIViewRrepresentable.
When I use those views out of a ScrollView, they size correctly.
When I place them inside a ScrollView, they vanish, unless I set a height for them.
Is this a known issue? How can I fix this?

A: You can implement sizeThatFits in your representable view to provide sizing information.

---

Q: I noticed that a lot of sample code uses var instead of let for read only properties of Views. Is there any difference? I usually use let in my structs unless I explicitly need mutability.

A: Prefer to use let unless a var is needed, for example for mutation, or computed properties

---

Q: Hello! A question about my project. l'm sorry if it was answered already.
Is it possible to bind directly to CoreData objects? Publishing them from a VM.. I remember running into issues with all of the optionals.

A: NSManagedObject conforms to ObservableObject, so you can use one in an @ObservedObject property and from there get bindings to its properties.

---

Q: When moving to @Observable from @ObservableObject, can you still use @Published, such as in cases where you want the @Observable type to have Combine-related functionalities in addition to updating SwiftUI views?

A: No, @Published only works when applied to properties contained in a type conforming to the ObservableObject protocol

---

Q: Are there any recommendations to display list's with large datasets with SwiftUI?
For example, I'm using now:
1. CoreData classes which load their properties lazily
2. These classes are wrapped in 'descriptors' classes that generate  all fields required to split content into different sections(like id, startDate) while the rest of the content is loaded on demand
3. UITableView prefetching mechanism is used to trigger this load
4. Dataset is around 20k elements

A: We just published a talk on SwiftUI performance that discusses a lot of important concepts around making large lists performant. I hope this will help, but feel free to come back with more specific questions! https://developer.apple.com/videos/play/wwdc2023/10160/

Follow-up-Q: Thanks! Yes, I’ve checked that one out already. If my assumptions are correct, as far as identity is provided correctly essentially the same approach as I’m using now should work.
I’m curious if I should be concerned in any way with prefetching, though. Would SwiftUI generate row content quickly enough to achieve the same as UITableView prefetching does?

Or rather, would SwiftUI trigger row content generation at the same time as UITableView prefetching allows us to or would it be similar to cell dequeuing?

Follow-up-A: SwiftUI does not offer a data prefetching facility like you'd find in UITableViewDataSourcePrefetching, but it will construct views for cells in advance, at its discretion

---

Q: Just wanted to ask if this behaviour was intentional or not: disabled buttons in alerts don't show at all. Say you had a text field in the alert and only wanted the "Done" button to be enabled when the text field was not empty, the done button wouldn't show, only a cancel button.
I filed a feedback report anyway, FB11558215, because I'm pretty sure this used to work before.

A: Thanks for the feedback!
This was not an intentional change and we're aware of the issue.

---

Q: I was wondering why the .preference-based APIs use a default value for the key (key: K.Type = K.self). Is there any case in which this is used? I have always found myself specifying the key explicitly. OTOH, for the new @Environment initializer this could be a useful thing to add, so you can do @Environment private var book: Book instead of @Environment(Book.self) private var book.

A: Hi Chris, thanks for this feedback. We're always looking at ways to make the syntax nicer and more intuitive.
One thing to point out for environment objects w/ @Observable, is that you can get both optional and non-optional properties, and also define your own properties with custom keys:

```swift
@Environment(Book.self) private var book: Book // default
@Environment(Book.self) private var book: Book?

@Environment(\.myFavoriteBook) private var book: Book?
```

One benefit of specifying Book.self explicitly is that it helps clarify that it's a key, like \.myFavoriteBook. So the first two properties above will access the same underlying storage, because they share the same Book.self key, whereas the third property is accessing different underlying storage, even though it has the same return type, because it uses a different key.

---

Q: I have built my iOS app with widgets in XCode 15, and suddenly all homescreen widgets have huge padding (I previously used to draw them myself, edge to edge, but now whole my ‘edge to edge’ view is inside the widget’s safe area, including the background). How can I tell SwiftUI not to do this? .ignoresSafeArea(.all) on a widget doesn’t help.

A: By default widgets now receive a default padding; you can disable that using .contentMarginsDisabled() modifier on your widget configuration. But there's probably some cases where you want to use the default content margin that we provide. For that you can combine it with \.widgetContentMargins. For much more detail about this you can watch https://developer.apple.com/videos/play/wwdc2023/10027/

---

Q: As far as I know, Pickers does not load their content lazily. When I have a picker with 100 items, all the elements are accessed right away. In UIKit, UIPickerView were lazy so we could have many items. Is that a bug or a feature?

A: Hey Axel, thanks for the feedback. This is a known issue and we are working on fixing it, but we’d appreciate a feedback request with more detail about why you’d like this capability!
You can learn how to file a request using Feedback Assistant by visiting https://developer.apple.com/bug-reporting/

---

Q: What is the best way to animate a parent when a child view’s frame changes in an animated way due to a change in the child view. Today the parent will not animate in this case, it just jumps to accommodate the new size of the child even though the child is animating its size change. I came up with this answer but I’m wondering if there is a built in way to do this: https://stackoverflow.com/a/76195425/9156195

A: It seems like you want withAnimation(_:_:), which will animate all of the changes that are triggered by its body without needing the .animation modifier.

---

Q: What is the recommended way to stop an animation that is repeating forever. Essentially looking for a repeat(while:) type of thing. Several different janky solutions online but none of them have worked consistently over multiple iOS versions so I don’t think any of them were “intended”.

A: Using the new CustomAnimation protocol, you can create an animation that can use values from the environment where the animation was created. This lets you build a custom animation that will repeat forever until an environment key is set in the view hierarchy.

---

Q: What is the cleanest alternative to multi columns pickers? Like to choose hour/minutes/seconds to configure a picker for example, or to choose the decimal and float part of a number (distance for a running app, height, weight, etc).

A: Hey Axel, thanks for the question.
I think this really depends on your specific use-case and design language. navigationLink styles could be a nice alternative. Is there any specific reason why multi-column pickers aren't a good fit for your use-case?

Follow-up-Q: Thanks. Yes, the use cases I described. How to you select two or three values in a NavigationLink? Like my height in meters, and centimeters. My weight in kilograms and grams. A duration in hours, minutes, seconds. I remind you that we don’t have the UIDatePicker.Mode.countDownTimer in SwiftUI.

Follow-up-A: In the navigationLink case you'll need one picker per unit. If you are requesting a control to pick multiple items, please could you file a feedback?
A workaround could be to use a menu of toggles. That would render as a menu picker with multiple selection enabled. Would that help your case?

---

Q: In SwiftData, can I create a custom query using custom SQL or similar? Some of the use cases may include UNION or CTE queries

A: Not using SQL, no. The full span of customization for a query is represented by the configuration properties of the FetchDescriptor type in SwiftData.

---

Q: It's currently not possible to present a sheet, fullScreen, confirmation dialog, alert, etc. from a Menu. It this a bug or a feature?

A: You can trigger sheet/etc. presentation from a Button within a Menu. Make sure you're attaching the .sheet modifier outside of the Menu content. If that's not working as expected, please file a feedback with more information about your use case!

---

Q: Is there a way to increase the total height of a .wheel Picker style? It doesn't seem possible to achieve a like-for-like swap from a UIPickerView in UIKit. The UIKit version displays more items on the wheel than the SwiftUI version; even when their parent views are the same size.

A: Hey Jack, thanks for the feedback, unfortunately, there currently isn't an API to set the height of a wheel picker item  but we’d appreciate a feedback request with more detail about why you’d like this capability! Having parity on both frameworks is very important to us.
You can learn how to file a request using Feedback Assistant by visiting https://developer.apple.com/bug-reporting/

---

Q: How can i make my app suitable for all device sizes ?

A: SwiftUI automatically makes apps look great on all screen sizes. Is there a specific case where you'd like to tailor your app's UI even further for specific device sizes?

---

Q: Is creating Bindings in a View's body a bad idea?
For example I have a 3rd library that I can't control, that provides a state model with username field and class for events like setUsername(String). is it ok to do

```swift
@StateObject var viewModel = MyViewModel()
...
TextField("", text: .init(get: { viewModel.state.username }, set: { viewModel.events.setUsername($0) })
```

where MyViewModel.state is @Published field.
What would be the best way to go about this?

A: Not knowing the details of this third-party library, it's not easy to come up with a best solution. That said, here's a pattern I personally enjoy: treat the third-party data source as a service, and de-couple it from your view's state model. This way you gain the option to create instances of your view models that may not depend on the third party, and you can use it to test your views in automated tests or previews.

---

Q: I'm building an app that shows a List on iPhone and a LazyVGrid with two columns on iPad and on Mac. Is there any documentation that lists differences between these two containers, for example I know that swipeActions aren't available outside of List. Any other gotchas I should know?

A: LazyVGrid is a layout primitive, while List is a featureful collection view, so most things you get "for free" with List need to be re-implemented in one way or another when building with a lazy grid/stack.
In other words, it's like comparing VStack and List—they both present a vertical array of views, but List has behavior and semantics that VStack lacks.

Follow-up-Q: Does List load views lazily? I’ve used LazyVGrid for a single-column list with a large number of items for its lazy loading.

Follow-up-A: Yes, List loads its row content lazily.

---

Q: Is there a way to resize the columns in a navigationstack on iPad?

A: Thanks for the question!
iPad doesn't support user-driven column resizing. You can use the navigationSplitViewColumnWidth modifier to set a custom width.

---

Q: I'm trying to use
 .containerBackground(for: .navigation)
on a form. I get the compile-time error "'navigation' is unavailable in iOS". Same for all ContainerBackgroundPlacements, like .widget or .tabView.
What containers does this work with?

A: .navigation and .tabView are only available on watchOS 10.0. For iOS/macOS/watchOS there's .widget when used with WidgetKit. There's also a few other container background placements related to some of the new StoreKit APIs.

---

Q: Are expanding sections only available with the sidebar list style?

A: Hi! You could control the expand state with the new programmatic expandable Section support with any list style, detailed here https://developer.apple.com/documentation/swiftui/section/init(isexpanded:content:header:)-561d7?changes=latest_minor

Follow-up-Q: Hmm. It seems it only works for sidebars.
For example this doesn’t show the arrow:

```swift
List {
    Section("Header", isExpanded: .constant(true)) {
        Text("Text")
    }
}
.listStyle(.insetGrouped)
```

Follow-up-A: Thanks for testing that! That is probably not intended behavior - could you please file a feedback so we could keep track of this issue?

---

Q: Using a compact date picker and trying to present an alert on the parent view, the alert will not show if the datepicker is still open, is there a way around this? or is this intended behavior?

A: Hey Naftali, this seems like it could be a bug, we’d appreciate a feedback request with more detail about why you’d like this capability!
You can learn how to file a request using Feedback Assistant by visiting https://developer.apple.com/bug-reporting/

---

Q: What is the best way to use State and Binding mixed in a child view? For example, if I pass a binding value in initializer, the view must use it. If binding value is nil, the view must create a state and use it. I created a property wrapper for this, and it works. But actually I didn't like it at all. I want to know if there is a better solution for this.

A: Hi Alperen, that's the general solution we'd recommend for that. If you have ideas for making that workflow easier, please file feedback and let us know!

---

Q: Is there a way to achieve floating view in ScrollView, like supplementary view in UICollectionView?

A: Yes, the new visualEffect() API can accomplish this:
https://developer.apple.com/documentation/swiftui/view/visualeffect(_:)

---

Q: Does the new @Bindable in SwiftUI support nested changes? For example, I understand that $friend.firstName would update if firstName changes, but would $friend.pet.name also update if name changes?
Also, what about nullables, such as ``$friend.pet?.name`? This has historically been a huge pain.

A: Yes, all of that works!

---

Q: When using NavigationStack w/ .navigationDestions, when I want to push a navigation value that represents "editing an object", where should ownership of that model-being-edited lie? I've found that if I push it onto the navigation stack, that works well. But if I push a description of it onto the stack, and then have the view instantiate the model during initialization, that view-owned model is re-instantiated many times (on each render). It seems odd to push the model-being-edited onto the Navigation stack. What am I missing? This has been my top issue/question with Navigation Stack. Thank you!

A: Thanks for the question!
I'm not 100% sure I understand. This might be a good question for a lab session.
In general, either approach should work. Another approach is to use a data store and look up the new model by ID, creating a new instance of the ID doesn't exist.

---

Q: With SwiftData is there a way to create a sectioned fetch request in the view?

A: This is not currently supported

---

Q: I have a Text with paragraphs of strings. It seems the user can select only the entirety of the Strings in the Text view. Is it possible to let the user select a portion of the contents of the Text view?

A: This is not currently possible on iOS / iPadOS. Please file a Feedback.

---

Q: Using @Observable, what is a use case where one might still need to use @State to reference a view model, as opposed to directly referencing the view model itself?  The sample video says that @State is still practical in a case where only the view needs to reference a property, but I was kind of unclear on why someone would ever use @State instead of the view model handling all of the logic?

A: This depends on how you would like to arrange the ownership of your data. Without @State, your Observable model instance must come from somewhere else, the view would not be the owner of this instance. Sometimes you want your view to be the logical owner of some data (such as a boolean for whether a sheet should bet shown, but realistically likely more complex to warrant a class). @State helps you achieve that.

---

Q: I see from an earlier response that you can adjust a predicate for a @Query from the init() is this the recommended way to handle searching where as you type your updating a child view  init() with the updated value for the predicate

A: Correct!

Follow-up-Q: Can you provide an example?

Follow-up-A: Here's an small example that updates the sort order of the query I used to answer a similiar question. You should be able to adapt it for updating parts of a predicate:

```swift
struct ContentView: View {
    @Query var recipes: [Recipe]
    init(order: SortOrder) {
        _recipes = Query(sort: \.name, order: order)
    }
    var body: some View { ... }
}
```

---

Q: I have IAP in my app, and I limit certain features based on if the user purchased premium. How can I make it so that while I'm working on the app I can use those features in the simulator/preview while ensuring that it wont by mistake end up in production?

A: This should be supported by StoreKit, but we're not the right people to help you further. StoreKit and In App Purchase is running a Q&A tomorrow at noon in the 
in-app-purchase channel! They're the real experts in that area and they'll likely be able to answer your questions more precisely.

---

Q: With the new containerRelativeFrame API, what are the actual containers that are supported? The documentation is a bit unclear ("including"). Can we modify our own views to become containers?

A: You can use the components listed in the docs as the supported "containers". Its not possible to make an arbitrary view a "container" with respect to container relative frame though I'd love a feedback detailing your use case.

---

Q: Do LazyVStacks leverage cell reuse behind the scenes? I really want to use the new ScrollView APIs (that aren't compatible with List) but am worried about performance when using ScrollView with LazyVStack in the context of infinite scrolling (eg. such as a social media feed).

A: Thanks for the question!
LazyVStack does not currently have cell reuse. In general you'll want to make the data associated with items in a LazyVStack as lightweight as possible.
Often this can be accomplished by minimizing the state created by the item and having it reference a separate data model.

---

Q: Is it possible to add multiple shortcuts to a single user interface elements? If I have multiple .keyboardShortcut()s attached to a single view, only one of them fires. (Also related, is it possible to attach keyboard shortcuts directly to an action, rather than requiring, say, an invisible button?)

A: Controls can only support one shortcut currently, as you've found. I'm track a request to add that support internally, and I'll consider this another vote of support.
Buttonless shortcuts also aren't possible, but there are a few other possible work-arounds:
* First, should there be a menu item for this function? If so, you can use the Commands API to add a menu item with a shortcut.
* Second, if you have focusable content in your scene, you can react to key presses using the onKeyPress modifier, new this year.

---

Q: What's the proper way to open a ScrollView at the bottom (like in Messages)?

A: You can use the new scrollPosition modifiers to set the scroll position!

---

Q: Is it possible to set the alert style with SwiftUI for macOS?
I want to present a Critical style alert, but I can’t find any way to configure this. I don’t want to drop back to using NSAlert if it can be avoided!

A: Hi - thanks for the question. You can use the new dialogSeverity() modifier for this. For example:

```swift
.alert(...) {
    ...
}
.dialogSeverity(.critical)
```

This modifier is available on macOS 13+

---

Q: I am working on building a sample app that uses the new SwiftData framework and the new @Bindable type.  In the past, one could use .constant when compiling a preview that injected a mock property to make the preview usable.  Is there something similar for @Bindable?

A: You can construct one of your model objects in the preview and pass it into the view and to use to initialize the Bindable property.

Follow-up-Q: So, for previews, I no longer need to use .constant() to inject a mock Bindable property?

Follow-up-A: That's correct. A @Bindable property is akin to a regular property without a property wrapper, you can initialize it to a regular value.

---

Q: Follow up to a previous question -- when I use a Table with the Table(of:columns:rows) initializer, and I define Sections with custom headers in the rows section, the custom header is ignored for the first section, while it is properly displayed for all subsequent Sections.  Is that expected?  Is there a way to provide a custom View for the header of the first Section?

A: Hi! We need more detail on this - did you see the header for the first section in the table header region? Is there a screenshot for us to better understand what it is now vs what is your wanted behavior?

---

Q: I have struggled with NavigationSplitView in my macOS apps, and safely sharing state/nullifying state with content/detail columns and child views. Aside from last year's SwiftUI Cookbook for Navigation, are there other good examples or learning resources?

A: Hi, I need a little more context around what you mean by nullifying state, or sharing state. The cookbook is a great resource, but if it doesn't speak to your use case, or you require some more nuance to managing selection, I'd recommend asking on the developer forums. I'd be happy to try to clarify any questions you might have right now though

Follow-up-Q: I tend to have a lot of crashes relating to deleting things in content/detail view while still holding a reference to them elsewhere in the view stack. Another thing I can't quite wrap my head around is how to mix navigation types in my "sidebar" (first column) - i.e. value-driven navigation + some other navigation destination. I would find it helpful to see - in general - more approaches to NavigationSplitView on macOS to try to get a better conceptual state for what I'm doing incorrectly.

Follow-up-A: The crashes around deleting are interesting and I've definitely seen less of those kinds of bugs. For the second question, are you talking about a construction like:

```swift
struct SomeView: View {
  var body: some View {
    NavigationSplitView {
        List {
            NavigationLink("Value-based", value: 4)
            NavigationLink("View-based") {
                Text("A more, single use destination")
            }
        }
        .navigationDestination(for: Int.self) { int in
            Text("View for \(int)")
        }
    } detail: {
        Text("Detail")
    }
  }
}
```

This example makes use of a concept we call "root-replacement".
Both of those NavigationLinks above will target the detail column (if I had a content column they would target that, not the detail column)

This means that the presented view will be shown in the detail column. Generally, we almost always advise using the value-based NavigationLinks

This is because view-based NavigationLinks by nature provide less programmatic control, and if you have a lot of them, state restoration depends on them being scrolled into view, so will not work properly if your navigation links are off screen

Follow-up-Q: Ahh, that in itself is helpful! One of my WIP apps has a List where I can select a value and navigate based on that, but I also have some views that don't have associated values and I wasn't sure how best to mix and match those in the same column. I thought. the List could only contain homogenous value-type destinations.

Follow-up-A: Another tip: In general right now, we do not recommend mixing List selection and .navigationDestination. In a construction like this

```swift
struct SomeView: View {
    
    @State var selection: Int? = nil
    var body: some View {
        NavigationSplitView {
            List(selection: $selection) {
                NavigationLink("Value-based", value: 4)
                NavigationLink("Value-based", value: 5)
            }
        } detail: {
            Text("Detail")
        }
    }
}
```

The links' values will drive that selection state. If you tried to use a navigationDestination  in this case, that leads to some ambiguity. So, for the time being, use one or the other.

I love that particular use case because that is when the navigation system can really shine.

There are a few ways to spell that, allow me to suggest another

Follow-up-Q: I was doing something like this:

```swift
NavigationSplitView{ 
...      
            NavigationLink(
              destination: MainDashboardView(userProfile: profile)
              .environment(\.realmConfiguration, realmConfiguration)
            ) {
              HStack(alignment: .center) {
                Image(systemName: "rectangle.on.rectangle")
                Text("All Repositories")
            }.buttonStyle(.borderless)
            List(profile.watchedRepositories, selection: $navModel.selectedRepository) { repository in
              let badgeCount = getRepositoryBadgeCount(forRepository: repository)
              NavigationLink(value: repository, label: {
                Label(repository.customName ?? repository.name, systemImage: "rectangle")
                  .badge(badgeCount)
              })
            }
            .frame(maxHeight: 250)
            .navigationTitle("Repositories")
            NavigationLink(
              destination: IgnoredPullRequestsView()
            ) {
              HStack(alignment: .center) {
                Image(systemName: "eye.slash")
                Text("Ignored PRs")
                Spacer()
              }.padding()
            }.buttonStyle(.borderless)
```

Which unsurprisingly doesn't work great. Your example is quite helpful! So in my case, I had a List where I could drive navigation by selection of values, but also other views where there isn't an associated value

Follow-up-A: That is a totally reasonable use case. You can even do stuff like

```swift
enum MySelection: Hashable {
    case value(Int)
    case settings

    var description: String {
        switch self {
        case .value(let int):
            "\(int)"
        case .settings:
            "Settings"
        }
    }
}

struct SomeView: View {

    @State var selection: MySelection? = nil
    var body: some View {
        NavigationSplitView {
            List(selection: $selection) {
                NavigationLink("Value-based", value: MySelection.value(4))
                NavigationLink("Value-based", value: MySelection.value(5))
                Text("Settings")
                    .tag(MySelection.settings)
            }
        } detail: {
            if let selection {
                Text(selection.description)
            } else {
                Text("make a selection")
            }

        }
    }
}
```

For some sidebar list items that just make more sense to be "hardcoded". I want to reiterate that even if a view doesn't necessarily have a "value", it's worth doing something like the above with the .settings case because there isn't another way to get full programmatic control over that selection

Follow-up-A: That's perfect, I think that will exactly solve my use case (for this app, anyway!) Thank you so much for the time and info! This is super helpful 

Follow-up-Q: One last sharp edge that we're working on. Links that do root-replacement should generally not contain a NavigationStack. If you need a NavigationStack in a detail column, it's best to have that like this:

```swift
NavigationSplitView {
  Sidebar()
} detail: {
  NavigationStack(path: $path) { ... }
}
```

and either
A) root replace that NavigationStack with non-NavigationStack views or
B) Use that $path binding to programmtically push views onto that stack
