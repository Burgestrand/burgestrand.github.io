---
title: String errors in Swift without monkey patching
date: 2017-10-11T15:59:45+0200
categories: [swift, ios, mobile]
---

A while back I [read about a neat trick](https://stackoverflow.com/a/40629365/187710).

``` swift
extension String: Swift.Error {}
```

I am a bit afraid of it.

By adding this line, you can now `throw "Any string"` since Strings now conform
to `Error`. However, I've always been a bit afraid of monkey patching things
that are not mine. It tends to cause problems. This personal FUD.

## Say hello to my little friend

It's an `Error`, it's a `String`, and it's lossless!

``` swift
struct ErrorMessage: Swift.Error, ExpressibleByStringLiteral, LosslessStringConvertible {
	let description: String
	
	init(stringLiteral string: String) {
		self.description = string
	}
	
	init?(_ description: String) {
		self.description = description
	}
}
```

Throwing this is nearly as convenient before, just add casting!

``` swift
func crashBangBoom() throws {
	throw "Boom!" as ErrorMessage
}
```

It becomes even smoother where there's a concrete error involved.

``` swift
let result = Result<String, ErrorMessage>(error: "Nein, nein, nein!")
```

Extra credit if expand this by making it conform to `LocalizedError` too!

That was all. Be well!
