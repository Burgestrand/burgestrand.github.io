---
title: Fun with Swift protocols and view controllers in storyboards
date: 2017-08-31T15:00:00+0200
categories: [swift, ios, storyboards]
---

Xibs, storyboards, or programmatic view controllers… what to use? My answer? It
depends. Very helpful, I know! I'll elaborate. In order of what I'm most
likely to use:

1. One storyboard per screen (view controller). Like a xib, but works with prototyped table view and collection view cells.
2. No storyboard, so a programmatic view controller. Dependency injection through initializer.
3. A self-contained storyboard, possibly multiple screens, I only use this if I don't need any data from the outside.


I wanted to share a Swift snippet with you to help with number **1** up there. It's not
unique, ground-breaking or new by any means, but it's useful and it feels nice.
Without any bells or whistles, here it is in all it's glory.

```swift
import UIKit

/// A protocol conformed to by all view controllers that expect to be
/// instantiated from a storyboard.
public protocol StoryboardDesigned: class {
    static var storyboard: Storyboard<Self> { get }
}

/// By default, a storyboard-designed view controller has named the storyboard
/// it belongs to the same as itself, sans `Controller`, and is the initial view
/// controller.
///
/// - example: PostViewController -> PostView.storyboard
public extension StoryboardDesigned where Self: UIViewController {
    public static var storyboard: Storyboard<Self> {
        let className = String(describing: self)
        let storyboardName = className.replacingOccurrences(of: "Controller", with: "")
        return Storyboard<Self>(storyboardName, identifier: nil, bundle: nil)
    }
}

/// A simple wrapper-type that we use to identify storyboards.
public struct Storyboard<T> {
    let name: String
    let identifier: String?
    let bundle: Bundle?

    init(_ name: String, identifier: String? = String(describing: T.self), bundle: Bundle? = nil) {
        self.name = name
        self.identifier = identifier
        self.bundle = bundle
    }

    var storyboard: UIStoryboard {
        return UIStoryboard(name: name, bundle: bundle)
    }

    func instantiate() -> T {
        guard let identifier = identifier else {
            return storyboard.instantiateInitialViewController() as! T
        }

        return storyboard.instantiateViewController(withIdentifier: identifier) as! T
    }
}
```

By using the above code, and assuming you stick to my convention of naming your
single-view storyboards the same as your view controller, and mark it as the
initial view controller of the storyboard, then you may use the above code like this:

```swift
final class PostViewController: UIViewController, StoryboardDesigned {
  // … outlets and other code in here …
}
```

```swift
func showPostViewController(from viewController: UIViewController) {
  let postViewController = PostViewController.storyboard.instantiate()
  viewController.present(postViewController, animated: true, completion: nil)
}
```

That's all! There's nothing more for now. Thanks for reading, I hope you found
it worthwhile.
