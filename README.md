# xcffib [![Build Status](https://travis-ci.org/tych0/xcffib.svg?branch=master)](https://travis-ci.org/tych0/xcffib)

`xcffib` is intended to be a (mostly) drop-in replacement for `xpyb`. `xpyb`
has an inactive upstream, several memory leaks, is python2 only and doesn't
have pypy support. `xcffib` is a binding which uses
[cffi](https://cffi.readthedocs.org/), which mitigates some of the issues
described above. `xcffib` also builds bindings for 27 of the 29 (xprint and xkb
are missing) X extensions in 1.10.

## Dependencies

Currently `xcb-types` (any tagged version, not just the old one on Hackage)
doesn't run against `xcb-proto` 1.10; there is a hacked branch available at
[tych0/xcb-types](http://github.com/tych0/xcb-types) that allows you to parse
xcb-proto 1.10 mostly correctly. Other than that, you should be able to install
all the deps from hackage or pip. The
[.travis.yaml](https://github.com/tych0/xcffib/blob/master/.travis.yml) has an
example of how to install the dependencies on Ubuntu flavors.

## Differences

There are lots of differences right now between `xpyb` and `xcffib`, mostly
because `xcffib` isn't done yet :-)

* In general, you should `s/xcb/xcffib/g`
* `xcb.Exception` is spelled `xcffib.XcffibException` and is also a parent of
   all exceptions generated by xcffib.
* `xcb.ConnectException` is gone, it was unused
* `xcffib.ConnectionException` is raised on connection errors
* `xcb.Iterator` is gone; similar functionality is implemented by
  `xcffib.pack_list`.
* `xcb.Request` is gone. It was an entirely internal and unnecessary interface.
* `xcffib.Connection.send_request` takes slightly different (but more sensible)
   arguments.
* Everywhere `xcb-proto` says `char`, `xcffib` uses a char. That means on input
  for a `<list type="char"/>`, you can use a python string literal. `xcffib`
  also gives you a string of length 1 out for each element in such a list,
  instead of an `int`. Finally, there is a helper method called `to_string` on
  `xcffib.List`, to convert these string-like things into native strings. In
  both python2 and python3 you get a native `str`. This means that for things
  like `xproto.STR`, you can just do `the_str.name.to_string()` instead of
  `''.join(map(chr, the_str.name))`.
* As above, `void` is also packed/unpacked as `char`s, since the convention is
  to use it as string data, e.g. in `xproto.ChangeProperty`.
* The submodule `xcb` is gone. The top module re-exported all these constants
  anyway, so they live there now. i.e. `xcb.xcb.CurrentTime` is now just
  `xcffib.CurrentTime`.

## Enhancements

* When sending requests with nested structs you no longer have to pack the
  contents yourself. For example, when calling `xproto.FillPoly`, you used to
  have to convert the `POINT`s you were passing in to some sort of buffer which
  had them `struct.pack`'d. Now, you can just pass an iterable (or
  `xcffib.List`) of `POINT`s and it will be automatically packed for you.
* Most of the lower level XCB connection primitives that were previously not
  exposed are now available via `xcffib.{ffi,C}`, assuming you want to go out
  of band of the binding.
* Checked vs. Unchecked requests are still supported (via Checked and Unchecked
  function calls). However, there is also an additional optional parameter
  `is_checked` to each request function, to allow you to set the checked status
  that way. Additionally, requests that are (un)checked by default, e.g.
  `QueryTree` (`CreateWindow`), have a `QueryTreeChecked`
  (`CreateWindowUnchecked`) version which just has the same default behavior.
* The `FooError` `BadFoo` duality is gone; it was difficult to understand what
  to actually catch if you wanted to handle an error. Instead, `FooError` and
  `BadFoo` are aliases, and both implement the X error object description and
  python Exception (via inheriting from `XcffibException`).

## Why haskell?

Why is the binding generator written in haskell? Because haskell is awesome.

## TODO

* XGE support? (xpyb doesn't implement this either)
* xprint and xkb support. These will require some non-trivial work in
  xcb-types, since it won't parse them correctly.
