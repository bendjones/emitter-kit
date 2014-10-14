<img src="http://i.imgur.com/PnCPxBz.jpg"/>

EmitterKit is a cleaner alternative to [**NSNotificationCenter**](http://nshipster.com/nsnotification-and-nsnotificationcenter/).

- No longer be forced to typecast event data in your closures.
- No longer be forced to `removeObserver` in your classes' `deinit` methods.
- Backwards-compatible with `NSNotificationCenter` (like `UIKeyboardWillShowNotification`)!
- Simpler syntax
- Key-value observation!
- And more probably...

---

### **Installation**

Not yet available on CocoaPods.

For now, you can do one of three things:

1. Add `EmitterKit.xcodeproj` as a submodule in your Xcode project and add `EmitterKit.framework` as a target dependency in your Build Phases.

2. Add the pre-built `EmitterKit.framework` to your Xcode project. Built with Xcode 6 GM.

3. Add the source files (in the `src` folder) to your Xcode project.

---

### **Event**

An `Event` emits data. Its generic parameter specifies what type of data it emits! This is awesome.

All listeners are removed when an `Event` is deallocated.

```Swift
let didLogin = Event<User>()

didLogin.once { user in
  println("Successfully logged in as \(user.name)!")
}

didLogin.emit(user)
```

`Event` is a subclass of `Emitter`.

I tend to use `Event`s as properties of my Swift classes, when it makes sense.

```Swift   
class MyScrollView : UIScrollView, UIScrollViewDelegate {
  let didScroll = Event<CGPoint>()
  
  override init () {
    super.init()
    delegate = self
  }
  
  func scrollViewDidScroll (scrollView: UIScrollView) {
    didScroll.emit(scrollView.contentOffset)
  }
}
```

Otherwise, I can use a **target** to associate an `Event` with a specific `AnyObject`. This is useful for classes I cannot add properties to, like `UIView` for example.

```Swift
let myView = UIView()
let didTouch = Event<UITouch>()

didTouch.once(myView) {
  println("We have a winner! \($0)")
}

didTouch.emit(myView, touch)
```

---

### **Signal**

A `Signal` is essentially an `Event` that can't pass data. Convenient, huh?

This is a subclass of `Emitter`, too.

```Swift
let didLogout = Signal()

didLogout.once {
  println("Logged out successfully... :(")
}

didLogout.emit()
```

---

### **Notification**

`Notification` wraps around `NSNotification` to provide backwards-compatibility with Apple's frameworks (e.g. `UIKeyboardWillShowNotification`) and third party frameworks. 

Use it to create `NotificationListener`s that will remove themselves when deallocated. Now, you no longer have to call `removeObserver()` in your deinit phase!

You **do not** need to retain a `Notification` for your listener to work correctly. This is one reason why `Notification` does not subclass `Emitter`.

```Swift
Notification(UIKeyboardWillShowNotification).once { data in
  println("keyboard showing. data: \(data)")
}
```

---

### **Listener**

A `Listener` represents a closure that will be executed for a certain reason. 

It is an abstract class. You cannot call any of its initializers directly.

It starts listening immediately upon creation. You can toggle it on and off with its `isListening` boolean property.

If its `once` boolean property equals `true`, it will stop listening after it executes once.

**Important:** Remember to retain a `Listener` if its `once` equals `false`! Make a `[Listener]` property and put it there.

```Swift
var listeners = [Listener]()

// Retain that sucka
listeners += mySignal.on {
  println("beep")
}

// Single-use Listeners retain themselves ;)
mySignal.once {
  println("boop")
}
```

---

### **Key-Value Observation**

EmitterKit adds `on()`, `once()`, and `removeListeners()` instance methods to every `NSObject`.

```Swift
let myView = UIView()
let myProperty = "layer.bounds" // supports dot-notation!

listeners += myView.on(myProperty) { 
  (values: Change<NSValue>) in
  println(values)
}
```

[Check out the `Change` class](https://github.com/aleclarson/emitter-kit/blob/master/src/ChangeListener.swift#L36-L56) to see what wonders it contains. It implements the `Printable` protocol for easy debugging!

The `NSKeyValueObservingOptions` you know and love are also supported! Valid values are `.Old`, `.New`, `.Initial`, `.Prior`, and `nil`. If you don't pass a value at all, it defaults to `.Old | .New`.

```Swift
myView.once("backgroundColor") { (values: Change<UIColor>) in
  println(values)
}
```

It runs on top of traditional KVO techniques, so everything works as expected!

**WARNING:** If you use these methods, you must call `removeListeners(myListenerArray)` before your `NSObject` deinits. Otherwise, your program will crash. I suggest making a subclass of `UIView`, overriding `willMoveToWindow()`, and putting `removeListeners()` in there. That's not always ideal if you're not working with a `UIView`, but that's all I use it for right now, so I can't help you in other cases.

---

### Other classes

- **Change**: Holds an `oldValue` and a `newValue` for an observed property. And it's generic!
- **Emitter**: An abstract class that supports `Event` and `Signal`. Not relevant to you really.
- **EmitterListener**: The `Listener` subclass for `Event`s and `Signal`s.
- **NotificationListener**: The `Listener` subclass for `Notification`s.

---

### Major changes in v3.0

- Added `NotificationListener`, `EmitterListener`, and `ChangeListener` (all subclasses of `Listener`).
- Added `Change`, a generic class with an `oldValue`, a `newValue`, and a `keyPath`.
- All `NSObject`s have an `on()` and `once()` method for KVO!
- `Notification` does not subclass `Emitter` anymore.
- `Notification` now has `emit` methods.
- `Notification` no longer need to be retained.
- Removed `removeAllListeners` from `Emitter`.
- Removed `listenersForTarget` from `Emitter`.
- `ListenerStorage` no longer exists. Use a `[Listener]` or whatever you want.
- Removed many exposed internal functions.

---

Crafted by Alec Larson [@aleclarsoniv](https://twitter.com/aleclarsoniv)
