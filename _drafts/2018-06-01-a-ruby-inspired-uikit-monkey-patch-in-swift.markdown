---
title: A ruby-inspired Swift-specific UIKit-tip
date: 2018-06-01T12:06:26+0200
categories: [swift, ios]
---

You ever code views? I do. Not all of them, all the time, but some times. I'd
like to share with you a little helper I've been using.

```swift
import Foundation
import UIKit

protocol SubviewAddable {}
extension UIView: SubviewAddable {}

extension SubviewAddable where Self: UIView {
    @discardableResult
    func add<Subview: UIView>(subview: Subview, fn: (Subview, Self) -> Void) -> Subview {
        addSubview(subview)
        fn(subview, self)
        return subview
    }
}
```

It's helpful in that it allows you to add a subview, and within the same place
configure the subview (optionally using its superview). Usage is rather simple:

```swift
let subview = view.add(subview: MyAmazingView()) { amazingView, superview in
    // configure amazingView
    // maybe constrain it to superview?
}
```

While small, it brings us a few really nice benefits:

- We can add and configure the subview in the same logical code block.
- Your block/code hierarchy follows the view hierarchy.
- The subview is returned after it's been added and configured.
- Because we're using generics from Swift, we don't lose any type information
  in neither the configure block nor the return value.

Answers to some things that might not be obvious:

- Why are we using a protocol for our extension? It's because we want to allow
  using `Self` as the type, which allows us keep type information about which
  kind of view we're adding to. Otherwise it would have to be always `UIView`,
  instead of whatever `AmazingView: UIView`-subclass we might have.
- Why are we passing `Self` to the configure block? It's to avoid dealing with
  the optional `view.superview`, and instead deal with a concrete non-optional
  type when configuring our subview.
- Why are we using `<Subview: UIView>`? It's to keep the type information about
  our subview when configuring it in the block, otherwise it would have to be
  a `UIView`, instead of our `AmazingView: UIView`-subclass.

A full, working example, might look like this:

```swift
class LoadingViewController: UIViewController {
    weak var loadingView: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .blue

        // Using it here to to add and configure the subview before weakly
        // storing a reference to it for later
        loadingView = view.add(subview: UIView()) { loadingView, view in
            loadingView.backgroundColor = UIColor(white: 0, alpha: 0.6)

            loadingView.translatesAutoresizingMaskIntoConstraints = false
            NSLayoutConstraint.activate([
                loadingView.topAnchor.constraint(equalTo: view.topAnchor),
                loadingView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
                loadingView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
                loadingView.leadingAnchor.constraint(equalTo: view.leadingAnchor)
            ])

            // Using it here to configure a spinner, and save it temporarily for
            // constraining the label to it
            let spinnerView = loadingView.add(subview: UIActivityIndicatorView()) { spinnerView, contentView in
                spinnerView.hidesWhenStopped = false
                spinnerView.startAnimating()

                spinnerView.translatesAutoresizingMaskIntoConstraints = false
                NSLayoutConstraint.activate([
                    spinnerView.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
                    spinnerView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
                ])
            }

            // Using it here mainly to configure the label
            loadingView.add(subview: UILabel()) { textView, contentView in
                textView.textColor = .white
                textView.text = "Loading…"

                textView.translatesAutoresizingMaskIntoConstraints = false
                NSLayoutConstraint.activate([
                    textView.topAnchor.constraint(equalTo: spinnerView.bottomAnchor, constant: 8),
                    textView.centerXAnchor.constraint(equalTo: spinnerView.centerXAnchor)
                ])
            }
        }
    }
}
```

That's all! Any questions, or feedback? Send me a tweet, or an e-mail.
