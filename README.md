# hiPPo

`hiPPo` is a C++ client designed to communicate with `SoHal`. The devices and
methods in `hiPPo` mirror the `SoHal` spec, but `hiPPo` handles all of the
websocket communication and JSONRPC-2.0 protocol for the user.

> <B>Note</B>: Please note that `hiPPo` is currently in heavy development
stage and there may be a few bugs in it.  Please report any bugs and they
will be promptly fixed

As an example, the following code should turn on `Sprout`'s projector:
```
#include "include/projector.h"

int main(int argc, char *argv[]) {
  uint64_t err = 0;
  uint32_t open_count = 0;
  hippo::Projector projector;

  if (err = projector.open(&open_count)) {
    // handle the error
  }
  if (err = projector.on()) {
    // handle the error
  }
  if (err = projector.close()) {
    // handle the error
  }
  return 0;
}
```

## Implementation details

### Standalone library

Currently `hiPPo` is a standalone library, as all hiPPo dependencies have
been linked statically into hippo.dll.

hiPPo uses the libwebsockets project (https://libwebsockets.org) for its
websockets communication with SoHal.

hiPPo uses the nlohmann's json project (https://github.com/nlohmann/json)
for generating and parsing the JSONRPC messages sent to SoHal.


### Overloaded class methods

While we're trying to verbatim copy `SoHal`'s API, C++ needs a bit
more 'fillcode'. For example, in `hippy` (Python sistership client),
in order to open the projector we need the following code (assuming
the object is already created):

```
open_count = projector.open()
```

but, of course, we can also just 'forget' about the `open_count` return
value (containing the number of clients that currently hold the projector
open) if we just want to open it:

```
projector.open()
```

In order to enable a similar functionality in C++, we're overloading the
class methods. For example, the `open` function has two different prototypes:

```
  virtual uint64_t open();
  virtual uint64_t open(uint32_t *open_count);
```

so, the C++ code can be written as:

```
  if (err = projector.open()) {
    // handle the error
  }
```
or
```
  uint32_t open_count = 0;
  if (err = projector.open(&open_count)) {
    // handle the error
  }
```

This is a bit more interesting in functions like `projector.keystone`,
where the `set` variant of the method also returns the actual values
it has been set to. In this case we have a `(hippo::Keystone *get)`
prototype for getting the keystone value, a
`(const hippo::Keystone &set, hippo::Keystone *get)` to set the keystone
and get the actual value it has been set to (in some cases it may differ)
back from `SoHal`, and a `(const hippo::Keystone &set)` prototype
for setting the keystone but not needed to receive the actual set value.

```
  uint64_t keystone(hippo::Keystone *get);
  uint64_t keystone(const hippo::Keystone &set);
  uint64_t keystone(const hippo::Keystone &set, hippo::Keystone *get);
```

This allow us to skip the parsing of the return value back from `SoHal`'s
into C++ if the user does not needed it. You will notice that all the
functions follow this approach when possible.

> <B>Note</B>: Please note that we recommend to always check the return
value for the `projector.keystone` function, but we expose these functions
for consistency with the rest of the API.


### No exceptions thrown

`hiPPo` error management is based in old-style functions returning error
codes, so, no exceptions will be thrown from `hiPPo`.

Error codes generated in SoHal and sent back to hiPPo or generated in
hiPPo itself, are returned to the user as a `uint64_t` value. HiPPo also
provides the `hippo::strerror()` API that will return a human-readable
string representation of the last error occurred.


### Source code style guide

We use Google's style guide for `hiPPo`. This was not our original preferred
style, but somebody wrote a very clear and useful  description in
`https://google.github.io/styleguide/cppguide.html` and they also have
`cpplint`, a tool to automatically check your code.
This is very cool and totally worth it to use. And we're already
getting used to it ;)

So yes, we're running `hiPPo` code through `cpplint` (for C++) before accepting
a pull request. We are also setting the warning level to the maximum and
treating all warnings as errors. Please make sure you follow this guide
if you submit any pull request. Thank you!
