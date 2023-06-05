# WWDC 2023 Platform State of the Union

- Swift & SwiftUI
- Experiences
- Hardware
- Tools
- visionOS

## Swift

### Macros

New kind of API that's easier to use and get right - Swift Macros

Macros done the Swift way

Attached macros
@AddAsync - to make code that requires completion handlers instead awaitable calls

Many community-authored macros - benefit from them through Swift packages
Many of this year's new APIs use macros

### C++ Interoperability

Swift interoperability extending to C++

Using C++ from Swift eliminates many issues

## SwiftUI

Doubling down on SwiftUI!

Many improvements for animations

- New API called AnimationPhase
- `.spring()` takes some parameters for bounce and thing
- Many SF Symbols support animation
- Keyframes - define values and let SwiftUI interpolate intermediate values\

Data Flow
- Data property wrappers

Just `@State` and `@Environment`

New `@Observable` macro - no more need to use a property wrapper to trigger updates

## SwiftData

- Built on top of Core Data but with an API redesigned
- Swift Macro system for a streamlined API

`@Model` macro
- Annotate structs
- Automatic Cloud sync
- New `@Query` property wrapper

SwiftUI + SwiftData work hand-in-hand

Spend less time on boilerplate and more time building your ideas

## Missed some stuff

There was some more stuff here that I totally missed because I was mentally reviewing the SwiftData Preview

WidgetKit, tvOS, watchOS, lots of stuff about using SwiftUI across these platforms?

## watchOS

New APIs, improved existing APIs. I don't develop watchOS apps (yet) so it doesn't mean much to me.

## Values

### Accessibility

Some nice accessibility features 

You, as a developer, must make your visionOS apps accessible to everyone! Do it! Bring spatial computing to diverse sets of users!

### Privacy

- Photos: Users can select limited photos or everything? New photo picker that lets users pick what to share
- App Privacy: Important for users to understand how you use their data. Labels for how you use their data. Two updates to help you understand how 3rd-party SDKs use your data:
  - Manifest: Combine manifests across 3rd-party SDKs into a summary report
  - Signatures for 3rd-party SDKs: When you adopt a new version of a 3rd-party SDK, Xcode validates it's signed by the same developer
- Communication Safety - Sensitive Content Analysis. Framework detects sensitive content entirely on-device. Sensitive content blurring.

## StoreKit

- In-app purchase stuff

### New Purchase flow views

- New collection of views to power your merchandising experience
  - `Product....`
  - `SubscriptionStoreView()`

### SKAdNetwork

- Measuring conversions/downloads + re-engagement - Version 5 later this year

## Tools

### Xcode

Previews! 

- New syntax using Macros
- Use Previews across frameworks

Git stuff

Testing
- Complete redesign of the test report
- Results overview
- Better support for jumping around in test results

Plug for Xcode Cloud

## visionOS

SwiftUI, RealityKit and ARKit, extended for visionOS

By default, apps launch into the Shared Space. It's where apps exist side-by side, similar to Mac.

- Window: SwiftUI Scene and behaves just as you'd expect
- Volume: Allows 3d content to sit alongside 2d content, showcase 3d objects
- FullSpace: Dedicated space in which your apps windows, volumes, or 3d objects appear across the user's view

In Xcode, add visionOS to destination, when you rebuild app gets stuff

SwiftUI
- Add depth or a 3d object
- ZStack - separate with depth
- Add subtle changes in depth to your elements with `.offset(z: )` - show emphasis or indicate a change in modality
- Create a Volume with SwiftUI, sits side-by-side with other apps
- SwiftUI Windows and Volumes can be inside a FullSpace and place stuff anywhere in a user's room
- Mix SwiftUI & RealityKit
- Ornaments - allow you to affix ui elements to
- Materials adapt to the environment around your user
- Recommend use SwiftUI to build visionOS apps

More stuff about RealityKit and ARKit, but I haven't used those before so it doesn't mean a ton to me

### Reality Composer Pro

See how your 3D content looks
Add materials to 3D models, scale, rotate, make sure it looks and feels how you want
Powerful new tool that works side-by-side with Xcode

### Test Flight

Available for Vision Pro

### Unity

- Unity-created apps can co-exist with visionOS apps and take advantage of Vision Pro
- Use Unity's authoring tools to create new visionOS games and apps

"Deep integration between Unity and visionOS" - your apps appear alongside other apps

### Vision Pro Developer Labs

- Go out to physical places to test Vision Pro apps on hardware
- Submit a request for Apple to test it on Vision Pro hardware

175 In-depth Sessions - 40 for visionOS alone!
