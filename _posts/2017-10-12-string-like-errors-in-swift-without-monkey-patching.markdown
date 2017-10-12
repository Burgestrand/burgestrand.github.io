---
title: String-like errors in Swift without monkey patching
date: 2017-10-12T12:34:45+0200
categories: [swift, ios, mobile]
---

A while back I [read about a neat trick](https://stackoverflow.com/a/40629365/187710).

``` swift
extension String: Swift.Error {}
```

I am a bit afraid of it.

By adding this line, you can now `throw "Any string"` since Strings now conform
to `Error`. However, I've always been a bit afraid of monkey patching things
that are not mine. It tends to cause problems. This is personal FUD.

## There are strings other than just `String`!

Swift has many neat protocols, and we'll be using three in particular today to
make a new kind of error that we'll call `ErrorMessage`.

### `Error`

> A type representing an error value that can be thrown

By adopting this protocol on our types, we make it possible to `throw` them.
If you use [Reactive Swift][], for example, the error type of all signals must
have adopted this protocol.

By adopting this protocol, we can `throw` any `ErrorMessage`.

[Reactive Swift]: https://github.com/ReactiveCocoa/ReactiveSwift/#readme

### `LosslessStringConvertible`

> A type that can be represented as a string in a lossless, unambiguous way.

By adopting this protocol, we're saying that every `ErrorMessage` is a `String`;
keep in mind that we're _not_ saying that every `String` is an `ErrorMessage`.

### `ExpressibleByStringLiteral`

> A type that can be initialized with a string literal.

Remember that we said before that not _every_ `String` is an `ErrorMessage`? By
adopting this protocol, we now said that it is. _Every_ `String` is also an
`ErrorMessage`.

Without this protocol we would need to explicitly wrap our error messages as
`ErrorMessage(…)` _every frickin' time_. With this protocol, however:

``` swift
let error: ErrorMessage = "Oh no, something went wrong!"
```

Yes, that'll work. No, I'm not pulling your leg. Yes, this _is_ the bees knees.

### All of us together, now!

``` swift
struct ErrorMessage: Swift.Error, ExpressibleByStringLiteral, LosslessStringConvertible {
	let description: String
	
	// ExpressibleByStringLiteral
	init(stringLiteral string: String) {
		self.description = string
	}
	
	// LosslessStringConvertible
	init(_ description: String) {
		self.description = description
	}
}
```

- `Error` says we can `throw` it.
- `LosslessStringConvertible` says every `ErrorMessage` is also a `String`.
- `ExpressibleByStringLiteral` says every `String` is also an `ErrorMessage`.

### Usage

If you have no type information available you'll need to cast it.

``` swift
func crashBangBoom() throws {
	throw "Boom!" as ErrorMessage
}

let error: ErrorMessage = "Oh no!"
```

However, if you *do* have inferred type information and can play with string
literals then things will be _smooooth_.

_Note: This example uses ReactiveSwift._ 
``` swift
let producer = SignalProducer<String, ErrorMessage> { observer, lifetime in
	observer.send(error: "Oh no!")
}
```

### Tips

You might want to check out `LocalizedError`, it has some more of that error
handling goodness.

That was all. Be well!
