---
title: A statically typed NotificationCenter in Swift
date: 2017-01-20T16:54:00+02:00
categories: [swift,mobile,ios]
---

Event hubs, event bus, notification center… all names for an architectural
solution to decouple components of your program. You do some work, and then you
announce to the world you did, but you don't care if anybody is listening.

There's an Apple-provided implementation available for this when doing mobile
development for iOS, it's called [NotificationCenter][1]. An example of where
you'd use this could be to post a notification every time the current user
changes.

To use it, you come up with a magic name for an event, let's say `userChanged`,
and every time we post this notification we vow to include the new user in
[`userInfo`][2] dictionary, let's say `userInfo["user"]` holds the new `User`
instance.

My main problem with this, is what's to say that:

- I won't misspell the event name? (`usrChanged`)
- I won't misspell the `user` key in `userInfo`? (`usr`, `newUser`)
- I won't forget to include the new `User` at all?
- I won't send the wrong type as the new user? (perhaps `nil`?)

I have good news though. Swift's generics allow us to make a very nice solution
that alleviates all these problems!

{% gist 575ee47fae55cba18d29f28227740a18 hub.swift %}

The above definition of `Hub`, `Events` and `Event<T>` allows us to:

- Ensure nobody ever misstypes an event by extending `Events` to allow usage of
  of the dot short-hand on `observe` and `post`:

  ```swift
  extension Events {
    static let userUpdated = Event<User?>("UserUpdated")
  }
  ```
- Guarantee that all *observed* events of type `T` actually contains a `T`:

  ```swift
  hub.observe(.userUpdated) { (user: User?) in /* … */ }
  ```
- Guarantee that all *posted* events of type `T` actually includes a `T`:

  ```swift
  hub.post(.userUpdated, User(name: "Alice"))
  hub.post(.userUpdated, nil)
  ```

And all of this is built on top of Apple's own NotificationCenter and your
normal Swift generics!

There's a GitHub repository for the code in this post, you're very welcome to
send pull-requests: <https://github.com/Burgestrand/swift-hub>. That's all for
now, thanks for staying!

[1]: https://developer.apple.com/reference/foundation/nsnotificationcenter
[2]: https://developer.apple.com/reference/foundation/notification/1779652-userinfo
